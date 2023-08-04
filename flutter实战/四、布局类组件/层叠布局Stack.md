Stack允许子组件相互覆盖，相当于Android中的FrameLayout，通过指定子组件的绝对位置进行布局。

## Stack

```dart
Stack({
  this.alignment = AlignmentDirectional.topStart, //横轴或(和)纵轴没有定位时的对齐方式
  this.textDirection, //辅助alignment确定其参考系是从左到右还是从右到左
  this.fit = StackFit.loose, //非Postioned子组件如何适配Stack的大小，StackFit.loose表示使用子组件的大小，StackFit.expand表示扩伸到Stack的大小。
  this.clipBehavior = Clip.hardEdge, //决定对超出Stack显示空间的部分如何剪裁
  List<Widget> children = const <Widget>[],
})
```

## Positioned

`Positioned`作为`Stack`的子组件，包裹一个被定位的组件，构造函数如下：

```dart
const Positioned({
  Key? key,
  this.left, 
  this.top,
  this.right,
  this.bottom,
  this.width,
  this.height,
  required Widget child,
})
```

注意：

1. 只能指定`left`、`right`、`width`三个属性中的两个，如指定`left`和`width`后，`right`会自动算出(`left`+`width`)，如果同时指定三个属性则会报错，垂直方向同理。
2. 如果这六个定位参数都不用，就相当于是没有用Positioned，直接是非Positioned的child

## sample

```dart
SizedBox(
  width: double.infinity,
  height: double.infinity,
  child: Stack(
    alignment: AlignmentDirectional.center,
    fit: StackFit.expand, //未定位的子组件的size扩展到最大
    children: [
      const Positioned(left: 18, child: Text('left:18')), //此文本会被下面的红框文本挡住
      Container(
        color: Colors.red,
        child: const Text(
          'text',
          style: TextStyle(color: Colors.white),
        ),
      ),
      const Positioned(top: 18, child: Text('top:18'))
    ],
  ),
)
```

## Stack的measure流程

Stack的子组件也分为两类，一类是Positioned，一类是非Positioned。

如果有非Positioned的子组件，则Stack自己的大小会是能包裹得下其中最大的那个非Positioned子组件的大小。

如果所有子组件都是Positioned，则Stack会是满足父类约束的尽量大，然后在根据子组件的定位给他们layout。