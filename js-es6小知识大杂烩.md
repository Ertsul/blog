---

#### 合并两个数组
- 数组的解构 **[...arr1, ...arr2]**
- Array.prototype.push.apply(arr1, arr2)，最后合并到arr1。
- arr3 = Array.prototype.slice.call(arr1, arr2)，最后合并到arr3。

```
let arr1 = [1, 2, 3];
let arr2 = [4, 5, 6];
let arr3 = [...arr1,  ...arr2];
console.log(arr3);
Array.prototype.push.apply(arr1,  arr2);
console.log(arr1);
```

#### arguments/伪数组 转化为数组
- [...arguments]
- Array.prototype.slice.call(arguments)

```
(function Fn() {
    let arr = [...arguments, 'Ertsul'];
    console.log(arr);
})('zero');
(function Fn1() {
    let arr = Array.prototype.slice.call(arguments);
    console.log(arr);
})('zero');
```

#### 判断字符类型
- Object.prototype.toString.call()
- typeof
- instanceof

#### 创建实例的方法
- 字面量 let obj = {...}
- new Object()构造函数
- 构造函数
- Object.create()
- 工厂模式

```
// 字面量
let obj1 = {
    name: 'zero',
    age: 22
};
// Object 构造函数
let obj2 = new Object();
obj2.name = 'zero';
console.log(obj2.name);
// 工厂模式
function Person(name) {
    let obj = new Object();
    obj.name = name;
    return obj;
}
let p1 = Person('zero');
console.log(p1.name);
// 构造函数
function Animal(name) {
    this.name = name;
}
let a2 = new Animal('dog');
console.log(a2.name);
```

#### 数组的拆解 flat
```
let arr2 = [1, [2, 3, [100, 8989]], [4, 5, 6, 7, 8]];
const flat = arr => arr.toString().split(',).map(item => +item);
console.log(flat(arr2)); // [1, 2, 3, 100, 8989, 4, 5, 6, 7, 8]
```

#### 继承的方法 -- 8种
- 通过原型继承
  - 缺点：引用类型存在共享问题。

````
function Fn1() { ... }
function Fn2() { ... }
Fn2.prototype = new Fn1();
````

- 构造函数：通过 **call** 更改 **this** 指向；实际上是调用了父类的构造函数。
  - 缺点：父类中的方法（构造函数中）子类不可见。

````
function Fn1() { ... }
function Fn2() { 
  Fn1.call(this);   // 将 this 绑定到 Fn1
}
````

````
function SuperType(){
    
}
SuperType.prototype.sayHi = function(){
    console.log('Hi');
}

function subType(){
    SuperType.call(this);
}

let superIns = new SuperType();
superIns.sayHi();   // Hi

let subIns = new subType();
subIns.sayHi();     // Uncaught TypeError
````

- 组合继承（原型 + 构造）
属性通过构造函数继承；方法通过原型继承。记得要更改 **Fn2.prototype.constructor** 的指向，指向子类。

````
// 组合继承
function Fn1() {
    this.name = 'zero'
}

Fn1.prototype.sayName = function () {
    console.log(this.name);
};

function Fn2() {
    // 继承属性
    Fn1.call(this);
    this.age = 22;
}
// 继承方法
Fn2.prototype = new Fn1();
Fn2.prototype.constructor = Fn2;   // 需要修复构造函数指向

Fn2.prototype.sayAge = function () {
    console.log(this.age);
};

let f = new Fn2();
f.sayName();
f.sayAge();
````

- 实例继承
在一个函数内实例化，然后添加新属性，并返回该对象。

````
function Fn() { ... }
function Fn2(name) {
 let obj = new Fn();
 obj.name = name;
 return obj;
}
````

- 原型式继承
将父类作为函数的参数，创建一个临时函数，继承于父类，返回父类。

````
function SuperType(name) {
    this.name = name
}

function Object(o){
    function F(){}
    F.prototype = o;
    return new F();
}

let subType = Object(new SuperType('zero'));
console.log(subType.__proto__); // SuperType 因为是实例，所以只能是 __proto__
````

- 拷贝继承
将原型/父类上的属性全部拷贝到子类上。

````
// 拷贝继承
function Fn1() {
    this.name = 'zero';
    this.age = 22
}
function Fn2() {
    let f1 = new Fn1();
    // 注意这里要用 in 操作符
    for (let item in f1) {
        Fn2.prototype[item] = f1[item];
    }
    Fn2.prototype.ownFun = function () {
        console.log('my own function.');
    }
}

let f = new Fn2();
console.log(f.name + ", " + f.age);
f.ownFun();
````

- 寄生组合式模型
  - 通过一个函数（创建对象，增强对象没，指定对象）：实现父类子类的方法继承。
  - 通过构造函数实现属性的继承。

