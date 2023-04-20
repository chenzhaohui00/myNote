通过Align组件，可以便捷地调整子组件在父组件中的位置。

## Align

```dart
Align({
  Key key,
  this.alignment = Alignment.center,
  this.widthFactor,
  this.heightFactor,
  Widget child,
})
```

`widthFactor`和`heightFactor`这两个系数用来通过子组件确定自己组件大小，子组件的宽高乘对应的系数就是`Align`组件自己的宽高。当`widthFactor`或`heightFactor`为`null`时组件的宽高将会占用尽可能多的空间

### Sample

```dart
Align(
  widthFactor: 2,
  heightFactor: 2,
  alignment: Alignment.topRight,
  child: FlutterLogo(
    size: 60,
  ),
),
```

这就是一个底色为浅蓝色，右上角有一个FlutterLogo的`Align`组件。`Align`自己的宽高均为120，通过子组件的size以及factor系数的来。

这个alignment是对齐参数，它接收`AlignmentGeometry`类型。

### Alignment

`Alignment`继承自`AlignmentGeometry`，表示矩形内的一个点，他有两个属性`x`、`y`，分别表示在水平和垂直方向的偏移，`Alignment`定义如下：

```dart
Alignment(this.x, this.y)
```

`Alignment` Widget会以**矩形的中心点作为坐标原点**，即`Alignment(0.0, 0.0)` 。左上角为`Alignment(-1.0, -1.0)`，右下角为`Alignment(1.0, 1.0)`  。为了使用方便，矩形的原点、四个顶点，以及四条边的终点在`Alignment`类中都已经定义为了静态常量。

`Alignment`可以通过其**坐标转换公式**将其坐标转为子元素的具体偏移坐标：

```text
(Alignment.x*childWidth/2+childWidth/2, Alignment.y*childHeight/2+childHeight/2)
```

### FractionalOffset

`FractionalOffset` 继承自 `Alignment`，它和 `Alignment`唯一的区别就是坐标原点不同！`FractionalOffset` 的坐标原点为矩形的左侧顶点，这和布局系统的一致，所以理解起来会比较容易。`FractionalOffset`的坐标转换公式为：

```text
实际偏移 = (FractionalOffse.x * childWidth, FractionalOffse.y * childHeight)
```

## Center组件

`Center`其实就是一个确定了对齐方式的`Align`，它的构造如下：

```dart
class Center extends Align {
  const Center({ Key? key, double widthFactor, double heightFactor, Widget? child })
    : super(key: key, widthFactor: widthFactor, heightFactor: heightFactor, child: child);
}
```