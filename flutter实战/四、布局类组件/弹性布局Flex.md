## 简介

Flex可以用来实现让其子Widget按比例均分的效果。

由于`Row`和`Column`都继承自`Flex`，参数基本相同，只有一个方向必须传入：

```dart
Flex({
  ...
  required this.direction, //弹性布局的方向, Row默认为水平方向，Column默认为垂直方向
  List<Widget> children = const <Widget>[],
})
```

## Expanded

此组件只能作为Flex的子Widget，可以按比例“扩伸”自己的size。

因为 `Row`和`Column` 都继承自 Flex，所以 Expanded 也可以作为它们的孩子。

```dart
const Expanded({
  int flex = 1, 
  required Widget child,
})
```

`flex`参数为弹性系数，默认值为1，如果为 0 或`null`，则`child`是没有弹性的，即不会被扩伸占用的空间。

sample：

```dart
Column(
  children: [
    Flex(
      direction: Axis.horizontal,
      children: [
        Expanded(
            flex: 1, child: Container(height: 30, color: Colors.red)),
        Expanded(
            flex: 2, child: Container(height: 30, color: Colors.green))
      ],
    ),
    //下面这个expanded默认flex=1，会占据父Colomn剩余的全部空间
    Expanded(
      child: Padding(
        padding: const EdgeInsets.only(top: 20),
        child: Column(
          children: [
            Expanded(flex: 2, child: Container(height: 50, color: Colors.red)),
            const Spacer(flex: 1), //Spacer的功能是占用指定比例的空间，实际上它只是Expanded的一个包装类
            Expanded(child: Container(height: 50, color: Colors.green)),
          ],
        ),
      )
    ),
  ],
),
```

