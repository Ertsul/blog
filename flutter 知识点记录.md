## Flutter 记录

### 1. Button 限制宽高

使用 SizeBox 进行包裹。

```dart
Widget _arrowIcon = SizedBox(
  width: duSetWidth(18.0),
  child: FlatButton(
    padding: EdgeInsets.all(0),
    onPressed: () {
      print('_arrowIcon  tap');
    },
    child: Icon(
      IconFont.iconmore,
      color: AppColors.grayThirdColor,
      size: duSetFontSize(14),
    ),
  ),
);

```

### 2.获取目标 widget 相关信息

使用 GlobalKey()

```dart
// 定义变量
GlobalKey _Key = GlobalKey();
// 绑定 widget
Center(
    key: _Key,
    child: Text('content')
);
// 使用
RenderBox _box = _alphabetListKey?.currentContext?.findRenderObject();
_box.localToGlobal(Offset.zero); // offset
// ...
```

### 3.沉浸式状态栏

```dart
void _statusBar([String color]) {
    // 白色沉浸式状态栏颜色  白色文字
    SystemUiOverlayStyle light = SystemUiOverlayStyle(
      systemNavigationBarColor: Color(0xFF000000),
      systemNavigationBarDividerColor: null,

      /// 注意安卓要想实现沉浸式的状态栏 需要底部设置透明色
      statusBarColor: Colors.transparent,
      systemNavigationBarIconBrightness: Brightness.light,
      statusBarIconBrightness: Brightness.light,
      statusBarBrightness: Brightness.dark,
    );

    // 黑色沉浸式状态栏颜色 黑色文字
    SystemUiOverlayStyle dark = SystemUiOverlayStyle(
      systemNavigationBarColor: Color(0xFF000000),
      systemNavigationBarDividerColor: null,

      /// 注意安卓要想实现沉浸式的状态栏 需要底部设置透明色
      statusBarColor: Colors.transparent,
      systemNavigationBarIconBrightness: Brightness.light,
      statusBarIconBrightness: Brightness.dark,
      statusBarBrightness: Brightness.light,
    );
    "white" == color?.trim()
        ? SystemChrome.setSystemUIOverlayStyle(light)
        : SystemChrome.setSystemUIOverlayStyle(dark);
  }
```

### 4.获取顶部状态栏高度

- MediaQuery.of(context).padding.top
- MediaQueryData.fromWindow(window).padding.top

### 5.安全区域，比如：iphone 12等全面屏底部遮挡问题

- SafeArea

### 6.国际化初始化生命周期问题

问题：在 initState 直接调用 `DemoLocalizations.of(context).all`

```
原因：initState 生命周期的作用主要是将当前的 state 与上下文 buildContext 产生关联，此时的上下文还不能使用。
```

![image-20201209164937816.png](https://i.loli.net/2021/03/11/2Ox4blhzp3SjwaJ.png)

解决方案：

- initState 添加异步逻辑

```dart
Future.delayed(Duration.zero, () { // ... });
```

- 放 didChangeDependencies 生命周期

### 7.切换 tab，addListener 触发两次

点击切换tab的时候执行了一个动画效果，滑动切换的时候是没有的，在这个过程中触发了一次Listener，所以触发了两次addListener方法。

解决方案：在 addListener 中添加判断

- `if(_newsTabController.index == _newsTabController.animation.value) { //... }`
- `if(_newsTabController.indexIsChanging) { //... }`

### 8.stack 溢出部分点击失效问题

如下图：蓝色区域为父级 stack 容器，灰色容器（1+2）为 stack 子级样式，由于 2 溢出父级容器，出现问题：2 点击事件不生效。

![image-20201224173117517.png](https://i.loli.net/2021/03/11/cmoSrClHk9LYtTV.png)

解决办法：更改布局，父级容器包裹整个区域，不让其溢出。

### 9.vscdoe 多渠道 flavor 配置

根目录添加 .vscode/launch.json 配置文件，添加配置：

```dart
{
  "version": "xxx",
  "configurations": [
    {
      "name": "wapcar",
      "request": "launch",
      "type": "dart",
      "args": [
        "--flavor",
        "wapcar"
      ]
    },
    {
      "name": "autofun",
      "request": "launch",
      "type": "dart",
      "args": [
        "--flavor",
        "autofun"
      ]
    }
  ]
}
```

### 10.[json 序列化](https://flutter.cn/docs/development/data-and-backend/json#code-generation)

- 安装依赖

```yaml
dependencies:
	json_annotation: ^3.1.1
dev_dependencies:
 	build_runner: ^1.10.13
	json_serializable: ^3.5.1
```

- 创建模型类 user.dart

```dart
import 'package:json_annotation/json_annotation.dart';

/// This allows the `User` class to access private members in
/// the generated file. The value for this is *.g.dart, where
/// the star denotes the source file name.
part 'user.g.dart';

/// An annotation for the code generator to know that this class needs the
/// JSON serialization logic to be generated.
@JsonSerializable()

class User {
  User(this.name, this.email);

  String name;
  String email;

  /// A necessary factory constructor for creating a new User instance
  /// from a map. Pass the map to the generated `_$UserFromJson()` constructor.
  /// The constructor is named after the source class, in this case, User.
  factory User.fromJson(Map<String, dynamic> json) => _$UserFromJson(json);

  /// `toJson` is the convention for a class to declare support for serialization
  /// to JSON. The implementation simply calls the private, generated
  /// helper method `_$UserToJson`.
  Map<String, dynamic> toJson() => _$UserToJson(this);
}
```

- 执行命令 flutter pub run build_runner build 后生成 user.g.dart 文件 

![image-20210122170901165.png](https://i.loli.net/2021/03/11/41H8cOWdCIlamjG.png)

### 11.延迟加载库

![image-20210122170901165.png](https://i.loli.net/2021/03/11/41H8cOWdCIlamjG.png)

### 12.获取屏幕宽高

```dart
double _screenHeight = MediaQuery.of(context).size.height
double _screenWidth = MediaQuery.of(context).size.width
```

### 13.解决 bottomNavigationBar切换 tabbar 刷新页面，原有状态消失问题

- AutomaticKeepAliveClientMixin 
- IndexedStack

```dart
Scaffold(
    bottomNavigationBar: _buildBottomNavigationBar(),
    body: IndexedStack( // indexedStack 可以确保 tabbar 时候不刷新页面
        index: _selectedIndex,
        children: _contentListView,
    ),
);
```

### 14.切换 tabbar 保存状态

```dart
class _HomeRecommednedState extends State<HomeRecommedned>
    with AutomaticKeepAliveClientMixin {
  @override
    bool get wantKeepAlive => true; //  with AutomaticKeepAliveClientMixin + 设置 wantKeepAlive 为 true，确保顶部切换 tabbar 保存 state
  }
```

### 15.FutrueBuilder / StreamBuilder

作用：异步 UI 更新。两者都可以用于接收异步事件数据。

区别：

`futureBuilder` 只有一个响应，类似于 js 中的 Promise。

`streamBuilder` 可以接收多个异步操作的结果，常用于多次读取数据的异步任务场景。

### 16. Flutter 布局约束

链接：https://flutter.cn/docs/development/ui/layout/constraints