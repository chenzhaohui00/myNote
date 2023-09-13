## 简介

`TabBarView`是对`PageView`的封装，相当于是有Tab的`PageView`，和`TabBar`配合使用。

所以使用`TabBarView`就是使用一个特定的`PageView`，其构造方法如下：

```dart
 TabBarView({
  Key? key,
  required this.children, // tab 页
  this.controller, // TabController
  this.physics,
  this.dragStartBehavior = DragStartBehavior.start,
}) 
```

由于`Tab`数一定是固定的，所以`TabBarView`的`children`数也一定是固定的，不能动态构建。

## TabBar

`TabBar`相当于一整条Tab条，子组件放一堆`Tab`组件，构造如下：

```dart
const TabBar({
  Key? key,
  required this.tabs, // 具体的 Tabs，需要我们创建
  this.controller, // TabController
  this.isScrollable = false, // 是否可以滑动
  this.padding,
  this.indicatorColor,// 指示器颜色，默认是高度为2的一条下划线
  this.automaticIndicatorColorAdjustment = true,
  this.indicatorWeight = 2.0,// 指示器高度
  this.indicatorPadding = EdgeInsets.zero, //指示器padding
  this.indicator, // 指示器
  this.indicatorSize, // 指示器长度，有两个可选值，一个tab的长度，一个是label长度
  this.labelColor, 
  this.labelStyle,
  this.labelPadding,
  this.unselectedLabelColor,
  this.unselectedLabelStyle,
  this.mouseCursor,
  this.onTap,
  ...
}) 
```

很多属性都是在配置 `indicator` 和 `label`。

`Tab`组件构造：

```dart
const Tab({
  Key? key,
  this.text, //文本
  this.icon, // 图标
  this.iconMargin = const EdgeInsets.only(bottom: 10.0),
  this.height,
  this.child, // 自定义 widget
})
```

## 联动

### 一、显示指定同一个`TabController`

要将`TabBarView`和`TabBar`联动有两种方式，一种是显式指定他们两个的tabController，指定成同一个。

示例如下：

```dart
class TabViewRoute1 extends StatefulWidget {
  @override
  _TabViewRoute1State createState() => _TabViewRoute1State();
}

class _TabViewRoute1State extends State<TabViewRoute1>
    with SingleTickerProviderStateMixin {
  late TabController _tabController;
  List tabs = ["新闻", "历史", "图片"];

  @override
  void initState() {
    super.initState();
    _tabController = TabController(length: tabs.length, vsync: this);
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text("App Name"),
        bottom: TabBar(
          controller: _tabController,
          tabs: tabs.map((e) => Tab(text: e)).toList(),
        ),
      ),
      body: TabBarView( //构建
        controller: _tabController,
        children: tabs.map((e) {
          return KeepAliveWrapper(
            child: Container(
              alignment: Alignment.center,
              child: Text(e, textScaleFactor: 5),
            ),
          );
        }).toList(),
      ),
    );
  }
  
  @override
  void dispose() {
    // 释放资源
    _tabController.dispose();
    super.dispose();
  }
}
```

> `TabController`在初始化时，需要传入一个 `SingleTickerProviderStateMixin`，通常就是让`State`类去 mixin 即可。

### 二、两者的父组件有同一个`DefaultTabController`

TabBar和TabBarView在没有指定contoller的时候，会在组件树中向上查找并使用最近的一个 `DefaultTabController`，所以只要我们让两者有一个共同的DefaultTabController的父级组件即可。

使用这种方式实现和上面的代码同样效果：

```dart
class TabViewRoute2 extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    List tabs = ["新闻", "历史", "图片"];
    return DefaultTabController(
      length: tabs.length,
      child: Scaffold(
        appBar: AppBar(
          title: Text("App Name"),
          bottom: TabBar(
            tabs: tabs.map((e) => Tab(text: e)).toList(),
          ),
        ),
        body: TabBarView( //构建
          children: tabs.map((e) {
            return KeepAliveWrapper(
              child: Container(
                alignment: Alignment.center,
                child: Text(e, textScaleFactor: 5),
              ),
            );
          }).toList(),
        ),
      ),
    );
  }
}
```