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
            while(this.onRejectedCallbacks.length) {
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
        onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : value => value
        onRejected = typeof onRejected === 'function' ? onRejected : reason => { throw reason }
        
        switch(this.status) {
            case PENDING:
                // resolve 或者 reject 在异步函数中调用，需要缓存起来
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

#### then 链式调用

then 链式调用实际是在 then 函数内部又返回一个新的 Promise，通过对上一个 then 返回值的判断，将值穿透到下一个 then 中。

修改 then 方法：

```js
// ...

class MyPromise {
	// ...
    
    then(onFulfilled, onRejected) {
        onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : value => value
        onRejected = typeof onRejected === 'function' ? onRejected : reason => { throw reason }

        const p2 = new MyPromise((resolve, reject) => {
            // 封装上一个 then 的 onFulfilled
            const fulfilled = () => {
                queueMicrotask(() => {
                    // 放入微任务，不然 p2 还未完成初始化
                    const x = onFulfilled(this.value)
                    resolvePromise(p2, x, resolve, reject)
                })
            }
            // 封装上一个 then 的 onRejected
            const rejected = () => {
                queueMicrotask(() => {
                    // 放入微任务，不然 p2 还未完成初始化
                    const x = onRejected(this.reason)
                    resolvePromise(p2, x, resolve, reject)
                })
            }

            switch (this.status) {
                case PENDING:
                    // resolve 或者 reject 在异步函数中调用，需要缓存起来
                    this.onFulfilledCallbacks.push(fulfilled)
                    this.onRejectedCallbacks.push(rejected)
                    break;
                case FULFILLED:
                    fulfilled()
                    break;
                case REJECTED:
                    rejected()
                    break;
                default:
                    break;
            }
        })

        return p2
    }
}


// 该函数用于获取上一个 then 的返回值，并更改新 MyPromise 的状态
function resolvePromise(p2, x, resolve, reject) {
    // 判断返回值是否为自身，防止循环调用
    if (p2 === x) {
        throw new Error('cycle promise call')
    }
    
    if (typeof x === 'function' || typeof x === 'object') {
        if (x === null) {
            return resolve(x)
        }
        
        // 获取 then 函数
        let then
        try {
          then = x.then  
        } catch {
          reject(err)
        }
        
        // then 是一个函数
        if (typeof then === 'function') {
            let called = false // 是否调用标志，防止重复调用
            try {
                then.call(x, res => {
                    if(called) return
                    called = true
                    resolvePromise(p2, res, resolve, reject)
                }, err => {
                    if(called) return
                    called = true
                    reject(err)
                })
            } catch {
                if(called) return
                called = true
                reject(err)
            }
        } else{
            // 对象，直接执行
            resolve(x)
        }
    } else {
        // 普通值，直接 resolve，变更 p2 的状态
        resolve(x)
    }
}
```

执行代码，测试链式调用：

```js
const p = new MyPromise((resolve, reject) => {
    setTimeout(() => {
        resolve('aa')
    }, 0);
})
p.then(res => {
    console.log('then1', res);
    return 'bb'
}).then(res => {
    console.log('then2', res);
    return new MyPromise((resolve, reject) => {
        resolve('cc')
    })
}).then(res => {
    console.log('then3', res);
})
// then1 aa
// then2 bb
// then3 cc
```

执行代码，测试返回值为自身：

```js
const promise = new MyPromise((resolve, reject) => {
    resolve('success')
})

const p1 = promise.then(value => {
    console.log(1)
    return p1
})

// 运行的时候会走reject
p1.then(value => {
    console.log(2)
}, reason => {
    console.log(3)
})
// 1
//  cycle promise call
```

#### 完善代码，添加 try/catch

```js
// 实现
const PENDING = 'pending'
const FULFILLED = 'fulfilled'
const REJECTED = 'rejected'

class MyPromise {
    constructor(executor) {
        try {
            executor(this.resolve, this.reject)
        } catch (error) {
            this.reject(error)
        }
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
        while (this.onFulfilledCallbacks.length) {
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
        while (this.onRejectedCallbacks.length) {
            this.onRejectedCallbacks.shift()(reason)
        }
    }
}

then(onFulfilled, onRejected) {
    onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : value => value
    onRejected = typeof onRejected === 'function' ? onRejected : reason => { throw reason }

    const p2 = new MyPromise((resolve, reject) => {
        // 封装上一个 then 的 onFulfilled
        const fulfilled = () => {
            try {
                queueMicrotask(() => {
                    // 放入微任务，不然 p2 还未完成初始化
                    const x = onFulfilled(this.value)
                    resolvePromise(p2, x, resolve, reject)
                })
            } catch (error) {
                reject(error)
            }
        }
        // 封装上一个 then 的 onRejected
        const rejected = () => {
            try {
                queueMicrotask(() => {
                    // 放入微任务，不然 p2 还未完成初始化
                    const x = onRejected(this.reason)
                    resolvePromise(p2, x, resolve, reject)
                })
            } catch (error) {
                reject(error)
            }
        }

        switch (this.status) {
            case PENDING:
                // resolve 或者 reject 在异步函数中调用，需要缓存起来
                this.onFulfilledCallbacks.push(fulfilled)
                this.onRejectedCallbacks.push(rejected)
                break;
            case FULFILLED:
                fulfilled()
                break;
            case REJECTED:
                rejected()
                break;
            default:
                break;
        }
    })

    return p2
}
}

function resolvePromise(p2, x, resolve, reject) {
    // 判断返回值是否为自身，防止循环调用
    if (p2 === x) {
        throw new Error('cycle promise call')
    }

    if (typeof x === 'function' || typeof x === 'object') {
        if (x === null) {
            return resolve(x)
        }

        // 获取 then 函数
        let then
        try {
            then = x.then
        } catch (err) {
            return reject(err)
        }

        // then 是一个函数
        if (typeof then === 'function') {
            let called = false // 是否调用标志，防止重复调用
            try {
                then.call(x, res => {
                    if (called) return
                    called = true
                    resolvePromise(p2, res, resolve, reject)
                }, err => {
                    if (called) return
                    called = true
                    reject(err)
                })
            } catch {
                if (called) return
                called = true
                reject(err)
            }
        } else {
            // 对象，直接执行
            resolve(x)
        }
    } else {
        // 普通值，直接 resolve，变更 p2 的状态
        resolve(x)
    }
}
```

