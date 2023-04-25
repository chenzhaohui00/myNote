## DecoratedBox

Decoration是装饰的意思，`DecoratedBox`看名字就知道是一个带装饰的盒布局控件。定义如下：

```dart
const DecoratedBox({
  Decoration decoration,
  DecorationPosition position = DecorationPosition.background,
  Widget? child
})
```

- `decoration`：代表将要绘制的装饰，它的类型为`Decoration`。
- position：此属性决定在哪里绘制 Decoration，它接收`DecorationPosition`的枚举类型，该枚举类有两个值：
  - `background`：在子组件之后绘制，即背景装饰。
  - `foreground`：在子组件之上绘制，即前景。

## BoxDecoration

`DecoratedBox`的`decoration`属性通常会直接使用`BoxDecoration`：

```dart
BoxDecoration({
  Color color, //颜色
  DecorationImage image,//图片
  BoxBorder border, //边框
  BorderRadiusGeometry borderRadius, //圆角
  List<BoxShadow> boxShadow, //阴影,可以指定多个
  Gradient gradient, //渐变
  BlendMode backgroundBlendMode, //背景混合模式
  BoxShape shape = BoxShape.rectangle, //形状
})
```

## Sample

```dart
 DecoratedBox(
   decoration: BoxDecoration(
     gradient: LinearGradient(colors:[Colors.red,Colors.orange.shade700]), //背景渐变
     borderRadius: BorderRadius.circular(3.0), //3像素圆角
     boxShadow: [ //阴影
       BoxShadow(
         color:Colors.black54,
         offset: Offset(2.0,2.0),
         blurRadius: 4.0
       )
     ]
   ),
  child: Padding(
    padding: EdgeInsets.symmetric(horizontal: 80.0, vertical: 18.0),
    child: Text("Login", style: TextStyle(color: Colors.white),),
  )
)
```