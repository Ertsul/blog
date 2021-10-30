## 挂载 Vue 实例

在引入 Vue 模块和实例化 Vue 对象后，要进行 $mount() 操作。

```js
new Vue({
  render: h => h(App)
}).$mount('#app')
```

$mount 方法主要调用 mountComponent 方法。

### mountComponent

函数参数有：

- vm：Vue 实例。
- el：挂载的 DOM 节点，可以是 ID 选择器或者实际的 DOM 节点。
- hydrating：该参数跟服务端渲染有关。

函数执行逻辑为：

1. 判断是否实例化 Vue 对象的时候是否有传 render 函数。没有则手动添加上 render 函数，并且这个 render 函数返回一个空的 VNode。主要源码如下：

```js
if (!vm.$options.render) {
    vm.$options.render = createEmptyVNode;
}
```

2. 实例化一个渲染 Watcher，该 Watcher 会调用回调函数 updateComponent 方法更新 Dom。主要源码如下:

```js
// 会在初始化和数据更新时执行回调
new Watcher(vm, updateComponent, noop, {
    before: function before () {
        if (vm._isMounted && !vm._isDestroyed) {
            callHook(vm, 'beforeUpdate');
        }
    }
}, true /* isRenderWatcher */);
```

updateComponent 方法主要执行：

- 调用原型上的 _render 函数，返回一个 VNode。
- 调用原型上的 _update 函数，更新 DOM。

主要源码如下：

```js
var updateComponent;
/* istanbul ignore if */
if (config.performance && mark) {
    updateComponent = function () {
        var name = vm._name;
        var id = vm._uid;
        var startTag = "vue-perf-start:" + id;
        var endTag = "vue-perf-end:" + id;

        mark(startTag);
        var vnode = vm._render(); // 生成VNdoe
        mark(endTag);
        measure(("vue " + name + " render"), startTag, endTag);

        mark(startTag);
        vm._update(vnode, hydrating); // 更新DOM
        mark(endTag);
        measure(("vue " + name + " patch"), startTag, endTag);
    };
} else {
    updateComponent = function () {
        vm._update(vm._render(), hydrating);
    };
}
```

3. 设置挂载标志 _isMounted 为 true，并调用 mounted 钩子。

接下来看 updateComponent 中的 _render 和 _update 方法。

#### _render

该函数在引入 Vue 模块的时候执行 renderMixin 函数的时候挂载在 Vue 原型上。该方法最主要的功能就是生成 VNode。

生成 VNode 最核心的就是调用 $createElement 方法。

```js
function renderMixin(Vue) {
    Vue.prototype._render = function() {
        vnode = render.call(vm._renderProxy, vm.$createElement)
    }
}
```

$createElement 在 initMixin 初始化方法中的 initRender 中已经定义，而 $createElement 最核心的调用 createElement 方法。

```js
function initRender (vm) {
  vm.$createElement = function (a, b, c, d) { return createElement(vm, a, b, c, d, true); };
}
```

createElement 方法内部最终调用 _createElement 方法，主要对函数参数进行处理。

```js
function createElement (
  context,
  tag,
  data,
  children,
  normalizationType,
  alwaysNormalize
) {
  if (Array.isArray(data) || isPrimitive(data)) {
    normalizationType = children;
    children = data;
    data = undefined;
  }
  if (isTrue(alwaysNormalize)) {
    normalizationType = ALWAYS_NORMALIZE;
  }
  return _createElement(context, tag, data, children, normalizationType)
}
```

##### _createElement

主要执行：

- 规范化 children：因为传入的 children 是任意类型的，需要将 children 转化为 VNode 类型的数组。
- 生成并返回 VNode。

规范化 children 主要源码如下：

```js
function _createElement (
  context,
  tag,
  data,
  children,
  normalizationType
) {
      if (normalizationType === ALWAYS_NORMALIZE) {
          children = normalizeChildren(children);
      } else if (normalizationType === SIMPLE_NORMALIZE) {
          children = simpleNormalizeChildren(children);
      } 
}

```

规范化 children 主要有两种类型的规范化，主要区别在于 render 函数是编译生成的还是用户手写的。

- SIMPLE_NORMALIZE：调用 simpleNormalizeChildren，调用场景是 render 是编译生成的。翻译注释可知：虽然编译生成的 children 已经是 VNode 类型的数组，但是由于 functional component 函数式组件返回的是一个数组而不是一个根节点，需要对其 children 进行扁平化处理，使其深度只有一层。

```js
// The template compiler attempts to minimize the need for normalization by
// statically analyzing the template at compile time.
//
// For plain HTML markup, normalization can be completely skipped because the
// generated render function is guaranteed to return Array<VNode>. There are
// two cases where extra normalization is needed:

// 1. When the children contains components - because a functional component
// may return an Array instead of a single root. In this case, just a simple
// normalization is needed - if any child is an Array, we flatten the whole
// thing with Array.prototype.concat. It is guaranteed to be only 1-level deep
// because functional components already normalize their own children.
function simpleNormalizeChildren (children) {
  for (var i = 0; i < children.length; i++) {
    if (Array.isArray(children[i])) {
      return Array.prototype.concat.apply([], children)
    }
  }
  return children
}
```

