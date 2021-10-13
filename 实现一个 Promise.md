#### 三个状态

- pending：等待状态
- fulfilled：执行成功状态
- rejected：执行失败状态

其中，状态流程为：`pending->fulfilled` 或者 `pending->rejected`

代码如下：

```js
const PENDING = 'pending'
const FULFILLED = 'fulfilled'
const REJECTED = 'rejected'
```

#### MyPromise 类

Promise 是一个类，使用时需要 new 一个 Promise 的实例。

```js
class MyPromise {
    value = null // 执行成功返回值
	onFulfilledCallbacks = [] // 成功状态：回调函数缓存队列
	reason = null // 执行失败原因
	onRejectedCallbacks = [] // 失败状态：回调函数缓存队列
}
```

### 构造函数及其参数

Promise 使用代码：

```js
new Promise((resolve, reject) => {
	resolve()   
})
```

new 一个 Promise 的实例的时候，会执行构造函数，同时也会立即执行传入的函数。

MyPromise 代码：

```js
// 实现
class MyPromise {
    constructor(executor) {
        executor()
    }
}
// 使用
new MyPromise((resolve, reject) => {
    // 状态变更
    resolve()
})
```

- new MyPromise 实例，传入执行函数实参赋值给形参 executor
- 形参 executor 获取实参值后立即执行
- executor 函数参数 resolve 和 reject 通过 MyPromise 内部类成员变量进行赋值

```js
// 实现
class MyPromise {
    constructor(executor) {
        executor(this.resolve, this.reject)
    }
    resolve = () => {}
    reject = () => {}
}
// 使用
new MyPromise((resolve, reject) => {
    // 状态变更
    resolve()
})
```



