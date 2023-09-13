## Padding

```dart
Padding({
  ...
  EdgeInsetsGeometry padding,
  Widget child,
})
```

`EdgeInsetsGeometry`是一个抽象类，开发中，我们一般都使用`EdgeInsets`类，它是`EdgeInsetsGeometry`的一个子类，定义了一些设置填充的便捷方法。

## EdgeInsets

`EdgeInsets`提供的便捷方法：

- `fromLTRB(double left, double top, double right, double bottom)`：分别指定四个方向的填充。
- `all(double value)` : 所有方向均使用相同数值的填充。
- `only({left, top, right ,bottom })`：可以设置具体某个方向的填充(可以同时指定多个方向)。
- `symmetric({ vertical, horizontal })`：用于设置对称方向的填充，`vertical`指`top`和`bottom`，`horizontal`指`left`和`right`。