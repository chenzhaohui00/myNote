# Widget简介

- `Widget`不等同于原生开发时的UI控件，他也可以表示一些功能性的组件，比如`GestureDetector`、`Theme`等。
- Flutter中就是通过嵌套`Widget`来构建UI的，在Flutter中万物皆`Widget`
- `Widget`的功能是“描述一个UI元素的配置信息”，比如对于 Text 来讲，文本的内容、对齐方式、文本样式都是它的配置信息。

## Widget类的重要成员

Key：Widget类中有一个成员Key，作用是决定是否在下一次`build`时复用旧的 `widget` 

canUpdate()：用来判断是否可以旧 `widget`的方法，里面使用了Key来做判断

createElement(): 创建对应的`Element`

## Flutter中的四棵树

1. `Widget`树
2. `Widget`树中的`Widget`一一对应生成的`Element`树
3. 通过`Element`树生成的`Render`树
4. 通过`Render`树生成的`Layer`树

## StatelessWidget

它是无状态的Widget，继承自Widget，重写了`createElement()`，返回一个`StatelessElement`。只要在`build`方法中返回嵌套的`widget`即可。

#### context

`build`方法有一个`context`参数，它是一个`BuildContext`，表示当前`widget`在`widget`树中的上下文。通过`context`可以对当前 widget 在 widget 树中位置中执行“相关操作”，比如它提供了从当前 widget 开始向上遍历 widget 树以及按照 widget 类型查找父级 widget 的方法。

## StatefulWidget

- `StatefulWidget`对应的是`StatefulElement`。
- `StatefulWidget`添加了一个`createState()`方法，用来创建和它相关的状态。
- `createState()`方法可能会被调用多次，例如，当一个 `StatefulWidget` 同时插入到 widget 树的多个位置时，Flutter 框架就会调用该方法为每一个位置生成一个独立的`State`实例

## State

#### 简介

一个 StatefulWidget 类会对应一个 State 类，State 中的保存的状态信息可以被`widget`读取，当 State 变化时，通过`setState()`方法通知`Flutter`框架发生变化，后者会重新调用`build`方法构建`widget`树。

`State`中有两个常用属性：

1.`widget`，即与当前`State`实例关联的`widget`实例。由于`state`实例不会因重新构建而变化，所以在重新构建后，如果`widget`实例遍了，`state`内的这个`widget`变量会指向新的`widget`实例。

2.`context`，其对应的`widget`实例的`BuildContext`对象。

#### State生命周期

生命周期图如下所示：

![img](../pictures/2-5.a59bef97.png)

- `initState`：初始化方法，只会调用一次。此回调中不能调用`BuildContext.dependOnInheritedWidgetOfExactType`
- `didChangeDependencies()`：当State对象的依赖发生变化时会被调用；
- `build()`:回调场景：
  - 在调用`initState()`之后。
  - 在调用`didUpdateWidget()`之后。
  - 在调用`setState()`之后。
  - 在调用`didChangeDependencies()`之后。
  - 在State对象从树中一个位置移除后（会调用deactivate）又重新插入到树的其他位置之后。
- `reassemble()`：此回调是专门为了开发调试而提供的，在热重载(hot reload)时会被调用，此回调在Release模式下永远不会被调用。
- `didUpdateWidget ()`：当 State 对象从树中被移除时，会调用此回调。如果移除后没有重新插入到树中则紧接着会调用`dispose()`方法。
- 如果移除后没有重新插入到树中则紧接着会调用`dispose()`方法。

## 获取Widget的state对象

#### 通过context

```dart
ScaffoldState _state = context.findAncestorStateOfType<ScaffoldState>()!;
```

#### 默认约定：of方法

由于1方法是无视state是否为私有的都能获取到。为了区分是否向外暴露State对象，在 Flutter 开发中便有了一个默认的约定：如果 StatefulWidget 的状态是希望暴露出的，应当在 StatefulWidget 中提供一个`of` 静态方法来获取其 State 对象，开发者便可直接通过该方法来获取；如果 State不希望暴露，则不提供`of`方法。这个约定在 Flutter SDK 里随处可见。

