## 引入 Vue

```js
// 便于调试，改成：
import Vue from 'vue/dist/vue.runtime.common.dev.js'
```

引入 vue 对象，主要执行：

- 往 vue 函数/对象的原型上挂载初始化方法和全局方法，做准备工作；
- 导出 Vue 模块。

主要源码如下：

```js
initMixin(Vue);
stateMixin(Vue);
eventsMixin(Vue);
lifecycleMixin(Vue);
renderMixin(Vue);

module.exports = Vue;
```

#### initMixin 函数

主要源码如下：

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

主要 Vue 原型上挂载出实例初始化方法`_init`，该方法主要执行：

- 合并传入 Vue 构造函数的 options，主要合并父级 options 和默认 options;
- 初始化生命周期 initLifecycle；
- 初始化事件中心 initEvents；
- 初始化渲染 initRender；
- 调用生命周期钩 beforeCreate；
- 在 data\props 初始化之前，初始化依赖 injections；
- initState：初始化 props/methods/data/computed/watch 各项；
- 在 data\props 初始化之后，初始化注入 provide；
- 调用生命周期钩子 created；
- 执行 $mount 挂载 Vue 实例。

##### 思考一：为什么要在 initState 初始化之前初始化 injections？而在 initState 初始化之后初始化 provide？

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

#### stateMixin

这一部分主要处理：数据劫持 + 双向数据绑定。

#### eventsMixin

在 Vue 原型上挂载全局方法：`$on`、`$once`、`$off`、`$emit`。

#### lifecycleMixin

在 Vue 原型上挂载跟 Vue 实例生命周期有关的方法：`_update`、`$forceUpdate`、`$destroy`。

#### renderMixin

在 Vue 原型上挂载跟渲染有关的方法：`$nextTick`、`_render`。