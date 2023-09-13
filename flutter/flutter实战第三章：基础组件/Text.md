### Text基础属性

- textAlign：  对齐方式
- maxLines
- overflow：TextOverflow枚举类的类型，显示不下的显示方式
- textScaleFactor：当前文字的缩放因子，一般是跟随系统字体大小设置改变时全局变化

### TextStyle

- fontSize：字号
- fontFamily：字体
- height：行高因子，具体的行高等于`fontSize`*`height`，不影响字体大小，默认影响上边距。

```dart
Text("Hello world",
  style: TextStyle(
    color: Colors.blue,
    fontSize: 18.0,
    height: 1.2,  
    fontFamily: "Courier",
    background: Paint()..color=Colors.yellow,
    decoration:TextDecoration.underline,
    decorationStyle: TextDecorationStyle.dashed
  ),
);
```

### TextSpan

富文本时使用，可以定义自己的`style`并且允许多层嵌套。配合`Text.rich()`使用：

```dart
Text.rich(TextSpan(
    children: [
     TextSpan(
       text: "Home: "
     ),
     TextSpan(
       text: "https://flutterchina.club",
       style: TextStyle(
         color: Colors.blue
       ),  
       recognizer: _tapRecognizer
     ),
    ]
))
```

这个`recognizer`是点击的处理器。

### DefaultTextStyle

作为一系列子Text的默认Style，子Text会继承这个样式

```dart
DefaultTextStyle(
  //1.设置文本默认样式  
  style: TextStyle(
    color:Colors.red,
    fontSize: 20.0,
  ),
  textAlign: TextAlign.start,
  child: Column(
    crossAxisAlignment: CrossAxisAlignment.start,
    children: <Widget>[
      Text("hello world"),
      Text("I am Jack"),
      Text("I am Jack",
        style: TextStyle(
          inherit: false, //2.不继承默认样式
          color: Colors.grey
        ),
      ),
    ],
  ),
);
```

### 字体

使用新字体需要先在asset中声明，然后在`TextStyle#fontFamily`中使用即可。不详述。