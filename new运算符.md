## new 运算符

### 介绍

[new](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/new) 操作符的作用为：

> `new` **运算符**创建一个用户定义的对象类型的实例或具有构造函数的内置对象的实例。

 [mdn](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/new) 解释内部操作流程为：

- 创建一个空的简单 JavaScript 对象（即`{}`）；
- 为步骤 1 新创建的对象添加属性 **__proto__**，将该属性链接至构造函数的原型对象 ；
- 将步骤 1 新创建的对象作为`this`的上下文 ；
- 如果该函数没有返回对象，则返回`this`。

步骤 2 的作用是：使新创建的对象可以访问到构造函数原型上的属性和方法。

步骤 3  的作用使：使新创建的对象可以访问构造函数中的属性和方法。

### 模拟实现

```js
function createObject(Con, ...args) {
  // 创建新对象
  const obj = new Object()
  // 链接到构造函数原型，使新对象可以访问到构造函数原型上的属性和方法
  obj.__proto__ = Con.prototype
  // 改变 this 指向，使新对象可以访问到构造函数中的属性和方法
  const result = Con.apply(obj, args)
  // 返回执行结果，执行结果是对象则返回对象，否则返回 obj
  return typeof result === 'object' ? result : obj
}

function Person(name, age) {
  this.name = name
  this.age = age

  return {
    name,
    age
  }
}

// const person = new Person('Zigurn', '25')
const person = createObject(Person, 'Zigurn', 25)
console.log(person) // { name: 'Zigurn', age: 25 }
```