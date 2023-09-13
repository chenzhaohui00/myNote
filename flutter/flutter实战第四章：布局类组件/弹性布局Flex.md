## 简介

Flex类似于Android中的LinearLayout，是用来做单方向的线性的布局组件，`Row`和`Column`都是它的子类。另外它可以用来实现让其子Widget按比例均分的效果。

`Row`和`Column`都继承自`Flex`，`Flex`的参数与这两者基本相同，只有一个方向必须传入：

```dart
Flex({
  ...
  required this.direction, //弹性布局的方向, Row默认为水平方向，Column默认为垂直方向
  List<Widget> children = const <Widget>[],
})
```

## Expanded

前面说了，Flex可以实现子组件的等比例分配，这个功能就是通过`Expanded`组件实现。

- `Expanded`组件只能作为Flex的子Widget，因为 `Row`和`Column` 都继承自 Flex，所以 Expanded 也可以作为它们的孩子。
- 继承自`Flexible`，可以按比例“扩伸”自己的size。

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

注意，上面的Expanded的child中的Chontainer写的这个width或height都会起效。

## Flex的measure流程

`Flex`的child分为两种，一种是`Flexible`，一种是非`Flexible`。`Flexible`是可以扩缩的widget，不确定自己的尺寸，会根据flex参数动态分配`Flex`的剩余空间。非`Flexible`是自己能确定尺寸的widget。

所以`Flex`的measure流程就是：

1. 先告诉所有的非`Flexible`的child一个0~double.infinity的松约束，让他们随便measure自己的宽/高。
2. 根据所有非`Flexible`的child的宽/高，计算剩余的尺寸
3. 剩余的`Flexible`的child根据flex参数，均分尺寸。

### 典型问题：Column包裹ListView

理解了Flex的measure流程，就知道了这个典型问题的答案。

Column作为一个Flex，会给ListView一个0~double.infinity的松约束，而ListView是必须要一个固定的主轴长度的，于是就会报错。
