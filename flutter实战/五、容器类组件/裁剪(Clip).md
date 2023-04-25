## 裁剪类组件

| 剪裁Widget | 默认行为                                                     |
| ---------- | ------------------------------------------------------------ |
| ClipOval   | 裁剪成一个内切圆，子组件为正方形时裁剪成圆形；为矩形时，裁剪成椭圆 |
| ClipRRect  | 将子组件裁剪为圆角矩形                                       |
| ClipRect   | 默认裁剪掉子组件布局空间之外的绘制内容（溢出部分裁剪）       |
| ClipPath   | 按照自定义的路径裁剪                                         |

## 栗子

```dart
import 'package:flutter/material.dart';

class ClipTestRoute extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    // 头像  
    Widget avatar = Image.asset("imgs/avatar.png", width: 60.0);
    return Center(
      child: Column(
        children: <Widget>[
          avatar, //不剪裁
          ClipOval(child: avatar), //剪裁为圆形
          ClipRRect( //剪裁为圆角矩形
            borderRadius: BorderRadius.circular(5.0), //圆角半径
            child: avatar,
          ), 
          Row(
            mainAxisAlignment: MainAxisAlignment.center,
            children: <Widget>[
              Align(
                alignment: Alignment.topLeft,
                widthFactor: .5,//宽度设为原来宽度一半，另一半会溢出
                child: avatar,
              ),
              Text("你好世界", style: TextStyle(color: Colors.green),)
            ],
          ),
          Row(
            mainAxisAlignment: MainAxisAlignment.center,
            children: <Widget>[
              ClipRect( //将溢出部分剪裁
                child: Align(
                  alignment: Alignment.topLeft,
                  widthFactor: .5,//宽度设为原来宽度一半
                  child: avatar,
                ),
              ),
              Text("你好世界",style: TextStyle(color: Colors.green))
            ],
          ),
        ],
      ),
    );
  }
}
```

## 自定义裁剪

可以看到，上面的栗子中使用的`ClipRect`只是将超出Widget区域的部分裁剪了，如果想要自定义裁剪区域，则需要自定义一个`CustomClipper`：

```dart
class MyClipper extends CustomClipper<Rect> {
  @override
  Rect getClip(Size size) => Rect.fromLTWH(10.0, 15.0, 40.0, 30.0);

  @override
  bool shouldReclip(CustomClipper<Rect> oldClipper) => false;
}
```

## ClipPath

使用`ClipPath`的思路和上面自定义裁剪相同，需要继承`CustomClipper<Path>`，然后在`getClip`中返回一个`Path`即可。