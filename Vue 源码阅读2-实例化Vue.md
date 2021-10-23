## 实例化 Vue 对象

```js
new Vue({
  render: h => h(App)
})
```

判断是否是 Vue 的实例，是的话则执行初始化方法`_init`。源码如下：

```js
function Vue(options) {
  if (!(this instanceof Vue)
  ) {
    warn('Vue is a constructor and should be called with the `new` keyword');
  }
  this._init(options);
}
```

`_init`方法在`initMixin()`方法中已挂载到 Vue 原型。源码如下：

```js
function initMixin(Vue) {
    Vue.prototype._init = function (options) {
        // 合并 options
        if (options && options._isComponent) {
      	  // 优化内部组件实例化，因为动态选项合并非常慢，而且内部组件选项不需要特殊处理。
          initInternalComponent(vm, options);
        } else {
          vm.$options = mergeOptions(
            resolveConstructorOptions(vm.constructor),
            options || {},
            vm
          );
        }
        
        initLifecycle(vm);
        initEvents(vm);
        initRender(vm);
        callHook(vm, 'beforeCreate');
        initInjections(vm); // resolve injections before data/props
        initState(vm);
        initProvide(vm); // resolve provide after data/props
        callHook(vm, 'created');

        if (vm.$options.el) {
          vm.$mount(vm.$options.el);
        }
    }
}
```

主要执行 Vue 原型上挂载的实例初始化方法`_init`。该方法主要执行：

1. 合并传入 Vue 构造函数的 options，主要合并父级 options 和默认 options;
2. 初始化生命周期 initLifecycle；
3. 初始化事件中心 initEvents；
4. 初始化渲染 initRender；
5. 调用生命周期钩 beforeCreate；
6. 在 data\props 初始化之前执行 initInjections，初始化依赖 injections；
7. initState：初始化 props/methods/data/computed/watch 各项；
8. 在 data\props 初始化之后执行 initProvide，初始化注入 provide；
9. 调用生命周期钩子 created；
10. 执行 $mount 挂载 Vue 实例。

#### initLifecycle

- 往当前实例的所有父级的`$children`中添加当前实例作为子节点；
- 设置当前实例的父级、根节点和子节点；
- 初始化生命周期状态。

主要源码如下：

```js
function initLifecycle (vm) {
  var options = vm.$options;

  // 往当前实例的所有父级的`$children`中添加当前实例作为子节点
  var parent = options.parent;
  if (parent && !options.abstract) {
    while (parent.$options.abstract && parent.$parent) {
      parent = parent.$parent;
    }
    parent.$children.push(vm);
  }
    
  // 设置当前实例的父级、根节点和子节点
  vm.$parent = parent;
  vm.$root = parent ? parent.$root : vm;
  vm.$children = [];
  
  vm.$refs = {};
  vm._watcher = null;
  vm._inactive = null;
  vm._directInactive = false;
  vm._isMounted = false;
  vm._isDestroyed = false;
  vm._isBeingDestroyed = false;
}
```

#### initEvents

主要获取父级`$listeners`并更新组件的的`$listeners`属性。

主要源码如下：

```js
function initEvents (vm) {
  var listeners = vm.$options._parentListeners;
  if (listeners) {
    updateComponentListeners(vm, listeners);
  }
}

function updateComponentListeners (
  vm,
  listeners,
  oldListeners
) {
  target = vm;
  updateListeners(listeners, oldListeners || {}, add, remove$1, createOnceHandler, vm);
  target = undefined;
}
```

updateListeners 源码如下：

```js
function updateListeners (
  on,
  oldOn,
  add,
  remove$$1,
  createOnceHandler,
  vm
) {
  var name, def$$1, cur, old, event;
  for (name in on) {
    def$$1 = cur = on[name];
    old = oldOn[name];
    event = normalizeEvent(name);
    if (isUndef(cur)) {
      warn(
        "Invalid handler for event \"" + (event.name) + "\": got " + String(cur),
        vm
      );
    } else if (isUndef(old)) {
      if (isUndef(cur.fns)) {
        cur = on[name] = createFnInvoker(cur, vm);
      }
      if (isTrue(event.once)) {
        cur = on[name] = createOnceHandler(event.name, cur, event.capture);
      }
      add(event.name, cur, event.capture, event.passive, event.params);
    } else if (cur !== old) {
      old.fns = cur;
      on[name] = old;
    }
  }
  // 删除不存在的事件 listener
  for (name in oldOn) {
    if (isUndef(on[name])) {
      event = normalizeEvent(name);
      remove$$1(event.name, oldOn[name], event.capture);
    }
  }
}
```

- 在初始化阶段，oldListeners 为空对象，直接将 listeners 设置为父级 listeners。
- 在组件更新节点（在 updateChildComponent 中调用），会对新旧 listenters 比对并更新。
  - 遍历 on，如果就不在 oldOn，直接添加事件；如果在，则更新 oldOn 为 on;
  - 遍历 oldOn，删除不在 on 中的事件。

#### initRender

为当前 Vue 实例绑定 createElement 方法，这样就可以在实例中获取上下文。

#### initInjections

在 initState 初始化之前初始化 injections。

#### initState

初始化 props/methods/data/computed/watch 各项并添加双向数据绑定逻辑。

#### initProvide

在 initState 初始化之后初始化 provide。

#### 思考一：为什么要在 initState 初始化之前初始化 injections？而在 initState 初始化之后初始化 provide？

- 因为 provide 可能会用到 props/data 中的数据，故需要在 data\props 初始化之后；
- 因为 inject 中的数据会处理成 data，故需要先获取 inject 数据再处理 data\props。

![image.png](https://i.loli.net/2021/10/23/zm4sWoqOvNwpbJV.png)

看下面 Vue provide/inject 例子:

```js
// 父组件
export default {
  provide() { // 2. 后初始化 provide，才能使用 data 中的 pMsg 变量
    return {
      msg: this.pMsg
    }
  },
  name: 'Home',
  data() { // 1. 先初始化 data 
    return {
      pMsg: 'this is parent component'
    }
  }
}

// 子组件
export default {
  inject: [ // 1. 先初始化 inject
    'msg'
  ],
  data() { // 2. 后初始化 data
    return {}
  }
}
```

