## Transform 矩阵变换

`Transform`组件可以对其子组件进行一些矩阵变换。比如下面通过`Matrix4`（4D矩阵）对子组件做沿y轴倾斜的操作，实现一个3D的效果：

```dart
Container(
  color: Colors.black,
  child: Transform(
    alignment: Alignment.topRight, //相对于坐标系原点的对齐方式
    transform: Matrix4.skewY(0.3), //沿Y轴倾斜0.3弧度
    child: Container(
      padding: const EdgeInsets.all(8.0),
      color: Colors.deepOrange,
      child: const Text('Apartment for rent!'),
    ),
  ),
)
```

## Transform做平移、旋转、缩放

Transform提供除了上面的直接用默认构造的方式外，还为平移、旋转、缩放提供了快捷的构造方法，使用如下：

```dart
//平移
DecoratedBox(
  decoration:BoxDecoration(color: Colors.red),
  //默认原点为左上角，左移20像素，向上平移5像素  
  child: Transform.translate(
    offset: Offset(-20.0, -5.0),
    child: Text("Hello world"),
  ),
)
//旋转
DecoratedBox(
  decoration:BoxDecoration(color: Colors.red),
  child: Transform.rotate(
    //旋转90度，使用弧度，不是角度，导包import 'dart:math' as math; 
    angle:math.pi/2 , 
    child: Text("Hello world"),
  ),
)
//缩放
DecoratedBox(
  decoration:BoxDecoration(color: Colors.red),
  child: Transform.scale(
    scale: 1.5, //放大到1.5倍
    child: Text("Hello world")
  )
);
```

## Transform的优缺点

### 缺点：大小和位置问题

`Transform`的变换是应用在绘制阶段，而并不是应用在布局(layout)阶段，所以无论对子组件应用何种变化，其占用空间的大小和在屏幕上的位置都是固定不变的，因为这些是在布局阶段就确定的。

所以使用了`Transform`变换的组件可能会发生位置和大小异常，容易出现和其他控件覆盖等问题。

### 优点：性能好

由于矩阵变化只会作用在绘制阶段，所以在某些场景下，在UI需要变化时，可以直接通过矩阵变化来达到视觉上的UI改变，而不需要去重新触发build流程，这样会节省layout的开销，所以性能会比较好。

## RotatedBox

`RotatedBox`和`Transform.rotate`功能相似，它们都可以对子组件进行旋转变换，但是有一点不同：`RotatedBox`的变换是在layout阶段，会影响在子组件的位置和大小。

```dart
Row(
  mainAxisAlignment: MainAxisAlignment.center,
  children: <Widget>[
    DecoratedBox(
      decoration: BoxDecoration(color: Colors.red),
      //将Transform.rotate换成RotatedBox  
      child: RotatedBox(
        quarterTurns: 1, //旋转90度(1/4圈)
        child: Text("Hello world"),
      ),
    ),
    Text("你好", style: TextStyle(color: Colors.green, fontSize: 18.0),)
  ],
),
```