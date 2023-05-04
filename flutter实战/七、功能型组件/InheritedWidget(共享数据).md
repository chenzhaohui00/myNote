## 简介

想想如果我们想要在整个控件树中共享一些数据或者对象要怎么做？最简单的当然是直接写在router类中，然后在widiget的build方法中访问这些数据或变量，但更多时候我们的Widget并不是都写在router类中的，那要怎么做呢？

似乎只能通过构造函数传递了，那如果需要共享数据的两个widget之间隔了五六层widget，难道要把数据或者对象一层层传下来？这显然可读性和可维护性太差了。

针对这种普遍的数据共享的需求，Flutter 提供了`InheritedWidget`，我们需要做的就是定义一个`InheritedWidget`的子类，在其中包裹数据并实现一个方法返回当此widget rebuild时是否需要通知依赖他的组件更新。

示例如下：

```dart
//继承InheritedWidget组件共享数据
class ShareIntDataWidget extends InheritedWidget {
  //待共享的数据
  final int data;

  const ShareIntDataWidget(
      {super.key, required this.data, required super.child});

  //rebuild时是否更新子组件
  @override
  bool updateShouldNotify(covariant ShareIntDataWidget oldWidget) {
    return data != oldWidget.data;
  }

  //提供一个方便的of方法得到此对象
  static ShareIntDataWidget of(BuildContext ctx) {
    // return ctx.dependOnInheritedWidgetOfExactType<ShareIntDataWidget>();
    return ctx.getElementForInheritedWidgetOfExactType<ShareIntDataWidget>()!.widget as ShareIntDataWidget;
  }
}
```

上面的`of`方法中提供了它在控件树中的孩子都可以很方便地获取到他的方式，也是两个常用的方法是：

```dart
//获取InheritedElement，然后调用InheritedElement.widget获取到InheritedWidget对象
InheritedElement? getElementForInheritedWidgetOfExactType<T extends InheritedWidget>();
//直接获取到InheritedWidget对象，并且依赖InheritedWidget，也就是订阅其变化
T? dependOnInheritedWidgetOfExactType<T extends InheritedWidget>({ Object? aspect });
```

如注释，这两个方法的区别就是，是否会在获取其数据的同时依赖此`InheritedWidget`，也就是订阅其变化。

然后我们就可以在子控件中使用其数据了：

```dart
class _TestWidget extends StatefulWidget {
  @override
  State<StatefulWidget> createState() => _TestWidgetState();
}

class _TestWidgetState extends State<_TestWidget> {
  @override
  Widget build(BuildContext context) {
    return Text(ShareIntDataWidget.of(context)!.data.toString(), textScaleFactor: 3);
  }

  @override
  void didChangeDependencies() {
    super.didChangeDependencies();
    debugPrint('didChangeDependencies');
  }
}
```

最后是route中使用的代码：

```dart
import 'package:flutter/material.dart';

class InheritedWidgetRoute extends StatefulWidget {
  const InheritedWidgetRoute({super.key});

  @override
  State<StatefulWidget> createState() => _InheritedWidgetRouteState();
}

class _InheritedWidgetRouteState extends State<InheritedWidgetRoute> {
  var _data = 5;

  @override
  Widget build(BuildContext context) {
    return Material(
      child: Column(mainAxisAlignment: MainAxisAlignment.center, children: [
        ShareIntDataWidget(data: _data, child: _TestWidget()),
        ElevatedButton(
            onPressed: () {
              setState(() {
                _data++;
              });
            },
            child: const Text(
              'Add Data',
            )
        )
      ]),
    );
  }
}
```