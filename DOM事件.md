## 事件
事件就是用户或者浏览器进行的特定行为，比如：鼠标点击事件。

## 事件名称

比如：click就是一个事件名。

## 事件流的执行过程

事件流：指的是 *DOM* 事件流（多个事件按照一定顺序执行），DOM(文档对象模型)结构是一个树型结构，当一个HTML元素产生一个事件时，该事件会在元素节点与根结点之间的路径传播，路径所经过的结点都会收到该事件，这个传播过程可称为DOM事件流。
事件流的执行过程：从window开始，最后回到window。
事件流被分为三个阶段：事件捕获（1-5），事件触发（5-6），事件冒泡（6-10）。

来自网上的图片：

![事件流.png](http://upload-images.jianshu.io/upload_images/659084-eb2b449c3348c5d6?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### [](#事件冒泡 "事件冒泡")事件冒泡

来自百度的解释：

> 当事件发生后，这个事件就要开始传播(从里到外或者从外向里)。为什么要传播呢？因为事件源本身（可能）并没有处理事件的能力，即处理事件的函数（方法）并未绑定在该事件源上。

就比如：我们用button产生一个click事件，但是由于button按钮本身可能不能处理这个事件，就要开始传播出去这个事件。
实现事件冒泡的条件是：元素之间有嵌套，每个元素都有相同的事件，这样当最里层的事件触发时，这个事件就会一级一级向上传播，也就是”冒泡“。

举个例子：在一个div里面嵌套另一个div，两个div都绑定了相同的事件监听函数click。这样，当里层点击事件触发时，外层的点击事件也会被触发。

````
<div id="outer">
 <div id="inner">
 </div>
</div>

<script>
 window.onload = function () {
   let inner = document.getElementById('inner')
   let outer = document.getElementById('outer')
   inner.addEventListener('click', function (e) {
     console.log('inner');
   })
   outer.addEventListener('click', function () { 
     console.log('outer');
   })
 }
</script>
````

结果为：
![事件冒泡.gif](http://upload-images.jianshu.io/upload_images/659084-be2355e585a26bc5?imageMogr2/auto-orient/strip)

##### 阻止事件冒泡

*   一般浏览器：event.stopPropagation()
*   IE浏览器：event.cancelBubble = true

##### 阻止默认事件

*   一般浏览器：event.preventDefault()
*   IE浏览器：event.returnValue = false

## addEventListener的第三个参数

addEventListener方法用于将事件和函数进行绑定，函数为：addEventListener(event, fn, useCapture)，这些参数分别是：事件名，事件处理函数，第三个参数 **useCapture** 是一个布尔值，用来设置该事件的执行是在哪个阶段发生的（默认是在冒泡阶段）。

*   true：表示该事件是在“事件捕获阶段”触发的。（由外向内）
*   false：表示该事件是在“事件冒泡阶段”触发的。（由内向外）

举个例子：在一个div里面嵌套另一个div，两个div都绑定了相同的事件监听函数click。

将第三个参数设置为true，事件在 **捕获阶段** 执行：

````
<div id="outer">
 <div id="inner">
 </div>
</div>

<script>
 window.onload = function () {
 let inner = document.getElementById('inner')
 let outer = document.getElementById('outer')
 inner.addEventListener('click', function (e) {
 console.log('inner');
 }, true)
 outer.addEventListener('click', function () { 
 console.log('outer');
 }, true)
 }
</script>
````

由于是 **捕获阶段** 执行，即：由外向内执行，所以结果为： outer inner。

结果为：
![image.png](http://upload-images.jianshu.io/upload_images/659084-82bb228984257ae1?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

将第三个参数设置为false，事件在 **冒泡阶段** 执行，输出结果为： inner outer。
![image.png](http://upload-images.jianshu.io/upload_images/659084-1d1d0c25024cc18d?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 事件处理函数/事件监听函数

事件触发后的处理函数。如：btn.onclick = fn，fn就是事件处理函数/事件监听函数。

#### HTML事件处理程序

例如：

````
<button onclick = "fn()">click</button>
````

这种方式的缺点是：结构和行为耦合在一起。

#### DOM0级事件处理程序

形式为：ele.on + ‘事件名称’ = 事件处理函数
例如： ele.onclick = function() {…}，this指向当前元素。

#### DOM2级事件处理程序

这种方式主要是通过这两个方法：addEventListener()，removeEventListener()
这种方式的优点就是：可以同一个事件处理程序绑定多个事件处理函数。

## 事件委托

事件委托要处理的问题是：解决事件处理程序过多的问题。简单来说，就是将很多个事件处理程序委托给一个事件处理程序，通过这个事件处理程序来管理这么多个事件处理程序。
好处：减少对DOM的访问，提高性能；可以对后来添加进来的元素也能绑定委托事件。

例如：在一个 ul 标签中有很多个 li 标签，我们的需求而是点击每一个 li 标签显示每一个 li 标签的内容。
如果没有使用事件委托，我们需要给每一个 li 标签都添加事件监听函数，很明显，这样的效率并不高，而且对整个网页的性能也会影响；如果我们使用事件委托，将这些 li 的事件委托给 ul 标签，这样就可以只用一个事件监听函数实现。

代码如下：

````
<ul id="list"> 
 <li>111111</li>
 <li>222222</li>
 <li>333333</li>
 <li>444444</li>
</ul>

<script>
 window.onload = function () { 
 // 传统的方法
 // let lis = document.getElementsByTagName('li')
 // console.log(lis);
 // for(let i = 0; i < lis.length; i++){
 //     lis[i].addEventListener('click', function () { 
 //         console.log(this.innerHTML);
 //     })
 // }
 // 使用事件委托
 let list = document.getElementById('list')
 list.addEventListener('click', function (e) { 
   let target = e.target
   console.log(target.innerHTML);
 }, false)
 }
</script>
````

事件委托一般使用的是 **事件冒泡阶段** 进行（虽然事件捕获阶段也可以），因为事件冒泡的事件流模型被所有的主流浏览器兼容，事件冒泡的兼容性更好。