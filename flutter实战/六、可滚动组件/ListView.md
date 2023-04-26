## ListView 构造

```dart
ListView({
  ...  
  //可滚动widget公共参数
  Axis scrollDirection = Axis.vertical,
  bool reverse = false,
  ScrollController? controller,
  bool? primary,
  ScrollPhysics? physics,
  EdgeInsetsGeometry? padding,
  
  //ListView各个构造函数的共同参数  
  double? itemExtent,
  Widget? prototypeItem, //列表项原型，不指定itemExtent时用来计算item长度/宽度
  bool shrinkWrap = false,
  bool addAutomaticKeepAlives = true,
  bool addRepaintBoundaries = true,
  double? cacheExtent, // 预渲染区域长度
    
  //子widget列表
  List<Widget> children = const <Widget>[],
})
```

- `itemExtent`：extent 是范围、长度的意思，`itemExtent`就是item的长度，当主轴方向为横向时，即为item的宽度。
- `prototypeItem`：不指定itemExtent时用来计算item长度/宽度
- `shrinkWrap`：是否根据ListView的子组件的总长度来设置`ListView`的长度，默认为 false，默认`ListView`会设置为可能的最长的长度。当`ListView`在一个主轴无边界的容器中时，`shrinkWrap`必须为`true`。
- `addRepaintBoundaries`：是否给每个子组件包裹一个“绘制边界”以避免不必要的重绘，但是当列表项重绘的开销非常小（如一个颜色块，或者一个较短的文本）时，不添加`RepaintBoundary`反而会更高效。如果列表项自身来维护是否需要添加绘制边界组件，则此参数应该指定为 false。
- `children`：子widget列表，即使使用此参数构建`ListView`，列表也是懒加载的，等到真正需要加载的时候才会对 Widget 进行布局和绘制。

> 创建滚动列表时，通过`itemExtent`或`prototypeItem`指定高度是一个好习惯，可以提供性能



## ListView.builder构造

通过`ListView.builder`构造创建列表项的`ListView`：

```dart
ListView.builder({
  // ListView公共参数已省略  
  ...
  required IndexedWidgetBuilder itemBuilder,
  int itemCount,
  ...
})
```

- `itemBuilder`：它是列表项的构建器，类型为`IndexedWidgetBuilder`，返回值为一个widget。当列表滚动到具体的`index`位置时，会调用该构建器构建列表项。
- `itemCount`：列表项的数量，如果为`null`，则为无限列表。



## 添加分隔线(ListView.separated构造)

```dart
class ListView3 extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    //下划线widget预定义以供复用。  
    Widget divider1=Divider(color: Colors.blue,);
    Widget divider2=Divider(color: Colors.green);
    return ListView.separated(
      itemCount: 100,
      //列表项构造器
      itemBuilder: (BuildContext context, int index) {
        return ListTile(title: Text("$index"));
      },
      //分割器构造器
      separatorBuilder: (BuildContext context, int index) {
        return index%2==0?divider1:divider2;
      },
    );
  }
}
```



## 通过prototypeItem固定高度

```dart
class FixedExtentList extends StatelessWidget {
  const FixedExtentList({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return ListView.builder(
   		prototypeItem: ListTile(title: Text("1")),
      //itemExtent: 56,
      itemBuilder: (context, index) {
        //LayoutLogPrint是一个自定义组件，在布局时可以打印当前上下文中父组件给子组件的约束信息
        return LayoutLogPrint(
          tag: index, 
          child: ListTile(title: Text("$index")),
        );
      },
    );
  }
}
```



## 添加Header

如果仅仅是在`Column`中添加一个`ListView`，可以这样写：

```dart
@override
Widget build(BuildContext context) {
  return Column(children: <Widget>[
    ListTile(title:Text("商品列表")),
    ListView.builder(itemBuilder: (BuildContext context, int index) {
        return ListTile(title: Text("$index"));
    }),
  ]);
}
```

但是这时会发现ListView的高度无法确定，触发以下错误：

```text
Error caught by rendering library, thrown during performResize()。
Vertical viewport was given unbounded height ...
```

正确的解决思路是，通过`Flex`分配好`ListView`的高度

```dart
@override
Widget build(BuildContext context) {
  return Column(children: <Widget>[
    ListTile(title:Text("商品列表")),
    Expanded(
      child: ListView.builder(itemBuilder: (BuildContext context, int index) {
        return ListTile(title: Text("$index"));
      }),
    ),
  ]);
}
```



## 实例

实现一个比较贴近现实业务场景的实例，实现一个有分页加载功能、底部有加载更多/到底了提示的`ListView`：

```dart
import 'package:flutter/material.dart';
import 'package:english_words/english_words.dart';
import 'package:flutter/rendering.dart';

class InfiniteListView extends StatefulWidget {
  @override
  _InfiniteListViewState createState() => _InfiniteListViewState();
}

class _InfiniteListViewState extends State<InfiniteListView> {
  static const loadingTag = "##loading##"; //表尾标记
  var _words = <String>[loadingTag];

  @override
  void initState() {
    super.initState();
    _retrieveData();
  }

  @override
  Widget build(BuildContext context) {
    return ListView.separated(
      itemCount: _words.length,
      itemBuilder: (context, index) {
        //如果到了表尾
        if (_words[index] == loadingTag) {
          //不足100条，继续获取数据
          if (_words.length - 1 < 100) {
            //获取数据
            _retrieveData();
            //加载时显示loading
            return Container(
              padding: const EdgeInsets.all(16.0),
              alignment: Alignment.center,
              child: SizedBox(
                width: 24.0,
                height: 24.0,
                child: CircularProgressIndicator(strokeWidth: 2.0),
              ),
            );
          } else {
            //已经加载了100条数据，不再获取数据。
            return Container(
              alignment: Alignment.center,
              padding: EdgeInsets.all(16.0),
              child: Text(
                "没有更多了",
                style: TextStyle(color: Colors.grey),
              ),
            );
          }
        }
        //显示单词列表项
        return ListTile(title: Text(_words[index]));
      },
      separatorBuilder: (context, index) => Divider(height: .0),
    );
  }

  void _retrieveData() {
    Future.delayed(Duration(seconds: 2)).then((e) {
      setState(() {
        //重新构建列表
        _words.insertAll(
          _words.length - 1,
          //每次生成20个单词
          generateWordPairs().take(20).map((e) => e.asPascalCase).toList(),
        );
      });
    });
  }
}
```

可以看到，主要思路就是：

1. `initState`时初始化首页数据
2. 数据中定义一个bottom的tag，在`ListView`的`build`方法中判断，当加载到了bottom时：
   1. 如果没有更多的数据，则返回到底了的样式的Widget。
   2. 加载新的数据，且返回加载更多样式的Widget。