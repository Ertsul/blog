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
const PENDING = 'pending'
const FULFILLED = 'fulfilled'
const REJECTED = 'rejected'

class MyPromise {
    value = null // 执行成功返回值
	onFulfilledCallbacks = [] // 成功状态：回调函数缓存队列
	reason = null // 执行失败原因
	onRejectedCallbacks = [] // 失败状态：回调函数缓存队列
}
```

#### 构造函数及其参数

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
const PENDING = 'pending'
const FULFILLED = 'fulfilled'
const REJECTED = 'rejected'

class MyPromise {
    constructor(executor) {
        executor()
    }
    
    value = null // 执行成功返回值
	onFulfilledCallbacks = [] // 成功状态：回调函数缓存队列
	reason = null // 执行失败原因
	onRejectedCallbacks = [] // 失败状态：回调函数缓存队列
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
const PENDING = 'pending'
const FULFILLED = 'fulfilled'
const REJECTED = 'rejected'

class MyPromise {
    constructor(executor) {
        executor(this.resolve, this.reject)
    }
    
    value = null // 执行成功返回值
	onFulfilledCallbacks = [] // 成功状态：回调函数缓存队列
	reason = null // 执行失败原因
	onRejectedCallbacks = [] // 失败状态：回调函数缓存队列

    resolve = () => {}
    reject = () => {}
}
// 使用
new MyPromise((resolve, reject) => {
    // 状态变更
    resolve()
})
```

#### resolve 和 reject

executor 函数通过 resolve 或者 reject 对内部状态进行变更。

- resolve: pendig -> fulfilled
- reject: pending -> rejected

实现代码：

```js
// 实现
const PENDING = 'pending'
const FULFILLED = 'fulfilled'
const REJECTED = 'rejected'

class MyPromise {
    constructor(executor) {
        executor(this.resolve, this.reject)
    }
    
    value = null // 执行成功返回值
	onFulfilledCallbacks = [] // 成功状态：回调函数缓存队列
	reason = null // 执行失败原因
	onRejectedCallbacks = [] // 失败状态：回调函数缓存队列

    resolve = (value) => {
        if (this.status === PENDING) {
            // 状态变更
            this.status = FULFILLED
            this.value = value
            // 执行缓存队列中的函数
            while(this.onFulfilledCallbacks.length) {
				this.onFulfilledCallbacks.shift()(value)
            }
        }
    }
    reject = (reason) => {
        if (this.status === PENDING) {
            // 状态变更
            this.status = REJECTED
            this.reason = reason
            // 执行缓存队列中的函数
            while(this.onFulfilledCallbacks.length) {
				this.onRejectedCallbacks.shift()(reason)
            }
        }
    }
}
```

#### then

then 方法主要执行以下逻辑：

- fulfilled 状态：执行 onFulfilled 函数
- rejected 状态：执行 onRejected 函数
- pending 状态：如 resolve 是在异步调用的场景，需要将 onFulfilled 和 onRejected 函数进行缓存

```js
// 实现
const PENDING = 'pending'
const FULFILLED = 'fulfilled'
const REJECTED = 'rejected'

class MyPromise {
    constructor(executor) {
        executor(this.resolve, this.reject)
    }
    
    status = PENDING
    value = null // 执行成功返回值
	onFulfilledCallbacks = [] // 成功状态：回调函数缓存队列
	reason = null // 执行失败原因
	onRejectedCallbacks = [] // 失败状态：回调函数缓存队列

    resolve = (value) => {
        if (this.status === PENDING) {
            // 状态变更
            this.status = FULFILLED
            this.value = value
            // 执行缓存队列中的函数
            while(this.onFulfilledCallbacks.length) {
				this.onFulfilledCallbacks.shift()(value)
            }
        }
    }
    reject = (reason) => {
        if (this.status === PENDING) {
            // 状态变更
            this.status = REJECTED
            this.reason = reason
            // 执行缓存队列中的函数
            while(this.onRejectedCallbacks.length) {
				this.onRejectedCallbacks.shift()(reason)
            }
        }
    }
    
    then(onFulfilled, onRejected) {
        switch(this.status) {
            case PENDING:
            	this.onFulfilledCallbacks.push(onFulfilled)
            	this.onRejectedCallbacks.push(onRejected)
            	break;
            case FULFILLED:
                onFulfilled(this.value)      
            	break;
            case REJECTED:
                onRejected(this.reason)  
            	break;
            default:
            	break;
        }
    }
}
```

至此，一个小型的 Promise 已经成型，使用下：

```js
// 同步
const p = new MyPromise((resolve, reject) => {
    resolve('aa')
})
p.then(res => {
    console.log(res); // aa
})

// 异步
const p1 = new MyPromise((resolve, reject) => {
    setTimeout(() => {
        resolve('bb')
    }, 10);
})
p1.then(res => {
    console.log('then1');
})
p1.then(res => {
    console.log("then2");
})
// then1 then2
```