- ALWAYS_NORMALIZE：调用 normalizeChildren，调用场景是 render 是用户手写的。

```js
// 2. When the children contains constructs that always generated nested Arrays,
// e.g. <template>, <slot>, v-for, or when the children is provided by user
// with hand-written render functions / JSX. In such cases a full normalization
// is needed to cater to all possible types of children values.
function normalizeChildren (children) {
  return isPrimitive(children)
    ? [createTextVNode(children)]
    : Array.isArray(children)
      ? normalizeArrayChildren(children)
      : undefined
}
```

normalizeArrayChildren 的逻辑如下：

- 如果有嵌套逻辑，则递归调用 normalizeArrayChildren；
- 如果是字符串或者数字，则创建文本节点。

规范化 children 之后，生成 VNode，主要有以下逻辑：

1. 如果标签名 tag 是字符串类型：
   - tag 为内置节点/html 标签名：直接创建 VNode 实例；
   - tag 为已经注册的组件名，则调用 createComponent 创建 VNode。
2. 组件类型，调用 createComponent 创建 VNode。

#### _update

该函数在引入 Vue 模块的时候执行 lifecycleMixin 函数的时候挂载在 Vue 原型上。

```js
function lifecycleMixin(Vue) {
  Vue.prototype._update = function (vnode, hydrating) {
    var vm = this;
    var prevEl = vm.$el;
    var prevVnode = vm._vnode;
    var restoreActiveInstance = setActiveInstance(vm);
    vm._vnode = vnode;
    // Vue.prototype.__patch__ is injected in entry points
    // based on the rendering backend used.
    if (!prevVnode) {
      // initial render
      vm.$el = vm.__patch__(vm.$el, vnode, hydrating, false /* removeOnly */);
    } else {
      // updates
      vm.$el = vm.__patch__(prevVnode, vnode);
    }
    restoreActiveInstance();
    // update __vue__ reference
    if (prevEl) {
      prevEl.__vue__ = null;
    }
    if (vm.$el) {
      vm.$el.__vue__ = vm;
    }
    // if parent is an HOC, update its $el as well
    if (vm.$vnode && vm.$parent && vm.$vnode === vm.$parent._vnode) {
      vm.$parent.$el = vm.$el;
    }
    // updated hook is called by the scheduler to ensure that children are
    // updated in a parent's updated hook.
  };
}
```

_update 的作用就是通过 VNode 生成真实的 DOM 节点并插入到父节点中。

上面代码最主要的逻辑就是调用 `__patch__` 函数并赋值给 vm.$el。

`__patch__` 调用逻辑源码：

```js
Vue.prototype.__patch__ = inBrowser ? patch : noop;
var patch = createPatchFunction({ nodeOps: nodeOps, modules: modules });
```

可见，在浏览器端，调用的是 createPatchFunction 函数，函数参数为：

- nodeOps：封装了 DOM 操作方法
- modules：定义了一些模块的钩子函数的实现

createPatchFunction 函数内部返回了一个 patch 函数，函数为：

```js
function createPatchFunction (backend) {
    return function patch (oldVnode, vnode, hydrating, removeOnly) {
        // ...
    }
}
```

逻辑捋下来，在 `__update` 中初始化逻辑 `vm.__patch__(vm.$el, vnode, hydrating, false /* removeOnly */);` 最终调用的 createPathFunction 返回的 patch 函数。

##### patch

patch 函数内部，略过服务端渲染和判空逻辑，只保留初始化逻辑，代码如下：

```js
function patch (oldVnode, vnode, hydrating, removeOnly) {
    var isInitialPatch = false;
    var insertedVnodeQueue = [];

    var isRealElement = isDef(oldVnode.nodeType);
    if (!isRealElement && sameVnode(oldVnode, vnode)) {
        // patch existing root node
        patchVnode(oldVnode, vnode, insertedVnodeQueue, null, null, removeOnly);
    } else {
        if (isRealElement) {
            oldVnode = emptyNodeAt(oldVnode);
        }

        // replacing existing element
        var oldElm = oldVnode.elm;
        var parentElm = nodeOps.parentNode(oldElm);

        // create new node
        createElm(
            vnode,
            insertedVnodeQueue,
            // extremely rare edge case: do not insert if old element is in a
            // leaving transition. Only happens when combining transition +
            // keep-alive + HOCs. (#4590)
            oldElm._leaveCb ? null : parentElm,
            nodeOps.nextSibling(oldElm)
        );
    }
}
```

patch 函数的实参为：

- oldVnode：vm.$el 挂载元素，在 $mount -> mountComponent 中已赋值
- vnode：_render 函数返回的 Vnode
- hydrating：false，非服务端渲染
- removeOnly：false

将函数先判断是否是实际的 DOM 节点，这里是真是的 DOM 节点，然后通过 emptyNodeAt 创建一个新的 VNode 对象。emptyNodeAt 源码：

