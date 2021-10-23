## 引入 Vue

```js
// 便于调试，改成：
import Vue from 'vue/dist/vue.runtime.common.dev.js'
```

引入 Vue 对象，主要执行：

- 往 Vue 函数/对象的原型上挂载初始化方法、全局方法和挂载 Vue 静态方法，做准备工作；
- 导出 Vue 模块。

主要源码如下：

```js
initMixin(Vue);
stateMixin(Vue);
eventsMixin(Vue);
lifecycleMixin(Vue);
renderMixin(Vue);

initGlobalAPI(Vue);

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

#### initGlobalAPI

直接往 Vue 上挂载初始化全局 API 静态方法（非原型上挂载），可通过 Vue 直接调用：`set`、`delete`、`nextTick``、observable`、`use`、`mixin`、`component`、`directive`、`filter`。