````
// 寄生组合式继承
function inheritPrototype(subType, superType) {
    // 创建对象：新建的对象指向父类的原型
    let prototype = Object(superType.prototype);
    // 增强对象：新建对象的 constructor 指向子类
    prototype.constructor = subType;
    // 指定对象：子类的原型指向新建对象
    subType.prototype = prototype;
}

function Fn1() {
    this.name = 'Ertsul';
}

Fn1.prototype.sayName = function () {
    console.log(this.name);
};

function Fn2() {
    Fn1.call(this);
    this.age = 22;
}

inheritPrototype(Fn2, Fn1);

Fn2.prototype.sayAge = function () {
    console.log(this.age);
};

let f = new Fn2();
f.sayName();f.sayAge();
````

- es6 的 extends 

````
// class
class F1 {
    constructor(name) {
        this.name = name;
    }
}

class F2 extends F1 {
    constructor(name, age){
        super(name);    // 调用父类的构造函数
        this.age = age;
    }
    showMsg(){
        console.log(this.name + ', ' + this.age);
    }
}

let p = new F2('zero', 22);
p.showMsg();  // zero, 22
````

#### 比较两个对象是否相等
- 遍历对象对象进行判断。
- 将对象转化为字符串进行判断。

````
let obj1 = {
    name: 'a',
    age: 1
}
let obj2 = obj1;
let obj3 = {
    name: 'a',
    age: 2
}
console.log(JSON.stringify(obj1));
console.log(JSON.parse(JSON.stringify(obj1)));
console.log(JSON.stringify(obj1) == JSON.stringify(obj2));  // true
console.log(JSON.stringify(obj1) == JSON.stringify(obj3));  // false    
````

#### 封装一个时间函数，时间通过 time 传递，回调函数通过 then 函数执行

````
function time(time){
    return new Promise((resolve, reject) => {
        setTimeout(resolve, time);
    })
}

let t = new time(5000);
t.then((a, b) => {
    console.log('time');
})
````

#### 原型相关的API
- Person.prototype.isPrototypeOf(p1)：判断是否属于某个实例
- Object.getPrototypeOf(p1)：获取原型
- p1.hasOwnProperty('name')：判断某个属性是否属于某个实例（只能获取属性实例）
- in操作符：判断某个属性是否属于某个实例（不管是实例中还是原型中）

````
function Person(name, age) {
    this.name = name;
    this.age = age;
}

let p1 = new Person('Ertsul', 22);
console.log(Person.prototype);      // constructor f
console.log(Person.prototype.constructor);  // Person
console.log(Person.prototype.__proto__);    // constructor 的原型
console.log(p1.__proto__);      // // constructor f

console.log(Person.prototype.isPrototypeOf(p1));    // true
console.log(Person.isPrototypeOf(p1));      // false
console.log(Object.getPrototypeOf(p1));     // constructor f

console.log(p1.hasOwnProperty('name'));     // true 存在实例中

Person.prototype.num = 100

console.log(p1.hasOwnProperty('num'));      // fasle    存在原型中

console.log('num' in p1);   // true 不管是实例还是原型中
````

#### js 判断数组类型
- arr instanceof Array
- arr.constructor === Array

````
let arr = [1, 2, 3];
let obj = {}
function judgeFn(arr){
    // return arr.constructor === Array ? true : false
    return arr instanceof Array ? true : false
}
console.log(judgeFn(arr));  // true
console.log(judgeFn(obj )); // false
````

#### 日期格式化
````
/* 日期格式化 */
function formatNumber(num) {
	const n = `${num}`;		// 转化为字符串
	return n[1] ? n : `0${n}`;
}
function formatTime(date){
	// 年 月 日
	const year = date.getFullYear();
	const month = date.getMonth() + 1;
	const day = date.getDate();
	// 时 分 秒
	const hour = date.getHours();
	const minute = date.getMinutes();
	const second = date.getSeconds();

	return `${[year, month, day].map(formatNumber).join('-')} ${[hour, minute, second].map(formatNumber).join(':')}`
}
let time = new Date();
console.log(time);		// Tue Aug 14 2018 16:52:06 GMT+0800 (中国标准时间)
let t1 = formatTime(time);
console.log(t1);		// 2018-08-14 16:52:06
````

