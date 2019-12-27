### BFC 

独立的渲染区域，规定了区域内部的布局情况，与区域外部的元素毫无关系。

#### 生成

- 根元素
- 浮动元素
- 绝对定位
- 弹性布局
- *overflow* 不为 *visible* 
- *display: table-cell;*

#### 作用

- 垂直方向上依次排列
- 同一 *BFC* 区域会有外边距重叠情况
- 计算高度，浮动元素也计算进去
- 不会被别的浮动元素遮挡
- 内部子元素的左边会与盒子的边框重叠

### 单行溢出省略

```css
{
	overflow: hidden;
	text-overflow: ellipsis;
	white-space: nowrap;
}
```

### 多行溢出省略

```css
{
	overflow: hidden;
    display: -webkit-box;
    -webkit-line-clamp: 3;
    -webkit-box-orient: vertical;
    text-overflow: ellipsis;
}
```

### 垂直水平居中

```css
.box {
    display: flex;
    align-items: center;
    justify-conent: center;
}
```

```css
.parent {
    position: relative;
}
.child {
    position: absolute;
    left: 50%;
    top: 50%;
    transform: translate3d(-50%, -50%, 0);
}
```

### 清除浮动

原因：父元素高度坍塌

- 增加空的子元素，添加样式`clear:both;`
- 父元素形成 *BFC*
- 父元素伪类

```css
.clearfix::after {
    content: " ";
    display: block;
    clear: both;
}
/* 兼容 IE6，IE7 */
.clearfix {zoom:1;}
```

### 盒子模型

盒子模型包括：外边距 *margin*、边框 *border*、内边距 *padding* 、内容 *content*。

- 标准盒子模型：`width = content-width`
- *IE* 盒子模型：`width = border-width + padding-width + content-width`

*box-sizing* ：设置盒子是*标准模型*还是 *IE 模型*。属性有：

- border-box：*IE* 模型
- padding-box
- content-box：标准模型

### 各种宽高

- *offsetWidth* 和 *offsetHeight*：元素的宽高，包括：*border + padding + content*。
- *offsetTop* 和 *offsetLeft*：元素相对于父元素的顶部和左边的距离。
- *scrollWidth* 和 *scrollHeight*：元素的宽高，包括滚动看不见的部分，包括：*padding*，不包括 *margin* 和 *border*。
- *scrollTop* 和 *scrollLeft*：元素滚动部分相对于显示区域的顶部和左边的距离。
- *clientX* 和 *clientY*：相对于浏览器的 *X* 轴和 *Y* 轴的距离。
- *pageX* 和 *pageY*： 相对于 *document* 的 *X* 轴和 *Y* 轴的距离。
- *clientWidth* 和 *clientHeight*：元素的宽高，包括：*padding*，不包括 *margin* 和 *border*。
- *window.innerWidth* 和 *window.innerHeight*：视窗的宽高。
- *element.getBoundingClientRect()*：用于获取元素的宽高和偏移量。

### new 操作符

- 创建一个空对象
- 空对象链接到构造函数的原型
- 执行构造函数，改变 *this* 指向
- 返回值如果是对象，则返回该对象；否则返回新对象。

```javascript
function Create(Con, ...arg) {
    let obj = Object.create(Con.prototype); // 创建空对象；指向构造函数的原型
    // let obj = {};
    // obj.__proto__ = Con.prototype;
    let result = Con.apply(obj, arg); // 改变 this 指向
    return Object.prototype.toString.call(result) === '[object Object]' ? restul : obj; // 返回值，如果是对象，就返回构造函数，否则返回 obj
}
```

```javascript
function Create() {
    let obj = new Object(); // 新建对象
    let Con = [].shift.call(arguments); // 获取第一个实参，构造函数
    obj.__proto__ = Con.prototype; // 更改原型链
    let result = Con.apply(obj, arg); // 修改 this 指向
    return Object.prototype.toString.call(result) === '[object Object]' ? restul : obj;
}
```

### 深复制

