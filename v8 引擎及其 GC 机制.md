

浏览器内核：又称渲染引擎，主要负责 html、css 的解析，页面布局，渲染和复合层合成。如：webkit。

javascript 引擎：主要负责 javascript 的解析。如：v8 引擎。

<img src="https://segmentfault.com/img/bV5zow" alt="内核引擎对应图 - 来源网络" style="zoom: 67%;" />

### v8 存储大小

- 64位系统：1.4G
- 32位系统：0.7G

由于存储空间有限，如果不进行 GC 的话，会造成内存空间不够用的情况。

### v8 内存结构

<img src="https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/8/16ee12280b78399d~tplv-t2oaga2asx-watermark.awebp" alt="v8 内存结构 - 网络图片" style="zoom:67%;" />

- 新生代区：大多数对象一开始都分配在这里，区域较小，垃圾回收频繁。
  - from-space：分配内存。
  - to-space：GC 回收时保留存活对象。
- 老生代区：新生代中的对象存活一段时间后会被转移到老生代区，区域较大，垃圾回收不太频繁。
  - 数据区：保存原始数据对象。
  - 指针区：保存指向其他对象的地址的指针。
- 大对象区：存放体积超越其他区域大小的对象，每个对象都会有自己的内存，垃圾回收不会移动大对象区。
- 代码区：代码对象，会被分配在这里，唯一拥有执行权限的内存区域。
- Map 区：存放 Cell 和 Map，每个区域都是存放相同大小的元素，结构简单。

### 回收算法

#### 新生代

主要使用 Scavenge 算法：

- 对象初始化时，在 from-space 中为新对象分配内存。
- GC 回收，扫描 from-space，判断对象的是否为存活对象（从根节点遍历所有对象节点）。
  - 存活对象：复制到 to-space。
  - 非存活对象：标记为非存活对象。
- 清除非存活对象。
- from-space 和 to-space 互换。

当新生代区中的对象经历过一次 Scavenge 算法（判断条件，对象内存地址是否变更）和新生代 to-space 内存占比大于 25%，该对象会从新生代区迁移到老生代区，该过程也叫对象晋升。

#### 老生代

老生代中保存着大量存活的对象，GC 回收也不频繁，如果老生代也采用 Scavenge 算法，会造成一半的内存浪费。

- 引用计数：判断该对象是否又其他引用指向它，如果没有，则视为垃圾并回收。缺点：对象之间可能存在循环引用的问题，造成内存泄漏。

```js
function fn() {
  const a = {}
  const b = {}
  // a 对象和 b 对象循环引用
  a.x = b
  b.xx = a
}
```

- 标记清除（Mark-Sweep）：分为标记和清除两个阶段。

  - 标记：GC 会构建一个根列表，从根节点出发，遍历所有可被访问到的节点，节点可被访问则标记为活动的，不可访问则标记为非活动的，视为垃圾。
  - 清除：清除非活动节点。

  缺点：由于被清理对象的内存地址可能是不连续的，经过标记清除之后，内存可能存在不连续的状态，即：内存碎片。

- 标记整理（Mark-Compact）：分为标记、整理和清除三个节点。

  - 标记：GC 会构建一个根列表，从根节点出发，遍历所有可被访问到的节点，节点可被访问则标记为活动的，不可访问则标记为非活动的，视为垃圾。
  - 整理：将活的对象移动到内存的另一端。
  - 清除：将原来一端的对象全部回收。

### 全停顿（stop-the-word）

不管是新生代还是老生代，执行 GC 都需要停止主线程 js 的执行，这就是全停顿。

解决全停顿方案有：

- 增量标记：将 GC 分段执行。每次执行 GC 返回主线程就做好标记，下一次继续 GC 从标记点继续执行。
- 并行回收：GC 逻辑在主线程和工作线程上。
- 并发回收：GC 逻辑在工作线程上。

---

参考链接：

- https://juejin.cn/post/6844904016325902344