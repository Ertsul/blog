## 原型模式

原型模式在《JavaScript高级程序设计》中的定义：

> 我们创建的每个函数都有一个prototype（原型）属性，这个属性是一个指针，指向一个对象，而这个对象的用途是包含可以由特定类型的所有实例共享的属性和方法。如果按照字面意思来理解，那么prototype就是通过调用构造函数而创建的那个对象实例的原型对象。使用这个对象的好处就是可以让所有对象实例共享它包含的属性和方法。

这里还有一个\_\_proto\_\_属性，每一个对象都拥有这个属性，这个属性指向对应的构造函数的prototype属性。

通过原型对象，我们就不用将信息添加到构造函数中了。

#### 实例对象共享原型对象的属性

````
function Person(){}
Person.prototype.name = 'A'
Person.prototype.age = 1
// 实例对象共享原型对象的属性
let p1 = new Person()
console.log(p1.name, p1.age);   // A 1
let p2 = new Person()
console.log(p2.name, p2.age);   // A 1
````

#### constructor（构造函数）属性

每个原型对象在默认情况下都会自动获取一个constructor（构造函数）属性，这个属性包含一个指针，指向这个原型所在的函数。在上面的例子中，创建Person函数后，它的prototype指向它的原型对象，这个原型对象就会获取获取constructor属性，指向Person函数。

#### 判断实例对象与原型对象之间的关系

*   Obj.prototype.isPrototypeOf(obj)

````
console.log(Person.prototype.isPrototypeOf(p1)) // true
 ````

*   Object.getPrototypeOf(obj)：返回对象实例的原型[es5]

````
console.log(Object.getPrototypeOf(p1) == Person.prototype);   // true
console.log(Object.getPrototypeOf(p1).name);    // A
````

*   obj.hasOwnPrototype(attr)：检测一个属性是否存在于实例中。只有当实例对象重写原型对象的属性后（非原型属性）才会返回true。

````
console.log(p1.hasOwnProperty('name')); // false
p1.name = 'Ertsul'
console.log(p1.hasOwnProperty('name')); //true
````

*   in操作符：通过in操作符和hasOwnProperty()可以判断某属性存在于实例还是原型对象中【in判断是否存在属性，hasOwnProperty()判断存在于哪里。】可以单独使用和for-in循环使用：无论属性属于实例还是原型中，都会返回true。(‘name’ in person1)

*   获取所有实例的属性名字，无论是否可以枚举。

````
console.log(Object.getOwnPropertyNames(p1)); // ["name"]
````

*   instanceof

````
console.log(p1 instanceof Person);    // true
````

#### 更简单的语法

````
function Person() {  }
Person.prototype = {
 name: '001'
}
let p1 = new Person()
console.log(p1.name);
````

这里的例外是：constructor属性不再指向Person；本质上相当于完全重写了Person对象了。注：重写原型对象切断了现有原型与任何之前已经存在的对象实例之间的联系，它们引用的仍然是最初的原型。[详见《JavaScript高级程序设计》P156-P157，原型的动态性]

####搜索机制

*   解释器会现在目标对象中搜索目标属性，如果当前对象没有目标属性，解释器继续在目标对象的原型对象中搜索。
*   通过delete只能删除对象的属性，而不能删除原型对象中的属性。

#### 组合模式（默认类型）：构造函数模式 + 原型模式

构造函数模式用于定义实例属性，而原型模式用于定义方法和共享的属性。

## 继承

#### 原型链

````
// 父亲
function Father() { 
 this.fatherProperty = true
}
Father.prototype.getFatherValue = function () { 
 return this.fatherProperty
}
// 儿子
function Son() { 
 this.sonProperty = false
}
// 继承父亲：通过实例化父亲，将实例化对象作为儿子的原型，这样儿子就继承了父亲的属性
Son.prototype = new Father()
Son.prototype.getSonValue = function () { 
 return this.sonProperty
}

let instance = new Son()
console.log(instance.fatherProperty);   // true
````

上面例子中，通过实例化父亲，将实例化对象作为儿子的原型，这样儿子就继承了父亲的属性。（重写儿子的原型）。这里，儿子原型对象的[[prototype]]（注：不是prototype，[[prototype]]存在于原型中）指向的是父亲的原型对象，而父亲的原型对象的[[prototype]]指向的是Object Prototype。（由于所有函数的默认原型都是Object的实例，所以默认原型都会包含一个内部指针，指向Object.prototype）

#### 借用构造函数

*   主要是为了解决因为引用类型值的原型属性会被所有实例共享的问题。
*   **借用构造函数/伪造对象/经典继承**：在子类型构造函数的内部调用超类型构造函数。

````
// 父亲
    function Father() { 
     this.info = ['Zero']
    }
    // 儿子
    function Son() { 
     // 继承父亲：在子类型构造函数的内部调用超类型构造函数
     // 《JavaScript高级程序设计》：函数只不过是在特定环境中执行代码的对象，因此通过使用 apply()和 call() 可以在新创建的对象上执行构造函数。 
     Father.call(this)        // 借用超类的构造函数
    }

    let f = new Father()
    f.info.push('100')
    console.log(f.info);    // ["Zero", "100"]

    let s = new Son()
    console.log(s.info);    // ["Zero"]
````

如果在继承的时候要传递参数，则在后面添加参数即可。如：Father.call(this， param1, param2)

#### \_\_proto\_\_

通过**\_\_proto\_\_**也可以实现子类型对超类型的继承。

````
const obj = {
 num : 123 
}
const obj1 = {
 __proto__ : obj
}
console.log(obj1.num);  // 123
````

#### Object.create(source)

````
const obj = {
 num : 123 
}
const obj2 = Object.create(obj)
console.log(obj2.num);  // 123
````

#### Object.assign(Object.create(obj), {…})

````
const obj = {
 num : 123 
}
const obj3 = Object.assign(
 Object.create(obj),
 {
 num2 : 234
 }
)
console.log(obj3.num, obj3.num2);   // 123 234
````

## 补充
#### 继承机制如下图：

![image.png](http://upload-images.jianshu.io/upload_images/659084-862bb97f2e2fe227.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 测试代码如下：

````
// 父类
function SuperType(){
}
// 子类
function SubType(){

}
// 子类继承父类
SubType.prototype = new SuperType();
// 实例化父类和子类
let superIns = new SuperType();
let subIns = new SubType();

// 子类
console.log(SubType.prototype);     // SuperType {}
console.log(subIns.__proto__);      // SuperType {}

// 父类
console.log(SuperType.prototype);       // {constructor: ƒ} 
console.log(superIns.__proto__);        // {constructor: ƒ}

console.log(SuperType.prototype.__proto__);     
// {constructor: ƒ, __defineGetter__: ƒ, __defineSetter__: ƒ, hasOwnProperty: ƒ, __lookupGetter__: ƒ, …}
// 即： Object prototype

console.log(SuperType.prototype.__proto__.constructor);
// ƒ Object() { [native code] } 即：Object
````