```javascript
function deepCopy(target) {
	let result = Object.prototype.toString.call(target) === '[object Object]' ? {} : [];
    for (let key in target) {
        if (target.hasOwnProperty(key)) {
            const value = target[key];
            if (Object.prototype.toString.call(value) === '[object Object]') {
                result[key] = deepCopy(value);
            } else {
                result[key] = value;
            }
        }
    }
    return result;
}
```

### 防抖与节流

#### 防抖 debounce

将高频操作转化为低频操作。

注意：`clearTimeout(timeout)` 并不会使 *timeout* 为 *null*，只会清空定时器。

```javascript
function debounce(fn, wait, immediate = false) {
  var timeout, result;

  timeout && clearTimeout(timeout);
  var debounced = function () {
    var context = this;
    var args = arguments;

    if (immediate) { // 事件停止触发后 n 秒，触发事件立刻执行回调
      var callNow = !timeout;
      timeout = setTimeout(function () {
        timeout = null;
      }, wait)
      if (callNow) {
        result = fn.apply(context, args);
      }
    } else { // 事件停止触发后，执行回调函数
      timeout = setTimeout(function () {
        fn.apply(context, args);
      }, wait);
    }
  }

  debounced.cancel = function () {
    clearTimeout(timeout);
    timeout = null;
  }

  return debounced;
}
```

#### 节流 throttle

 持续触发事件，每隔一段时间，只执行一次事件 。

- 时间戳

```javascript
function throttle(fn, wait) {
    var previous = 0;
    
    return function () {
        var args = arguments;
        var context = this;
        var now = +new Date();
        if (now - previous > wait) {
            fn.apply(context, args);
            previous = now;
        }
    }
}
```

- 定时器

```javascript
function throttle(fn, wait) {
    var timeout;

    return function() {
        var args = arguments;
        var context = this;
        if (!timeout) {
            timeout = setTimeout(function() {
                timeout = null;
                fn.apply(context, args);
            }, wait)
        }
    }
}
```

### 数组扁平化

- 递归

```javascript
function flattern(target) {
	let result = [];
    for (let idx in target) {
        const value = target[idx];
        if(Object.prototype.toString.call(value) === '[object Array]') {
           result = [...result, ...flattern(value)];
        } else {
           result = [...result, value]; 
        }
    }
    return result;
}
```

- reduce

```javascript
function flattern(target) {
    return target.reduce((prev, next) => prev.concat(Array.isArray(next) ? flattern(next) : next), []);
}
```

### 模块

#### CommonJs

- 同步加载：*CommonJs* 主要用于服务器，比如：*node.js*，文件模块的读取主要在硬件磁盘；而浏览器更多是基于网络请求。
- 值的拷贝：模块内部与导出的值不相影响。
- 运行时加载：*CommonJs* 会先将模块整个引进，然后生成一个对象，从这个对象中取值。
- 支持动态导入，即：`require(${path}/xx.js)`。

#### ES6

- 值的引用：*JS* 引擎对脚本进行静态解析的时候，遇到 *import*，会先生成一个只读引用；在脚本真正执行的时候，再根据这个只读引用，到被加载的模块中去取值。
- 编译时加载：在 *import* 的时候可以指定输出值，而不是加载整个模块。
- 不支持动态导入。
- 会编译成 `CommanJs`执行。

#### AMD/CMD

- 异步加载

### 数字添加千位符

- `num.toLocaleString()`

- 正则：`num1 = num.toString().replace(/(\d)(?=(\d{3})+$)/g, $1 => $1 + ',')`

  如：*1234567890*， 匹配的是 *7 890*、*4 567890*，类推。 

### 大小驼峰、中划线转化

#### 中划线转大驼峰

```javascript
function toBigHump(str) {
  return str.replace(/\-([a-z])/g, ($1, $2) => $2.toUpperCase());
}
```

#### 中划线转小驼峰

```javascript
function toSmallHump(str) {
  return str.replace(/[^]\-([a-z])/g, ($1, $2) =>  $2.toUpperCase()).split('-')[1];
}
```

#### 驼峰转中划线