#### 字符串转化为标准日期格式
````
/* 字符串转化为标准日期格式 */
function time2string(str) {
	const chunks = str.split(' ');
	const date = chunks[0].split('-');
	const time = chunks[1].split(':');

	return new Date(date[0], date[1] - 1, date[2], time[0], time[1], time[2]);
}
let str = "2018-08-14 23:59:59";
let t2 = time2string(str);
console.log(t2);	// Tue Aug 14 2018 23:59:59 GMT+0800 (中国标准时间)
````
#### 防抖节流
````
/* 防抖节流 debounce throttle */
// 防抖 - 连续快速触发的解决方案
function debounce(fn, wait) {
	let timeout = null;	// 初始化 timer 定时器
	return function (){
		timeout && clearTimeout(timeout);	// 清空定时器
		timeout = setTimeout(fn, wait);		// 设置定时器
	}
}
// 节流 - 一定时间内请求一次 -- 定时器法
function throttle1(fn, wait) {
	let timer = null;	// 初始化 timer 定时器
	return function () {
		let context = this;		// 保存上下文 this
		let args = arguments;	// 记录参数
		if(!timer) {	// 定时器为空
			timer = setTimeout(() => {
				fn.apply(context, args);
				timer = null;	// 清空定时器
			}, wait);
		}
	}
}
// 节流 - 一定时间内请求一次 -- 时间戳法
function throttle2(fn, wait) {
	let prev = Date.now();	// 记录前一个时间
	return function () {
		let context = this;		// 保存上下文 this
		let args = arguments;	// 记录参数
		let now = Date.now(); 	// 记录当前时间
		if(now - prev >= wait){		// 时间戳大于设置的时间
			fn.apply(context, args);
			prev = Date.now();	// 记录前一个时间
		}
	}
}

// window.addEventListener('scroll', debounce(() => {
 //    console.log(Math.random());
// }, 500));
// window.addEventListener('scroll', throttle1(() => {
// 	console.log(Math.random());
// }, 1000));
window.addEventListener('scroll', throttle2(() => {
	console.log(Math.random());
}, 1000))
````
- JSON.stringfy(obj, [replace, space])
  - 巧用第二个参数，可以实现json对象的过滤替换。
  - 第三个参数，是缩进的空格数。    
  - 如果被序列化对象内部含有toJSON方法，则该对象不会被序列化，而是序列化toJSON方法的返回值。

```
let obj1 = {
    name: 'zero',
    age: 22,
    hobby: 'ball'
};

let str1 = JSON.stringify(obj1);
console.log(str1, typeof str1, str1.constructor === String);    // {"name":"zero","age":22,"hobby":"ball"} string true
let str2 = JSON.stringify(obj1, (key, value) => {
    if(value === 'zero') {
        return 'Ertsul.'
    }else {
        return value;
    }
});
console.log(str2);  // {"name":"Ertsul.","age":22,"hobby":"ball"}

let obj2 = {
    num: 100,
    toJSON(){
        return 'apple'
    }
};

let str3 = JSON.stringify(obj2);
console.log(str3);  // apple
```

- 数组的 sort()
sort()默认会将每个元素转化为字符串，然后按照字符串进行排序，是不稳定的排序。
如果要对数组的数字进行排序，则需要：
>array.sort((val1, val2) => {return val - val2});

通过这样实现，因为传递的参数是数字，所以通过减法操作（字符进行减法操作返回的是NaN），可以得到预期结果。


```
let arr10 = [1, 2, 19, 12, 22];
console.log(arr10.sort()); // [1, 12, 19, 2, 22]
console.log(arr10.sort((val1, val2) => {  // [1, 2, 12, 19, 22]
  return val1 - val2;
}));  
```

- reduce函数
> reduce() 方法对累加器和数组中的每个元素（从左到右）应用一个函数，将其简化为单个值。

其完整的函数为： 


```
arr.reduce((prev, next, cur, srcArr) => {
  ......
}, initVal)
```

- prev: 上次返回的值
- next: 下一个数组元素的值
- cur: 当前数组元素的索引值
- scrArr: 源数组
- initVal: 设置第一次的 **prev**

例子 1：

```
let arr = ['apple', 'pear', 'bananas'];
arr.reduce((prev, next, cur, arr) => {
  console.warn(prev, next, cur, arr);
  return next;
}, 'fruit')
```

![image.png](http://upload-images.jianshu.io/upload_images/659084-65c2bd1bf155dfaf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

例子 2：统计一个数组中单词出现的次数

```
// 统计数组中单词出现的次数
let arr = ["apple", "orange", "apple", "orange", "pear", "orange", "bananas", "bananas"];
function getCount(arr) {
  return arr.reduce((prev, next, cur) => {
    prev[next] = (prev[next] + 1) || 1
    return prev;
  }, {})
}
console.warn(getCount(arr));  // {apple: 2, orange: 3, pear: 1, bananas: 2}
```

- Object.seal()
防止对象纂改：不能添加也不能删除属性

```
let obj = {
  name: 'Ersul'
}
Object.seal(obj); // 防止对象纂改：不能删除也不能添加属性
delete obj.name;
console.warn(obj.name); // Ertsul
obj.age = 22;
console.warn(obj.age); // undefined
```

- Object.freeze()
冻结对象，不能修改对象任何属性的值。如若

```
let obj = {
  name: 'Ersul'
}

Object.freeze(obj); // 冻结对象，不能修改属性的值
obj.name = 'Zero';
console.warn(obj.name); // Ertsul
```

- 数组字符串的相互转化

```
// 数组 --> 字符串 : join()
let arr3 = [1, 2, 3, 4, 5];
console.log(arr3.join('-'));
// 字符串 --> 数组 : split()
let str = 'apple,pear,bananas';
console.log(str.split(','));
```