```js
function emptyNodeAt (elm) {
    return new VNode(nodeOps.tagName(elm).toLowerCase(), {}, [], undefined, elm)
}
```

将挂载点 oldVnode 转化为 VNode 之后，调用 createElm 创建新的 node 节点，生成实际的 DOM 节点。createElm 主要初始化源码为：

```js
function createElm (
    vnode,
    insertedVnodeQueue,
    parentElm,
    refElm,
    nested,
    ownerArray,
    index
 ) {
    if (isDef(vnode.elm) && isDef(ownerArray)) {
      // This vnode was used in a previous render!
      // now it's used as a new node, overwriting its elm would cause
      // potential patch errors down the road when it's used as an insertion
      // reference node. Instead, we clone the node on-demand before creating
      // associated DOM element for it.
      vnode = ownerArray[index] = cloneVNode(vnode);
    }

    vnode.isRootInsert = !nested; // for transition enter check
    if (createComponent(vnode, insertedVnodeQueue, parentElm, refElm)) {
      return
    }

    var data = vnode.data;
    var children = vnode.children;
    var tag = vnode.tag;
    if (isDef(tag)) {
      {
        if (data && data.pre) {
          creatingElmInVPre++;
        }
        if (isUnknownElement$$1(vnode, creatingElmInVPre)) {
          warn(
            'Unknown custom element: <' + tag + '> - did you ' +
            'register the component correctly? For recursive components, ' +
            'make sure to provide the "name" option.',
            vnode.context
          );
        }
      }

      vnode.elm = vnode.ns
        ? nodeOps.createElementNS(vnode.ns, tag)
        : nodeOps.createElement(tag, vnode);
      setScope(vnode);

      /* istanbul ignore if */
      {
        createChildren(vnode, children, insertedVnodeQueue);
        if (isDef(data)) {
          invokeCreateHooks(vnode, insertedVnodeQueue);
        }
        insert(parentElm, vnode.elm, refElm);
      }

      if (data && data.pre) {
        creatingElmInVPre--;
      }
    } else if (isTrue(vnode.isComment)) {
      vnode.elm = nodeOps.createComment(vnode.text);
      insert(parentElm, vnode.elm, refElm);
    } else {
      vnode.elm = nodeOps.createTextNode(vnode.text);
      insert(parentElm, vnode.elm, refElm);
    }
  }
```

上面代码分析可得，由于挂载元素（vm.$el）是实际的 DOM 元素，故上面的主要执行逻辑为：

```js
vnode.elm = vnode.ns
     ? nodeOps.createElementNS(vnode.ns, tag)
 : nodeOps.createElement(tag, vnode);
setScope(vnode);

/* istanbul ignore if */
{
    createChildren(vnode, children, insertedVnodeQueue);
    if (isDef(data)) {
        invokeCreateHooks(vnode, insertedVnodeQueue);
    }
    insert(parentElm, vnode.elm, refElm);
}
```

接下来调用平台的方法创建元素，这里创建的元素是挂载元素，然后设置调用 createChildren 创建子元素节点。createChildren 函数源码：

```js
function createChildren (vnode, children, insertedVnodeQueue) {
  if (Array.isArray(children)) {
    {
      checkDuplicateKeys(children);
    }
    for (var i = 0; i < children.length; ++i) {
      createElm(children[i], insertedVnodeQueue, vnode.elm, null, true, children, i);
    }
  } else if (isPrimitive(vnode.text)) {
    nodeOps.appendChild(vnode.elm, nodeOps.createTextNode(String(vnode.text)));
  }
}
```

每个 VNode 都有一个 children 属性，用于包裹子元素节点。createChildren 函数通过判断 children 是否是数组列表，是的话次深度优先遍历递归调用 createElm 方法创建 DOM 节点。

接着调用 invokeCreateHooks 执行所有的 VNode create 钩子，并将 VNode push 到 insertedVnodeQueue 中。invokeCreateHooks 函数源码如下：

```js
function invokeCreateHooks (vnode, insertedVnodeQueue) {
    for (var i$1 = 0; i$1 < cbs.create.length; ++i$1) {
        cbs.create[i$1](emptyNode, vnode);
    }
    i = vnode.data.hook; // Reuse variable
    if (isDef(i)) {
        if (isDef(i.create)) { i.create(emptyNode, vnode); }
        if (isDef(i.insert)) { insertedVnodeQueue.push(vnode); }
    }
}
```

最后调用 `insert(parentElm, vnode.elm, refElm);` 将最终的 DOM 插到父节点中。注意，深度遍历递归的顺序是先子后父，故最终会回溯到挂载节点上。insert 源码如下：

```js
function insert (parent, elm, ref$$1) {
    if (isDef(parent)) {
        if (isDef(ref$$1)) {
            if (nodeOps.parentNode(ref$$1) === parent) {
                nodeOps.insertBefore(parent, elm, ref$$1);
            }
        } else {
            nodeOps.appendChild(parent, elm);
        }
    }
}
```

