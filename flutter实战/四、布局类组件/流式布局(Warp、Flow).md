我们把超出屏幕显示范围会自动折行的布局称为流式布局。Flutter中通过`Wrap`和`Flow`来支持流式布局。

## Wrap

```dart
Wrap({
  ...
  this.direction = Axis.horizontal, //方向
  this.alignment = WrapAlignment.start, //主轴方向子组件的对齐方式
  this.spacing = 0.0, //主轴方向子组件的间距
  this.runAlignment = WrapAlignment.start, //折行后，纵轴方向内所有行的对齐
  this.runSpacing = 0.0, //折行后，纵轴方向的行间距
  this.crossAxisAlignment = WrapCrossAlignment.start, //纵轴方向单行内子组件的对齐方式
  this.textDirection, //水平方向的子组件布局顺序，用于确定alignment对齐的参考系，即：textDirection的值为TextDirection.ltr，则alignment的start代表左，end代表右，即从左往右的顺序
  this.verticalDirection = VerticalDirection.down, //纵轴布局方向，up向上，down向下
  List<Widget> children = const <Widget>[],
})
```

sample

```dart
Wrap(
  direction: Axis.horizontal,
  alignment: WrapAlignment.center,
  spacing: 20,
  //多个行在纵轴上的对齐
  runAlignment: WrapAlignment.end,
  //同一行的不同Widget在纵轴方向上的对齐
  crossAxisAlignment: WrapCrossAlignment.start,
  //行间距
  runSpacing: 2,
  children: const [
    Chip(
      label: Text('张飞'),
      avatar: CircleAvatar(
        backgroundColor: Colors.blue,
        child: Text('张'),
      ),
    ),
    Chip(
      label: Text('赵子龙'),
      avatar: CircleAvatar(
        backgroundColor: Colors.blue,
        child: Text('赵'),
      ),
    ),
    Chip(
      label: Text('孙中山'),
      avatar: CircleAvatar(
        backgroundColor: Colors.blue,
        child: Text('孙'),
      ),
    ),
    Chip(
      label: Text('李宗盛'),
      avatar: CircleAvatar(
        backgroundColor: Colors.blue,
        child: Text('李'),
      ),
    ),
    Chip(
      label: Text('周树人'),
      avatar: CircleAvatar(
        backgroundColor: Colors.blue,
        child: Text('周'),
      ),
    ),
    Text(
      'Text',
      style: TextStyle(fontSize: 20),
    )
  ],
)
```

## Flow

`Flow`用来做自定义layout实现的流式布局。

自定义方式是实现一个`FlowDelegate`，在其`paintChildren()`中根据传入的`FlowPaintingContext`获取到自己的size以及当前要计算的child的size，计算出要绘制的坐标，然后调用`context.paintChild()`进行绘制。

`FlowDelegate`另外还有两个方法需要实现，sample如下：

```dart
class TestFlowDelegate extends FlowDelegate {
  EdgeInsets margin;

  TestFlowDelegate({this.margin = EdgeInsets.zero});

  @override
  void paintChildren(FlowPaintingContext context) {
    var x = margin.left;
    var y = margin.top;
    for (int i = 0; i < context.childCount; i++) {
      //1. 获取第i个child的右坐标
      var right = x + context.getChildSize(i)!.width + margin.right;
      //2. 比较是否已经超过了整体的最大宽度
      if (right < context.size.width) {
        //没有超过，直接绘制
        context.paintChild(i, transform: Matrix4.translationValues(x, y, 0));
        x = right;
      } else {
        //超过了，x归零从左面画起，y增加一行的高度
        x = margin.left;
        y += context.getChildSize(i)!.height + margin.bottom + margin.top;
        context.paintChild(i, transform: Matrix4.translationValues(x, y, 0.0));
        x = x + context.getChildSize(i)!.width + margin.right;
      }
    }
  }

  @override
  Size getSize(BoxConstraints constraints) {
    // 简单写法，实际开发中我们需要根据子元素所占用的具体宽高来设置Flow大小
    return const Size(double.infinity, 200.0);
  }

  @override
  bool shouldRepaint(covariant FlowDelegate oldDelegate) {
    return oldDelegate != this;
  }
}
```

Flow的使用：

```dart
Flow(
  delegate: TestFlowDelegate(margin: const EdgeInsets.all(9)),
  children: [
    Container(width: 80, height: 80, color: Colors.red),
    Container(width: 80, height: 80, color: Colors.green),
    Container(width: 80, height: 80, color: Colors.blue),
    Container(width: 80, height: 80, color: Colors.black),
    Container(width: 80, height: 80, color: Colors.pink),
    Container(width: 80, height: 80, color: Colors.yellow),
  ],
)
```

