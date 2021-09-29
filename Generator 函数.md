[介绍链接](https://es6.ruanyifeng.com/#docs/generator)

总结几个要点：

- 函数定义：`function`关键字后加星号`*`;
- yield：暂停标识，代表代码执行到该位置；
- next 方法：遍历器`Iterator`指针指向下一个`yield`，也就是执行到下一个`yield`；
- next 方法返回值：当前`yield`执行结果和当前遍历器`Iterator`是否遍历结束。`{value: xxx, done: false / true}`

🌰1，定义和使用：

```js
// Genetator 函数定义
function* genFn() {
  yield 1
  yield 2
  yield 3
}
const gen = genFn()

console.log('yeild 1', gen.next()) // yeild 1 { value: 1, done: false }
console.log('yeild 2', gen.next()) // yeild 2 { value: 2, done: false }
console.log('yeild 3', gen.next()) // yeild 3 { value: 3, done: false }
```

🌰2，`next`方法传参：

向`next`方法传参会将实参的值作为上次`yield`的结果。

```js
// Genetator 函数定义
function* genFn() {
  const res01 = yield 1
  console.log('res01', res01); // res01 100（这里 next 参数为 100，故上次 yield 的执行结果为 100）
  const res02 = yield res01
  console.log('res02', res02); // res02 200
  const res03 = yield res02
  console.log('res03', res03); // res03 undefined
}
const gen = genFn()

console.log('yield 1', gen.next()) // yield 1 { value: 1, done: false }
console.log('yield 2', gen.next(100)) // yield 2 { value: 100, done: false }
console.log('yield 3', gen.next(200)) // yield 3 { value: 200, done: false }
console.log('last', gen.next()) // last { value: undefined, done: true }
```

🌰3，异步函数：

```js
// 异步函数
function timer(num) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      return resolve(num + 1)
    }, 10)
  })
}

// Genetator 函数定义
function* genFn() {
  const res01 = yield timer(1)
  const res02 = yield timer(res01)
  const res03 = yield timer(res02)

  return res03
}

const gen = genFn()

console.log('yield 1', gen.next()) // yield 1 { value: Promise { <pending> }, done: false }
console.log('yield 2', gen.next()) // yield 2 { value: Promise { <pending> }, done: false }
console.log('yield 3', gen.next()) // yield 3 { value: Promise { <pending> }, done: false }
```

由于每个`yield`的结果都是一个`Promise`，所以如果要输出每次 `yield `的结果，需要进行修改。

修改下，打印出每次`Promise`的结果：

```js
const gen = genFn()

gen.next().value.then((res1) => {
  console.log('res1', res1) // 2

  gen.next(res1).value.then((res2) => {
    console.log('res2', res2) // 3

    gen.next(res2).value.then((res3) => {
      console.log('res3', res3) // 4
      return res03
    })
  })
})
```

上面这样有点 callback hell 的感觉，改造下，解决回调地狱的问题。

参考`async/await`的使用：

- `async`函数返回值是一个`Promise `；
- 异步函数同步化使用；

```js
// 异步函数
function timer(num) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      return resolve(num + 1)
    }, 10)
  })
}

async function fn() {
  const res01 = await timer(1)
  const res02 = await timer(res01)
  const res03 = await timer(res02)

  return res03
}

fn().then((res) => console.log(res))
```

将`Generator`函数转化为类似`async/await`格式：

```js
function generator2async(generatorFn) {
  return function () {
    const gn = generatorFn.apply(this, arguments)
    return new Promise((resolve, reject) => {
      function exec(arg) {
        let res
        try {
          res = gn.next(arg)
        } catch (error) {
          return reject(error)
        }
        const { value, done } = res
        // 遍历器执行结束
        if (done) {
          return resolve(value)
        }
        return Promise.resolve(value)
          .then((val) => exec(val)) // 执行下一次
          .catch((err) => {
            // console.log(err)
            return reject(err)
          })
      }
      exec() // 第一次执行
    })
  }
}

// 异步函数
function timer(num) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      return resolve(num + 1)
    }, 10)
  })
}

// Genetator 函数定义
function* genFn() {
  const res01 = yield timer(1)
  const res02 = yield timer(res01)
  const res03 = yield timer(res02)

  return res03
}

const aysncedFn = generator2async(genFn)

aysncedFn().then((res) => console.log(res)) // 4
```

