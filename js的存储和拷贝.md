## 数据类型
#### 基本数据类型
- Number
- String
- Boolean
- Undefined
- Null
#### 引用类型
多个基本数据类型复合形成。
- Object

## 存储方式
#### 基本数据类型的存储方式
- 每当定义一个基本数据类型的变量，会在 **栈区** 开辟一个内存空间，用于存放该变量。栈区的特点是：静态分配，大小固定。
- 当一个变量通过直接复制的方式复制给另一个变量，系统会在 **栈区**  重新开辟一个内存空间；两个变量互不影响。

````
let num1 = 10;
let num2 = num1;
num2 = 20;
console.log(num1);     // 10 
````

#### 引用类型的存储方式
- 每当定义一个引用类型，如：对象，会在 **堆区** 开辟一个内存空间；然后如果创建一个该对象的实例，会在 **栈区** 开辟一个内存存放该实例，该实例实际上是一个指向 **堆内存** 对象的指针。堆区的特点是：动态分配，大小不固定。
- 当一个实例直接复制给另一个实例，系统会在 **栈区**  重新开辟一个内存空间，但是新实例同样也是一个指向  **堆内存** 对象的指针，所以，这样的操作，这样的修改都会对新旧实例产生影响。

````
let obj1 = {
    name: 'zero',
    age: 22,
    num: ['1', '2', '3']
}

//直接复制
let obj2 = obj1;

obj2.name = 'Ertsul';
obj2.num[0] = '一';
console.log(obj1, obj2);
// name:"Ertsul"
// age:22
// num:["一", "2", "3"]
````

## 浅拷贝和深拷贝
#### 区别
两者的区别主要在于 **复制层次** 的不同：
- 浅拷贝主要复制到对象属性这一层次，如果对象里面有子对象，则无法对子对象完成复制；之后对于子对象的修改 **会** 影响到原复制对象。
- 深拷贝则是浅拷贝的 **加强版** ，可以实现对于子对象的拷贝；之后对于子对象的修改 **不会** 影响到原复制对象。主要实现方法有：
  - 递归
  - JSON解析
#### 浅拷贝

````
// 浅拷贝
function shallowCopy(source) {
    let result = {};
    for(let key in source){
        result[key] = source[key]
    }
    return result;
}
let obj1 = {
    name: 'zero',
    age: 22,
    num: ['1', '2', '3']
}

let obj2 = {};
obj2 = shallowCopy(obj1);

obj2.name = 'Ertsul';    // 不会产生影响
obj2.num[0] = '一';      // 产生影响
console.log("obj1", obj1);
console.log("obj2", obj2);
````

结果如图：

![image.png](http://upload-images.jianshu.io/upload_images/659084-2238425cd781911c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 深拷贝

##### 递归

````
// 递归实现深拷贝
function deepCopy(source, res){
    var res = res || {};
    for(let i in source){
        if(typeof source[i] === 'object'){
            if(source[i].constructor === Array){
                res[i] = []
            }else {
                res[i] = {}
            }
            deepCopy(source[i], res[i]);   // 递归子对象属性
        }else{
            res[i] = source[i]
        }
    }
    return res;
}

let obj1 = {
    name: 'zero',
    age: 22,
    num: ['1', '2', '3']
}

let result = {}
result = deepCopy(obj1, result);
result.name = 'Ertsul';
result.num[1] = '二';
console.log("result", result);
console.log("obj1", obj1);
````

结果如图：

![image.png](http://upload-images.jianshu.io/upload_images/659084-522f00e896a05506.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### JSON解析

````
// JSON实现深拷贝
let obj1 = {
    name: 'zero',
    age: 22,
    num: ['1', '2', '3']
}
let result1 = JSON.parse(JSON.stringify(obj1));
result1.num[2] = '三';
console.log('result1', result1);
console.log('obj1', obj1);
````

结果如图：
![image.png](http://upload-images.jianshu.io/upload_images/659084-b747df9bb831bf93.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
