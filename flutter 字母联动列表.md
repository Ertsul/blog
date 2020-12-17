# flutter 字母联动列表

网上的第三方包，用的最多就是：[azListView](https://pub.dev/packages/azlistview)。

但是它有 bug：当列表到达页面底部的时候，滑动最后几个字母，还会触发回弹到屏幕顶部。如下：

![azlisview_list.gif](https://i.loli.net/2020/12/07/Kkad3H7t82GXyLq.gif)

经过分析，一开始以为是 ListView 的 physics 属性影响，[physics ](https://book.flutterchina.club/chapter6/intro.html)介绍：

- ClampingScrollPhysics：Android下微光效果。
- BouncingScrollPhysics：iOS下弹性效果。
- NeverScrollableScrollPhysics：禁止滚动。

但是设置该属性其实没有效果，还是会出现上面的问题。

后面多次测试 ListView，发现是 ListView 跳转事件 jumpTo 的影响，后面改用 animateTo，animateTo 的用法如下：

```
_scrollController.animateTo( offsetHeight, duration: Duration(milliseconds: 10), curve: Curves.linear, )
```

但是又有新的问题：

- 当连续快速滑动的时候，这个 duration 相当于多了个防抖的作用。
- 点击字母列表的时候，也会有问题。

效果如下：

![animato_list.gif](https://i.loli.net/2020/12/07/iQeIG5ay6TCEKHU.gif)

azListView 存在问题：

- azListView 包体较大；
- 存在的问题修改源码耗时较久。

----

自己实现，效果如下：

![alphabet_link_list.gif](https://i.loli.net/2020/12/07/Ts58gGam3C6FBI2.gif)

## 布局

整个页面用 Stack 布局，分为：

- 主列表（头部 + 城市列表）
- 主列表顶部字母提示
- 字母列表（字母列表 + 当前字母提示）

代码如下：

```dart
/// _buildBody
Widget _buildBody() {
  return Stack(
    children: [
      _buildMainListView(),
      _buildSusBarView(),
      Positioned(
        top: duSetHeight(120),
        right: duSetWidth(10),
        child: Row(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [_buildAlphabetListView()],
        ),
      ),
    ],
  );
}
```

## GestureDetector

[GestureDetector 文档链接](https://book.flutterchina.club/chapter8/gesture.html)

- onVerticalDragUpdate：滑动更新
- onVerticalDragEnd：滑动结束
- ...

## 主要思路

### 1.初始化的时候，先计算获取以下数值。

- 计算主列表每个字母分组需要跳转的 offsetTopList 和每个字母分组所占用的高度 groupHeight；

【计算：遍历字母列表，字母的 offsetTop = 之前的所有字母所有的 offsetTop 累加；当前字母的 groupHeight = 每个主列表项 itemHeight * 每个字母分组数量。】

```dart
/// 计算主列表每个字母需要滚动的距离和每个字母项所占用的高度
void _calcEachLetterOffsetTop() {
  List<String> _alphabetList = widget.alphabetList;

  /// offset top
  _alphabetListOffsetTopMap[_alphabetList[0]] = 0; // '#' header offsetTop
  _alphabetListOffsetTopMap[_alphabetList[1]] =
      widget.headerHeight; // 第一个字母，滚动距离为顶部 header 高度 offsettop
  /// height
  _alphabetListHeightMap[_alphabetList[0]] =
      widget.headerHeight; // '#' header height
  _alphabetListHeightMap[_alphabetList[1]] =
      widget.alphabetCountMap[_alphabetList[1]] * widget.mainListItemHeight +
          widget.guideBarHeight; // 第一个字母所占用的高度
  // 第二项开始循环
  for (int i = 2; i < _alphabetList.length; i++) {
    String _curLetter = _alphabetList[i]; // 当前字母

    // 保存当前字母项的高度
    _alphabetListHeightMap[_curLetter] =
        widget.alphabetCountMap[_curLetter] * widget.mainListItemHeight +
            widget.guideBarHeight;

    // 计算主列表每一个字母项需要滚动的 offsetTop（之前的所有字母所有的 offsetTop 累加）
    double _offsetTopSum = 0.0;

    for (int j = i - 1; j > 0; j--) {
      String _preLetter = _alphabetList[j]; // 之前的字母
      // 将之前所有字母的高度累加
      // 主列表每个字母单元高度 = 子项数量 * 子项高度 + 主列表导航条高度
      _offsetTopSum +=
          widget.alphabetCountMap[_preLetter] * widget.mainListItemHeight +
              widget.guideBarHeight;
    }
    _alphabetListOffsetTopMap[_curLetter] =
        _offsetTopSum + widget.headerHeight; // 设置当前字母的滚动 offsetTop
  }
}
```

- 获取主列表不需要滚动的字母列表。

【计算：从字母表后面遍历，将每个字母分组所占用的高度 groupHeight 进行累加，与屏幕的高度进行比较，如果累加结果小于屏幕高度，则对该字母进行标记。】

主要代码如下：

```dart
/// 获取主列表不需要滚动的字母列表
void _getNotScrollAlphebetList() {
  final _screenHeight =
      duSetHeight(MediaQuery.of(context).size.height); // 屏幕高度
  // final _screenHeight = duSetHeight(size.height);
  double _heightSum = 0.0;
  // 判断那些字母不需要触发列表滚动，从后面字母开始循环累加，判断是否小于屏幕高度
  for (int i = widget.alphabetList.length - 1; i >= 0; i--) {
    String _curLetter = widget.alphabetList[i];
    bool _ifLessThanScreenHeight =
        _heightSum + _alphabetListHeightMap[_curLetter] <= _screenHeight;
    if (_ifLessThanScreenHeight) {
      _heightSum += _alphabetListHeightMap[_curLetter]; // 累加
      _notScrollAlphabetLst.insert(0, _curLetter); // 添加到列表
    } else {
      break;
    }
  }
}
```

### 2. 滑动主列表，获取当前的字母，更新右侧字母表状态和更新顶部字母导航条。

- 获取主列表当前滑动位置的 screenOffsetTop；
- 遍历 offsetTopList，判断 screenOffsetTop 在哪两个字母的区间内，则可以定位到目标字母。

主要代码如下：

```dart
/// 更新字母表当前激活的字母
void _updateAlphebetListView() {
  // 主列表上锁中（滚动执行中），直接返回
  if (_mainListScrollLock == true) return;
  List<String> _alphabetList = widget.alphabetList;
  // 监听主列表滚动，判断主列表的 offsetTop 在哪两个字母的 offsetTop 区间内，将右侧字母导航列表激活为当前字母
  for (int i = 0; i < widget.alphabetList.length; i++) {
    // 不为最后一个字母 + 主列表 offsetTop 大于当前字母的 offsetTop + 主列表的 offsetTop 小于下一个字母的 offsetTop
    bool _ifInCurRange = i != widget.alphabetList.length - 1 &&
        _scrollController.offset <=
            _alphabetListOffsetTopMap[_alphabetList[i + 1]] &&
        _scrollController.offset >
            _alphabetListOffsetTopMap[_alphabetList[i]];
    if (_ifInCurRange) {
      _selectedAlphabetIndex = i; // 设置字母表选中字母索引
      _mainListScrollLock = false; // 解锁
      setState(() {});
      return;
    }
  }
}
```

### 3. 滑动字母表，获取当前的字母，主列表跳转到目标字母和更新顶部字母导航条。

- 获取字母表当前位置相对于字母表容器顶部的高度 paddingOffsetTop【获取字母表当前位置对于视窗的高度 - 字母表容器顶部的高度相对于视窗的高度】;
- 计算每个字母项占用字母表容器的高度 alphabetItemHeight 【(字母表容器的高度 - padding) / 字母数量】；
- 获取当前索引，paddingOffsetTop / alphabetItemHeight；
- 获取当前字母和主列表 offsetTopList 滚动高度，
- 更新视图。

主要代码如下：

```dart
/// 滑动字母表：滚动主列表 + 更新字母列表
void _onVerticalDragUpdateHandle(DragUpdateDetails event, int index) {
  /// 更新字母表
  if (_alphabetListOffset == null) {
    // 获取字母表的 margin top
    RenderBox _box = _alphabetListKey?.currentContext?.findRenderObject();
    _alphabetListOffset = _box.localToGlobal(Offset.zero);
  }
  // 字母表的padding
  double _alphabetPadding = widget.alphabetListItemHeight;
  // 计算当前字母项索引：(当前活动的 dy - 字母表的 marginTop - 字母表的 padding) / 每个字母所占的高度
  int _curActiveLetterIndex =
      ((event.globalPosition.dy - _alphabetListOffset.dy - _alphabetPadding) /
              widget.alphabetListItemHeight)
          .round();
  if (_curActiveLetterIndex < 0) {
    // 滑出字母表区域上面，保持为 0
    _selectedAlphabetIndex = 0;
  } else if (_curActiveLetterIndex >= widget.alphabetList.length) {
    // 滑出字母表区域下面，保持为最后一个
    _selectedAlphabetIndex = widget.alphabetList.length - 1;
  } else {
    _selectedAlphabetIndex = _curActiveLetterIndex;
  }
  setState(() {});

  /// 滚动主列表
  String _curLetter = widget.alphabetList[_selectedAlphabetIndex]; // 获取当前字母项
  _scrollMainList(_curLetter);
}
```

### 4.当主列表已经滑动到底部，继续滑动字母表剩下的字母，判断当前字母时候标记为不需要滚动主列表，直接滑动列表底部，不滑动到字母分组位置。

主要代码如下：

```
/// 滚动主列表 void _scrollMainList(String letter) {  if (_notScrollAlphabetLst.contains(letter)) {    // 当前字母不需要滚动    _scrollController        .jumpTo(_scrollController.position.maxScrollExtent); // 滚动到列表底部    return;  }  _scrollController.jumpTo(    _alphabetListOffsetTopMap[letter],  ); }
```

### 5. 主列表滚动锁。

为防止出现上次滚动还没结束，下次滚动就开始执行的问题，给主列表滚动上锁。

```
/// 更新字母表当前激活的字母 void _updateAlphebetListView() {  // 主列表上锁中（滚动执行中），直接返回  if (_mainListScrollLock) return;  // ... }
```