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
  - *scrollTop* 和 *scrollLeft*：元素滚动部分相对于可是部分的顶部和左边的距离。
  - *clientX* 和 *clientY*：相对于浏览器的 *X* 轴和 *Y* 轴的距离。
  - *pageX* 和 *pageY*： 相对于 *document* 的 *X* 轴和 *Y* 轴的距离。
  - *clientWidth* 和 *clientHeight*：元素的宽高，包括：*padding*，不包括 *margin* 和 *border*。
  - *window.innerWidth* 和 *window.innerHeight*：视窗的宽高。
  - *element.getBoundingClientRect()*：用于获取元素的宽高和偏移量。

  