```javascript
function toLine(str) {
  let res = str.replace(/([A-Z])(?=.*)/g, $1 => "-" + $1.toLowerCase());
  return res[0] == "-" ? res.slice(1) : res;
}
```

### 获取时间戳

- `Date.parse(new Date())`
- `+new Date()`

### 模拟 call 、apply 和 bind

#### call

`callFn.myCall(callObj, 'iuiu');`

- 获取被绑定的对象 *callObj*。
- 获取执行函数 *callFn*。
- 获取函数参数。
- 执行函数并返回结果。

```javascript
Function.prototype.myCall = function(context) {
	// callFn.myCall(callObj, 'iuiu');
    context = context || window; // 被绑定的对象 => callObj
    context.fn = this; // this 指向（待执行的函数） => callFn
    const args = [].slice.call(arguments, 1); // 获取函数参数
    const result = context.fn(...args); // 函数返回值，此时 fn/getName 的 this 已经指向 callObj
    delete context.fn;
    return result;
}
```

#### apply

`applyFn.myApply(applyObj, ['iuiu']);`

- 获取被绑定的对象 *applyObj*。
- 获取执行函数 *applyFn*。
- 判断是否有函数参数数组，有则展开。
- 执行函数并返回结果。

```javascript
Function.prototype.myApply = function(context) {
    // applyFn.myApply(applyObj, ['iuiu']);
    context = context || window; // 被绑定的对象 => applyFn
    context.fn = this; // this 指向（待执行的函数） => callFn
    const result = arguments[1] ? context.fn(...arguments[1]) : context.fn(); // 判断是否有函数参数数组
    delete context.fn;
    return result;
}
```

#### bind

`bindFn.myBind(bindObj, 'iuiu')();`

- 获取执行函数 *bindFn*。

- 获取 *bindFn* 的参数。

- 返回函数 *F*。

- 判断函数 *F* 的执行方式：

  - *new* 方式：直接 *new F()* 函数。
  - 直接执行函数。

- 返回函数执行结果。

  

```javascript
Function.prototype.myBind = function(context) {
	// bindFn.myBind(bindObj, 'iuiu')();
    if(typeof this !== 'function') {
        throw new TypeError('TypeError');
    }
    var _this = this;
    var args = [].slice.call(arguments, 1); // 获取 bindFn 函数参数
    return function F() { // 返回函数
        if(this instanceof F) { // new F() 情况
           return new _this(...args, ...arguments); // args 为外层参数，argumente 为内层函数 F 的参数
        }
        return _this.apply(context, [...args, ...arguments]); // 执行函数
    }
}
```

###  性能优化

#### html

- 公共代码抽离成组件
- 减少 *reflow*

#### css

- 嵌套不要超过 *3* 层（构建 *CSSOM* 树的过程会阻塞，耗性能）
- *css* 文件放头部 

##### js

- 放 *body* 底部（*js* 文件执行会阻塞渲染）
- *defer / async*
- 复杂计算使用 *webworker*

#### 图片

- 压缩图片
- 小图采用 *base64*
- 雪碧图
- 图片懒加载
- 采用 *WebP* 图片格式
- 对于移动端，可以先计算设备的宽高，然后再去请求对应的图片，没必要请求原图

#### 网络

- 缓存
  - 强缓存：*Expires / Cache-control*
  - 协商缓存：*Last-Modified / If-Modified-Since*
- *DNS* 预解析： `<link rel="dns-prefetch" href="//xxx.cn" />`
- 预加载：`<link rel="preload" href="http://xxx.com" />`

#### webpack

- *tree-shaking*
- 生产模式压缩代码

### 常用状态码

- 2xx：消息
  - 200：请求成功
- 3xx：重定向
  - 301：永久重定向
  - 302：临时重定向
  - 304：自上次请求，未修改的文件
- 4xx：客户端
  - 400：错误请求
  - 401：用户身份授权
  - 403：拒绝被拒绝
- 5xx：服务端
  - 500：服务端的未知错误
  - 502：网关错误
  - 503：服务暂时无法使用



#### 

