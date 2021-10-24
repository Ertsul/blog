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



