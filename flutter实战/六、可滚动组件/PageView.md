## 构造方法

```dart
PageView({
  Key? key,
  this.scrollDirection = Axis.horizontal, // 滑动方向
  this.reverse = false,
  PageController? controller,
  this.physics,
  List<Widget> children = const <Widget>[],
  
  //切页回调，入参为index页码
  this.onPageChanged,
  //每次滑动是否强制切换整个页面，如果为false，则会根据实际的滑动距离显示页面
  this.pageSnapping = true,
  //主要是配合辅助功能用的，后面解释
  this.allowImplicitScrolling = false,
  //后面解释
  this.padEnds = true,
})
```



## 缓存

设置`pageSnapping=true`时默认会缓存当前页的前后一页，否则不会有缓存。

另外测试时发现，如果滑动到另一页时滑动过大，在滑动到下一页时会有一个继续滑动并弹回的效果，此时下下个的Widget也会build。