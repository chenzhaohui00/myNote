## CustomScrollView 简介

假设这样一个场景，一个列表存在多个滑动方向相同的按需加载的布局，希望他们能一起滚动，第一个列表滚动完毕直接让第二个列表继续滚动。仔细想也就是用一个`Scrollable`滑动，对应多个`Sliver`。

在 Android 上这个问题一般的处理方式就是`RecyclerView`套`RecyclerView`，或者说`RecyclerView`的一些item也是一个`RecyclerView`。这个方案不只是对应同方向的多个list，也可以处理滑动方向不同的列表。

在 Flutter 上，对于相同滑动方向的多个`Sliver`使用一个`Scrollable`滑动，需要使用`CustomScrollView`。`CustomScrollView`内有一个`slivers`属性，用来接收其所有的child sliver，而他本身作为一个`ScrollView`底层就是用的`Scrollable`的。这就做到了一个`Scrollable`对应多个`Sliver`。



## Sliver

前面介绍`Sliver`的时候就说了，`Sliver`是负责按需加载的，在需要加载子组件的时候，对子组件进行 build 和 layout 。 ListView、GridView、PageView、TabBarView等这些可滚动列表背后都是靠`Sliver`进行的按需加载。他们和对应的Sliver类的对应关系如下：

| Sliver名称                | 功能                               | 对应的可滚动组件                 |
| ------------------------- | ---------------------------------- | -------------------------------- |
| SliverList                | 列表                               | ListView                         |
| SliverFixedExtentList     | 高度固定的列表                     | ListView，指定`itemExtent`时     |
| SliverAnimatedList        | 添加/删除列表项可以执行动画        | AnimatedList                     |
| SliverGrid                | 网格                               | GridView                         |
| SliverPrototypeExtentList | 根据原型生成高度固定的列表         | ListView，指定`prototypeItem` 时 |
| SliverFillViewport        | 包含多个子组件，每个都可以填满屏幕 | PageView                         |

当需要直接使用`Sliver`的场景，就直接使用他们对应的Sliver即可。比如说想在`CustomScrollView`中使用一个`ListView`，只要使用`SliverList`就ok了。

除了和列表对应的 Sliver 之外还有一些用于对 Sliver 进行布局、装饰的组件，**它们的子组件必须是 Sliver**，我们列举几个常用的：

| Sliver名称                      | 对应 RenderBox      |
| ------------------------------- | ------------------- |
| SliverPadding                   | Padding             |
| SliverVisibility、SliverOpacity | Visibility、Opacity |
| SliverFadeTransition            | FadeTransition      |
| SliverLayoutBuilder             | LayoutBuilder       |

还有一些其他常用的 Sliver：

| Sliver名称             | 说明                                                   |
| ---------------------- | ------------------------------------------------------ |
| SliverAppBar           | 对应 AppBar，主要是为了在 CustomScrollView 中使用。    |
| SliverToBoxAdapter     | 一个适配器，可以将 RenderBox 适配为 Sliver，后面介绍。 |
| SliverPersistentHeader | 滑动到顶部时可以固定住，后面介绍。                     |

以上的这些Sliver组件，所有需要传入子组件且子组件必须是Sliver组件的，他们的接收Sliver的属性名都不叫child或者children，而是叫slivers或者sliver。比如CustomScrollView有slivers属性，SliverPadding有sliver属性。

> Sliver组件并没有统一的父类，但是Flutter默认提供的Sliver组件的命名都是以Sliver开头的。

这里的sample比较多，我们就不列举了，可以看书或者sample app。额外写两个比较值得特意写的Sliver组件。



## SliverToBoxAdapter

上面说了，一些组件只接受Sliver的子组件，虽然大多数列表组件都有对应的Sliver组件可以直接使用。但是更多的控件是没有Sliver组件的，而`SliverToBoxAdapter`就可以用来做子组件转换的。将需要转换的组件传到这个Adapter中即可。可以看出这是适配器模式的应用。

### SliverToBoxAdapter 转换后会发生滑动异常的case

`CustomScrollView`接收的孩子虽然可以使用`SliverToBoxAdapter`来转换，但是如果`CustomScrollView` 有孩子也是一个完整的可滚动组件时，则要求这些可滚动组件的滑动方向和`CustomScrollView`一致，否则滑动手势会被子组件拦截，且由于他们滑动方向不一致，外层的`Scrollable`无法正常滑动。

要解决这个问题，可以使用 `NestedScrollView`。



## SliverPersistentHeader

做吸顶效果的`ListView`的组件， `SliverPersistentHeader` 的定义：

```dart
const SliverPersistentHeader({
  Key? key,
  // 构造 header 组件的委托
  required SliverPersistentHeaderDelegate delegate,
  this.pinned = false, // header 滑动到可视区域顶部时是否固定在顶部
  this.floating = false, // header消失后，向主轴反方向滑动时header会显示并浮动在顶部
})
```

### SliverPersistentHeaderDelegate

`delegate` 是用于生成 header 的委托，类型为 SliverPersistentHeaderDelegate，它是一个抽象类，需要我们自己实现，定义如下：

```dart
abstract class SliverPersistentHeaderDelegate {

  // header 最大高度；pined为 true 时，当 header 刚刚固定到顶部时高度为最大高度。
  double get maxExtent;
  
  // header 的最小高度；pined为true时，当header固定到顶部，用户继续往上滑动时，header
  // 的高度会随着用户继续上滑从 maxExtent 逐渐减小到 minExtent
  double get minExtent;

  // 构建 header。
  // shrinkOffset取值范围[0,maxExtent],当header刚刚到达顶部时，shrinkOffset 值为0，
  // 如果用户继续向上滑动列表，shrinkOffset的值会随着用户滑动的偏移减小，直到减到0时。
  //
  // overlapsContent：一般不建议使用，在使用时一定要小心，后面会解释。
  Widget build(BuildContext context, double shrinkOffset, bool overlapsContent);
  
  // header 是否需要重新构建；通常当父级的 StatefulWidget 更新状态时会触发。
  // 一般来说只有当 Delegate 的配置发生变化时，应该返回false，比如新旧的 minExtent、maxExtent
  // 等其他配置不同时需要返回 true，其余情况返回 false 即可。
  bool shouldRebuild(covariant SliverPersistentHeaderDelegate oldDelegate);

  // 下面这几个属性是SliverPersistentHeader在SliverAppBar中时实现floating、snap 
  // 效果时会用到，平时开发过程很少使用到，读者可以先不用理会。
  TickerProvider? get vsync => null;
  FloatingHeaderSnapConfiguration? get snapConfiguration => null;
  OverScrollHeaderStretchConfiguration? get stretchConfiguration => null;
  PersistentHeaderShowOnScreenConfiguration? get showOnScreenConfiguration => null;

}
```
