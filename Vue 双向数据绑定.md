## Vue 数据劫持

### 原理

 *Vue* 的双向数据绑定是使用**发布订阅者模式**、**数据劫持**和`Object.defineProperty()`（ Vue 3.0  改用 Proxy）实现。核心原理如下图所示：

![image.png](https://upload-images.jianshu.io/upload_images/659084-587c891bf6c3f81e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 解析到页面中数据绑定，通过触发 *getter* 新增一个订阅者/观察者 *Watcher* 到订阅者列表 *subs*。
- 数据改动，通过触发 *setter* 通知所有的订阅者。 

### 实现

#### 数据劫持

数据劫持是使用 `Object.defineProperty()`进行实现的。通过遍历数据对象 *data* 的所有属性，设置每个属性的 *getter* 和 *setter* 。

- 添加 *observe* 函数。作用是通过该函数获取当前对象的所有属性，并调用 *defineReative* 函数设置 *getter* 和 *setter*。

```javascript
function observe(obj) {
  // 过滤不是对象的值
  if(!obj || Object.prototype.toString.call(obj) !== '[object Object]') {
    return;   
  }
  // 遍历对象的属性
  Object.keys(obj).forEach(key => {
    defineReative(obj, key, obj[key]); //  调用 defineReative 函数设置 getter 和 setter
  })
}
```

- *defineReative* 函数。设置  *getter* 和 *setter*。

```javascript
function defineReative(obj, key, val) {
  observe(val); // 递归
  let dp = new Dep(); // 实例化一个依赖收集对象，用于添加订阅者到订阅者列表和通知订阅者更新
  Object.defineProperty(obj, key, {
    configurable: true,
    enumerable: true,
    get: function() {
      if (Dep.target) { // 添加订阅者时候会将全局 Dep.target 指向 Watcher 自己，会触发 getter
        dp.addSub(Dep.target);
      }
      return val;
    },
    set: function(newVal) {
      val = newVal;
      dp.notify(); // 通知所有的订阅者
    }
  })
}
```

#### 依赖收集

- 添加 *Dep* 类。用于添加订阅者和通知所有订阅者更新。

```javascript
class Dep {
  constructor() {
    this.subs = [];
  }
  addSub(sub) {
    this.subs.push(sub);
  }
  notify() {
    this.subs.forEach(sub => {
      sub.update(); // sub 指向 Watcher 实例
    })
  }
}
```

- 设置全局 `Dep.target = null`。用于判断是否有订阅者 *Watcher* 触发 *getter* ，有的话添加到订阅者列表。

#### 订阅者

- 添加 *Watcher* 类。用于添加订阅者实例，通过改变全局 *Dep.target* ，触发 *getter* 并添加到订阅者列表。  

```javascript
class Watcher{
  constructor(obj, key, cb) {
    Dep.target = this; // 改变全局 Dep.target，指向自身
    this.obj = obj;
    this.key = key;
    this.value = obj[key]; // 触发 getter 操作，并添加到订阅者列表
    Dep.target = null; // 重新将 Dep.target 置为 null
  }
  update() {
    const value = this.obj[this.key]; // 获取最新值
    this.cb(value); // 通过回调更新 DOM 节点内容
  }
}
```

#### 模拟操作

- *html* 页面添加标签并进行数据绑定。

```html
<div class="container">{{name}}</div>
```

- 添加更新 *DOM* 节点内容回调函数。

```javascript
function update(value) {
  document.querySelector(".container").innerText = value;
}
```

- 初始化 *data* 对象。

```javascript
let data = {
	name: "zero"
}
```

- 触发数据劫持。

```javascript
observe(data);
```

- 添加订阅者。

```javascript
new Watcher(data, "name", update);
```

- 更改 *name*。

```javascript
data.name = 'Ertsul';
```

打开浏览器，会看到 *{{name}}* 已经被改成 *Ertsul*。

![GIF2.gif](https://upload-images.jianshu.io/upload_images/659084-1c971d01c5af3b86.gif?imageMogr2/auto-orient/strip)

































