## EC、ECS、VO、AO、Scope Chain

题目：

```javascript
var a = 1, b = 2;
function fn() {
	var a = 100, b = 200;
	console.log(a + b)
}
fn()
```

一道很常见的 javascript 题目，输出`300`。

从这道题一出发，捋一遍 EC、ECS、VO、AO、Scope Chain 相关知识点。

### EC

执行上下文（Execution Context），也叫执行环境。

在《javascript高级程序设计》中的定义：

> 定义了变量或函数有权访问的其他数据，决定了它们各自的行为。

即：EC 决定了当前代码对其他变量或函数的访问权限。

javascript 中会维护一个`执行上下文栈 ECS（Execution Context Stack）`。

- 栈底是全局上下文（Global Context）；
- 当一个函数执行的时候，会将当前函数的 EC push 进 ECS，函数执行结束 pop 出 ECS。

`EC` 主要包含三个成员：

- `VO`：变量对象
- `Scope Chain`：作用域链
- `this`

### VO

变量对象（Variable Object）

作用：

- 保存 EC 的作用域链
- 保存 EC 的 arguments 和变量

可以理解为数据作用域。

当函数执行的时候，进入函数的 EC，VO 激活为活动对象 AO。当函数开始执行前，VO 进行初始化，包括 arguments 和内部的变量；当函数真正执行代码会将修改 VO 中的 arguments 和内部的变量。

比如上面的题目：

```javascript
var a = 1, b = 2;
function fn() {
	var a = 100, b = 200;
	console.log(a + b)
}
fn()
```

函数 fn 在开始执行前，会初始化 VO 的 arguments 和内部的变量。如下：

```javascript
const fnVO = {
	arguments: {
        length: 0
    },
    a: undefined,
    b: undefined
}
```

当函数开始执行，修改 VO 的 arguments 和内部的变量。如下：

```javascript
const fnVO = {
	arguments: {
        length: 0
    },
    a: 100,
    b: 200
}
```

### Scope Chain

作用域链。

在《javascript高级程序设计》中的解释：

>作用域链的用途，是保证对执行环境有权访问的所有变量和函数的有序访问。作用域链的前端，始终都是当前执行的代码所在环境的变量对象。

当在 EC 中查找一个变量，会先在当前作用域中进行查找，查找不到就向父级作用域查找，如此循环，直到找到为止；如果找不到，就返回 undefined。

在作用域链的最外级，是全局作用域 `Global Context`	。

当函数创建的时候，函数内部有个属性 `[[scope]]`保存父级 EC 的作用域链；当函数开始执行，先创建函数的 EC，推入 ECS；然后将`[[scope]]`保存的作用域复制到 EC 中进行保存，接着初始化 VO 的 arguments 和变量，最后将当前 VO 添加到保存的作用域最前端，这样就形成了作用域链。

### 题目流程

结合 EC、ECS、VO、AO、Scope Chain，解析题目。

```javascript
var a = 1, b = 2;
function fn() {
	var a = 100, b = 200;
	console.log(a + b)
}
fn()
```

流程为：

- 函数创建的时候，函数的内部属性`[[scope]]`会保存外部 EC 的作用域链；

```javascript
fn.[[scope]] = [
    GlobalContext.VO
]
```

- 函数开始执行前，先创建函数的 EC，推入 ECS；

```javascript
ECStack = [
    fnContext,
    globalContext
];
```

- 将`[[scope]]`保存的作用域复制到函数执行上下文 EC 中进行保存；

```javascript
const fnContext = {
	Scope: fn.[[scope]]
}
```

- 初始化函数 VO 的 arguments 和变量；

```javascript
const fnContext = {
	Scope: fn.[[scope]],
    fnVO: {
	arguments: {
        length: 0
    },
    a: undefined,
    b: undefined
}
```

- 将函数的 VO 添加到作用域的前端；

```javascript
const fnContext = {
	Scope: [fnVO, [[Scope]]],
    fnVO: {
        arguments: {
            length: 0
        },
        a: undefined,
        b: undefined
    }
}
```

- 函数执行，修改 VO 对象；

```javascript
const fnContext = {
	Scope: [fnVO, [[Scope]]],
    fnVO: {
        arguments: {
            length: 0
        },
        a: 100,
        b: 200
    }
}
```

- 函数执行结束， EC 出 ECS。

```javascript
ECStack = [
    globalContext
];
```

---

参考链接：

- https://github.com/mqyqingfeng/Blog/issues/7
- https://javascript.info/reference-type