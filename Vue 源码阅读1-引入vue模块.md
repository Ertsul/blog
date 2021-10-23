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

主要 Vue 原型上挂载出实例初始化方法`_init`。

#### stateMixin

这一部分主要处理：数据劫持 + 双向数据绑定。

#### eventsMixin

在 Vue 原型上挂载全局方法：`$on`、`$once`、`$off`、`$emit`。

#### lifecycleMixin

在 Vue 原型上挂载跟 Vue 实例生命周期有关的方法：`_update`、`$forceUpdate`、`$destroy`。

#### renderMixin

在 Vue 原型上挂载跟渲染有关的方法：`$nextTick`、`_render`。