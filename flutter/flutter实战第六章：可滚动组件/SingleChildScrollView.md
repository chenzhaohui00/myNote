## 简介

`SingleChildScrollView`相当于 Android 中的 `ScrollView`，**通常`SingleChildScrollView`只应在期望的内容不会超过屏幕太多时使用**，这是因为`SingleChildScrollView`不支持基于 Sliver 的延迟加载模型，`SingleChildScrollView`构造如下：

```dart
SingleChildScrollView({
  this.scrollDirection = Axis.vertical, //滚动方向，默认是垂直方向
  this.reverse = false, 
  this.padding, 
  bool primary, //是否使用 widget 树中默认的PrimaryScrollController
  this.physics, 
  this.controller,
  this.child,
})
```

`SingleChildScrollView`其实在build的时候内部了创建了一个`Scrollable`组件以实现滑动特效。

## Sample

```dart
class SingleChildScrollViewTestRoute extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    String str = "ABCDEFGHIJKLMNOPQRSTUVWXYZ";
    return Scrollbar( // 显示进度条
      child: SingleChildScrollView(
        padding: EdgeInsets.all(16.0),
        child: Center(
          child: Column( 
            //动态创建一个List<Widget>  
            children: str.split("") 
                //每一个字母都用一个Text显示,字体为原来的两倍
                .map((c) => Text(c, textScaleFactor: 2.0,)) 
                .toList(),
          ),
        ),
      ),
    );
  }
}
```