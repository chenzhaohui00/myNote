## Color

Flutter中也使用ARGB的方式表示一个Color，每个部分也是一个字节。Color类中一个颜色用一个int值保存。

### 将颜色字符串转成 Color 对象

```dart
Color(0xffdc380d); //如果颜色固定可以直接使用整数值
//颜色是一个字符串变量
var c = "dc380d";
Color(int.parse(c,radix:16)|0xFF000000) //通过位运算符将Alpha设置为FF
Color(int.parse(c,radix:16)).withAlpha(255)  //通过方法将Alpha设置为FF
```

### 颜色亮度

Color类有一个`computeLuminance()`方法，它可以返回一个[0-1]的一个值，数字越大颜色就越浅，可以据此实现一个根据背景色调整文字颜色的Widget，如下：

```dart
class AutoNavBar extends StatelessWidget {
  final Color color;
  final String title;

  const AutoNavBar({super.key, required this.title, required this.color});

  @override
  Widget build(BuildContext context) {
    return Container(
      height: 50,
      width: double.infinity,
      alignment: Alignment.center,
      color: color,
      child: Text(
        title,
        style: TextStyle(
          fontSize: 20,
            color:
                color.computeLuminance() > 0.5 ? Colors.black : Colors.white),
      ),
    );
  }
}
```

核心就是通过计算背景色的亮度，决定文字是黑色还是白色。

### MaterialColor

`MaterialColor`是实现Material Design中的颜色的类，它包含一种颜色的10个级别的渐变色。`MaterialColor`通过"[]"运算符的索引值来代表颜色的深度，有效的索引有：50，100，200，…，900，数字越大，颜色越深。`MaterialColor`的默认值为索引等于500的颜色。

我们可以根据 shadeXX 来获取具体索引的颜色，比如`Colors.blue.shade50`。



## Theme

通过`Theme`可以更好地复用颜色和字体样式，从而让整个 app 的设计看起来更一致。`Theme`是基于`InheritedWidget`实现的，所有子Widget都可以通过`Theme.of`获取到离自己最近的祖先Theme。

另外，应用顶部的`MaterialApp` Widget可以设置一个theme和一个darkTheme，对应Light Mode和Dark Mode。

### ThemeData

`ThemeData`是用来描述 `Theme` 的 bean类，实际上`Theme.of`获取的就是`ThemeData`对象。`ThemeData`的部分数据定义如下：

```dart
ThemeData({
  Brightness? brightness, //深色还是浅色
  MaterialColor? primarySwatch, //主题颜色样本，见下面介绍
  Color? primaryColor, //主色，决定导航栏颜色
  Color? cardColor, //卡片颜色
  Color? dividerColor, //分割线颜色
  ButtonThemeData buttonTheme, //按钮主题
  Color dialogBackgroundColor,//对话框背景颜色
  String fontFamily, //文字字体
  TextTheme textTheme,// 字体主题，包括标题、body等文字样式
  IconThemeData iconTheme, // Icon的默认样式
  TargetPlatform platform, //指定平台，应用特定平台控件风格
  ColorScheme? colorScheme, //包含12种颜色，可用于配置大多数组件的颜色属性
  ...
})
```

需要说明一下`primarySwatch`，它是主题颜色的一个"样本色"，通过这个样本色可以在一些条件下生成一些其他的属性，例如，如果没有指定`primaryColor`，并且当前主题不是深色主题，那么`primaryColor`就会默认为`primarySwatch`指定的颜色，还有一些相似的属性如`indicatorColor`也会受`primarySwatch`影响。

### 一个换肤Sample

```dart
class ThemeTestRoute extends StatefulWidget {
  @override
  _ThemeTestRouteState createState() => _ThemeTestRouteState();
}

class _ThemeTestRouteState extends State<ThemeTestRoute> {
  var _themeColor = Colors.teal; //当前路由主题色

  @override
  Widget build(BuildContext context) {
    ThemeData themeData = Theme.of(context);
    return Theme(
      data: ThemeData(
          primarySwatch: _themeColor, //用于导航栏、FloatingActionButton的背景色等
          iconTheme: IconThemeData(color: _themeColor) //用于Icon颜色
      ),
      child: Scaffold(
        appBar: AppBar(title: Text("主题测试")),
        body: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            //第一行Icon使用主题中的iconTheme
            Row(
                mainAxisAlignment: MainAxisAlignment.center,
                children: <Widget>[
                  Icon(Icons.favorite),
                  Icon(Icons.airport_shuttle),
                  Text("  颜色跟随主题")
                ]
            ),
            //为第二行Icon自定义颜色（固定为黑色)
            Theme(
              data: themeData.copyWith(
                iconTheme: themeData.iconTheme.copyWith(
                    color: Colors.black
                ),
              ),
              child: Row(
                  mainAxisAlignment: MainAxisAlignment.center,
                  children: <Widget>[
                    Icon(Icons.favorite),
                    Icon(Icons.airport_shuttle),
                    Text("  颜色固定黑色")
                  ]
              ),
            ),
          ],
        ),
        floatingActionButton: FloatingActionButton(
            onPressed: () =>  //切换主题
                setState(() =>
                _themeColor =
                _themeColor == Colors.teal ? Colors.blue : Colors.teal
                ),
            child: Icon(Icons.palette)
        ),
      ),
    );
  }
}
```

点击换肤按钮后，会发现第一行使用Theme颜色的icon发生了变化。