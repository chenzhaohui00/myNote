## NestedScrollView

前面提到过， `CustomScrollView` 只能组合 Sliver，如果有孩子也是一个可滚动组件（通过 SliverToBoxAdapter 嵌入）且它们的滑动方向一致时便不能正常工作。需要使用`NestedScrollView` 组件来解决这个问题，我们看看它的定义：

```dart
const NestedScrollView({
  ... //省略可滚动组件的通用属性
  //header，sliver构造器
  required this.headerSliverBuilder,
  //可以接受任意的可滚动组件
  required this.body,
  this.floatHeaderSlivers = false,
}) 
```

`NestedScrollView`固定分成两部分，`headerSliver`和`body`，`headerSliverBuilder`相当于`CustomScrollView`的`slivers`属性，显示在主轴的前面，`body`用来接收一个可滚动组件，比如`ListView`、`CustomScrollView`，显示在主轴的后面。

比如下面这个栗子：

```dart
Material(
  child: NestedScrollView(
    headerSliverBuilder: (BuildContext context, bool innerBoxIsScrolled) {
      // 返回一个 Sliver 数组给外部可滚动组件。
      return <Widget>[
        SliverAppBar(
          title: const Text('嵌套ListView'),
          pinned: true, // 固定在顶部
          forceElevated: innerBoxIsScrolled,
        ),
        buildSliverList(5), //构建一个 sliverList
      ];
    },
    body: ListView.builder(
      padding: const EdgeInsets.all(8),
      physics: const ClampingScrollPhysics(), //重要
      itemCount: 30,
      itemBuilder: (BuildContext context, int index) {
        return SizedBox(
          height: 50,
          child: Center(child: Text('Item $index')),
        );
      },
    ),
  ),
);
```

header部分的slivers有两个，一个SliverAppBar，一个SliverList，body部分是一个ListView。也就是



## SliverAppBar

`SliverAppBar` 是 AppBar 的Sliver 版，大多数参数都相同。除此以外他还提供和`SliverPersistentHeader`相似的`pinned`和`floating`功能。他的一些独特的参数如下：

```dart
const SliverAppBar({
  this.collapsedHeight, // 收缩起来的高度
  this.expandedHeight,// 展开时的高度
  this.pinned = false, // 是否固定
  this.floating = false, //是否漂浮
  this.snap = false, // 当漂浮时，此参数才有效
  bool forceElevated //导航栏下面是否一直显示阴影
  ...
})
```

`pinned`功能和`SliverPersistentHeader`相同，`floating`和`snap`同时为true时提供和`SliverPersistentHeader`相似的功能，不同之处是，当用户反向滑动时，`SliverPersistentHeader`会根据用户滑动的距离显示header的长度，而`SliverAppbar`的floating效果是只要用户反向滑动了，header会立刻显示出全部长度并覆盖列表。

比如一个纵向向下滑动的列表，header为200的高度，用户滑动到中间，header消失了，然后用户往回滑动了50的距离，此时如果header是`SliverPersistentHeader`，则会显示出header，但只显示50的高度，同时下面的list也会向下滑动50，这样用户依然能看到之前的条目，直到用户滑动了200以上距离，header完全显示出来才会继续向上滑动；而如果header是`SliverAppBar`，则header会立刻完全显示除200的高度，下方的列表不会动，也就意味着上面的一些item会被floating的header覆盖掉。

`SliverAppBar`的这种效果很多时候是我们不想要的，因为用户很可能只是想往回滑一点看看上面的item，这样直接覆盖了一个header，用户为了看到相同的item反而要滑动更多，Flutter官方提供了一个解决方案，就是将SliverAppBar嵌套一个Widget：`SliverOverlapAbsorber`，然后在 body 中往 `CustomScrollView` 的 sliver列表的最前面插入了一个 `SliverOverlapInjector`。

这样做的结果就是，当用户往回滑动时，`SliverOverlapAbsorber`可以获取到header的高度，然后`SliverOverlapInjector`会帮我们操作scrollable滑动对应的距离。对应上面的栗子，当用户往回滑动了50，高度为200的header立刻显示，下面的list也会再向下滑动200距离，这样用户看到的就还是之前的item。

示例代码如下：

```dart
class SnapAppBar extends StatelessWidget {
  const SnapAppBar({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: NestedScrollView(
        headerSliverBuilder: (BuildContext context, bool innerBoxIsScrolled) {
          return <Widget>[
            SliverOverlapAbsorber(
              handle: NestedScrollView.sliverOverlapAbsorberHandleFor(context),
              sliver: SliverAppBar(
                floating: true,
                snap: true,
                expandedHeight: 200,
                flexibleSpace: FlexibleSpaceBar(
                  background: Image.asset(
                    "./imgs/sea.png",
                    fit: BoxFit.cover,
                  ),
                ),
                forceElevated: innerBoxIsScrolled,
              ),
            ),
          ];
        },
        body: Builder(builder: (BuildContext context) {
          return CustomScrollView(
            slivers: <Widget>[
              SliverOverlapInjector(
                handle: NestedScrollView.sliverOverlapAbsorberHandleFor(context),
              ),
              buildSliverList(100)
            ],
          );
        }),
      ),
    );
  }
}
```