```dart
Builder(builder: (context) {
  return ElevatedButton(
    onPressed: () {
      // 直接通过of静态方法来获取ScaffoldState
      ScaffoldState _state=Scaffold.of(context);
      // 打开抽屉菜单
      _state.openDrawer();
    },
    child: Text('打开抽屉菜单2'),
  );
}),
```

使用：

```dart
Builder(builder: (context) {
  return ElevatedButton(
    onPressed: () {
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text("我是SnackBar")),
      );
    },
    child: Text('显示SnackBar'),
  );
}),
```

#### 通过GlobalKey

1.给目标`StatefulWidget`添加`GlobalKey`。

```dart
//定义一个globalKey, 由于GlobalKey要保持全局唯一性，我们使用静态变量存储
static GlobalKey<ScaffoldState> _globalKey= GlobalKey();
...
Scaffold(
    key: _globalKey , //设置key
    ...  
)
```

2.通过`GlobalKey`来获取`State`对象

```dart
_globalKey.currentState.openDrawer()
```

> 1. 使用 GlobalKey 开销较大，如果有其他可选方案，应尽量避免使用它
> 1. 同一个 GlobalKey 在整个 widget 树中必须是唯一的，不能重复

## 通过RenderObject自定义Widget

`RenderObject`是定义`Widget`最原始的方式，而`StatelessWidget`和`StatefulWidget`只是两个帮助自定义`Widget`的工具类。

```dart
class CustomWidget extends LeafRenderObjectWidget{
  @override
  RenderObject createRenderObject(BuildContext context) {
    // 创建 RenderObject
    return RenderCustomObject();
  }
  @override
  void updateRenderObject(BuildContext context, RenderCustomObject  renderObject) {
    // 更新 RenderObject
    super.updateRenderObject(context, renderObject);
  }
}

class RenderCustomObject extends RenderBox{

  @override
  void performLayout() {
    // 实现布局逻辑
  }

  @override
  void paint(PaintingContext context, Offset offset) {
    // 实现绘制
  }
}
```

不包含子组件的`Widget`可以直接继承`LeafRenderObjectWidget`，后者继承于`Widget`，并在createElement中返回了一个`LeafRenderObjectElement`

```dart
abstract class LeafRenderObjectWidget extends RenderObjectWidget {
  const LeafRenderObjectWidget({ Key? key }) : super(key: key);

  @override
  LeafRenderObjectElement createElement() => LeafRenderObjectElement(this);
}
```

如果自定义的 widget 可以包含子组件，则可以根据子组件的数量来选择继承SingleChildRenderObjectWidget 或 MultiChildRenderObjectWidget，它们也实现了createElement() 方法，返回不同类型的 Element 对象。

## Flutter SDK内置组件库

#### 基础组件

导入：

```dart
import 'package:flutter/widgets.dart';
```

常用组件：

- `Text`: 文本
- `Row`、`Column`: 行和列
- `Stack`：类似Android中的`FrameLayout`，允许子`Widget`堆叠
- `Container`：矩形，可以装饰一个`BoxDecoration`，比如background、边框或阴影。也可以有margin、padding和应用于其大小的约束(constraints)

如果我们的应用中引入了 Material 和 Cupertino 两个库之一，则不需要再引入`flutter/ widgets.dart`了，因为它们内部已经引入过了。

#### Material组件

android风格的组件。比如前面使用到的`Scaffold`、`AppBar`、`TextButton`等，导入：

```dart
import 'package:flutter/material.dart';
```

在 Material 组件库中有一些组件可以根据实际运行平台来切换表现风格，比如`MaterialPageRoute`，在路由切换时，如果是 Android 系统，它将会使用 Android 系统默认的页面切换动画(从底向上)；如果是 iOS 系统，它会使用 iOS 系统默认的页面切换动画（从右向左）。

#### Cupertino组件

ios风格的组件。导入：

```dart
import 'package:flutter/material.dart';
```