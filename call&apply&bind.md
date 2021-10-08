## call、apply 、bind

### 作用

这几个函数的最主要作用就是：**改变 this 指向**。

基于这个作用，可以延伸出很多额外的作用：

- 合并数组

```js
let arr1 = [1, 2, 3]
let arr2 = [4, 5, 6]
// 合并 arr2 到 arr1
Array.prototype.push.apply(arr1, arr2)
console.log(arr1) // [ 1, 2, 3, 4, 5, 6 ]
```

- 获取数组中的最大值和最小值

```js
const maxNum = Math.max.apply(Math, [1, 2, 3, 4])
console.log(maxNum) // 4 
```

- 类数组使用数组方法

```js
const arrLike = {
  0: 'a',
  1: 'b',
  length: 2
}
// arrLike.push('c') // TypeError: arrLike.push is not a function
Array.prototype.push.call(arrLike, 'c')
console.log(arrLike) // { '0': 'a', '1': 'b', '2': 'c', length: 3 }
```

- ...

另外，bind 还能用于柯里化。

```js
function add(num1) {
  return function (num2) {
    return num1 + num2
  }
}

const sum = add(1)(2)
console.log(sum) // 3
```

### 区别

call 和 apply 主要区别在于传递参数方式不同：

- call：多个参数的列表
- apply：一个包含多个参数的数组

bind 跟 call/apply 不同点主要在于：bind 返回值是一个函数。

### 模拟实现

#### call

```js
// es3
Function.prototype.call1 = function (context) {
  context = context || window // 获取目标绑定对象
  context.fn = this // 执行函数
  // 获取函数参数
  var args = []
  for (var i = 1; i < arguments.length; i++) {
    args.push('arguments[' + i + ']')
  }
  var result = eval('context.fn(' + args + ')') // 执行函数
  delete context.fn
  return result
}
// es6
Function.prototype.call2 = function (context, ...args) {
  context = context || window // 获取目标绑定对象
  context.fn = this // 执行函数
  const result = context.fn(...args) // 执行函数
  delete context.fn
  return result
}
```

#### apply

```js
// es3
Function.prototype.apply1 = function (context, arr) {
  context = context || window // 获取目标绑定对象
  context.fn = this // 执行函数
  var result
  if (!arr || !arr.length) {
    // 无函数参数
    result = context.fn()
  } else {
    // 获取函数参数
    var args = []
    for (var i = 0; i < arr.length; i++) {
      args.push('arr[' + i + ']')
    }
    result = eval('context.fn(' + args + ')') // 执行函数
  }
  delete context.fn
  return result
}
// es6
Function.prototype.apply2 = function (context, arr) {
  context = context || window // 获取目标绑定对象
  context.fn = this // 执行函数
  let result
  if (!arr || !arr.length) {
    // 无函数参数
    result = context.fn()
  } else {
    result = context.fn(...arr) // 执行函数
  }
  delete context.fn
  return result
}
```

#### bind

```js
Function.prototype.bind1 = function (context) {
  if (typeof this !== 'function') {
    throw new Error(
      'Function.prototype.bind - what is trying to be bound is not callable'
    )
  }

  context = context || window
  var self = this // 执行函数
  // 保存第一层参数
  var args = Array.prototype.slice.call(arguments, 1) // arguments 是类数组，需要通过 Array.prototype.slice.call 获取函数参数

  var noopFn = function () {} // 空函数，用于修改返回的执行函数 boundFn 的 prototype
  var boundFn = function () {
    // 返回的函数
    // 获取第二层参数
    var boundArgs = Array.prototype.slice.call(arguments, 1)
    return self.apply(
      this instanceof noopFn ? this : context, // 判断是否为实例化对象
      args.concat(boundArgs)
    )
  }

  // 修改返回的执行函数 boundFn 的 prototype 为执行函数的 prototype
  // 这样实例就可以继承绑定函数原型的属性和方法
  noopFn.prototype = this.prototype
  boundFn.prototype = new noopFn()

  return boundFn
}
```