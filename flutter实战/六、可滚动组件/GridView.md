## SliverGridDelegate

网格布局`GridView`的构造函数跟`ListView`基本完全一致，唯一核心的一个区别的属性是`gridDelegate`，GridView构造函数如下：

```dart
GridView({
    Key? key,
    Axis scrollDirection = Axis.vertical,
    bool reverse = false,
    ScrollController? controller,
    bool? primary,
    ScrollPhysics? physics,
    bool shrinkWrap = false,
    EdgeInsetsGeometry? padding,
    required this.gridDelegate,  //就这一个不同的
    bool addAutomaticKeepAlives = true,
    bool addRepaintBoundaries = true,
    double? cacheExtent, 
    List<Widget> children = const <Widget>[],
    ...
  })
```

`gridDelegate`参数的类型是`SliverGridDelegate`，他用来确定如何`Layout`子组件，网格布局最重要的就是确认子组件的宽高，然后确认横轴方向每行可以放几个子组件，对此Flutter中提供了两个`SliverGridDelegate`的子类：`SliverGridDelegateWithFixedCrossAxisCount`和`SliverGridDelegateWithMaxCrossAxisExtent`。

## SliverGridDelegateWithFixedCrossAxisCount

见名知义，就是固定横轴数目，构造函数：

```dart
SliverGridDelegateWithFixedCrossAxisCount({
  @required double crossAxisCount,  //横轴子组件数量
  double mainAxisSpacing = 0.0, //主轴间距
  double crossAxisSpacing = 0.0, //横轴间距
  double childAspectRatio = 1.0, //子元素横轴的长度和主轴长度的比例
})
```

### 栗子

```dart
GridView(
  gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
      crossAxisCount: 3, //横轴三个子widget
      childAspectRatio: 1.0 //宽高比为1时，子widget
  ),
  children:<Widget>[
    Icon(Icons.ac_unit),
    Icon(Icons.airport_shuttle),
    Icon(Icons.all_inclusive),
    Icon(Icons.beach_access),
    Icon(Icons.cake),
    Icon(Icons.free_breakfast)
  ]
);
```

### GridView.count构造

Flutter 提供了`count`构造，用来快捷地使用`SliverGridDelegateWithFixedCrossAxisCount`，上面的栗子等同于这么写：

```dart
GridView.count( 
  crossAxisCount: 3,
  childAspectRatio: 1.0,
  children: <Widget>[
    Icon(Icons.ac_unit),
    Icon(Icons.airport_shuttle),
    Icon(Icons.all_inclusive),
    Icon(Icons.beach_access),
    Icon(Icons.cake),
    Icon(Icons.free_breakfast),
  ],
);
```



## SliverGridDelegateWithMaxCrossAxisExtent

固定子元素的最大长度，构造函数：

```dart
SliverGridDelegateWithMaxCrossAxisExtent({
  double maxCrossAxisExtent, //子元素在横轴上的最大长度
  double mainAxisSpacing = 0.0,
  double crossAxisSpacing = 0.0,
  double childAspectRatio = 1.0,
})
```

确定了最大长度后，就可以得出这个长度的子组件可以在横轴上的数量。其他参数和`SliverGridDelegateWithFixedCrossAxisCount`相同。

### GridView.extent构造

```dart
GridView.extent(
   maxCrossAxisExtent: 120.0,
   childAspectRatio: 2.0,
   children: <Widget>[
     Icon(Icons.ac_unit),
     Icon(Icons.airport_shuttle),
     Icon(Icons.all_inclusive),
     Icon(Icons.beach_access),
     Icon(Icons.cake),
     Icon(Icons.free_breakfast),
   ],
 );
```



## GridView.builder构造

上面都是固定子组件`children`的构造方法，如果需要实现动态创建子组件的`GridView`则需要使用`GridView.builder`构造

```dart
GridView.builder(
  gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
      crossAxisCount: 4, childAspectRatio: 2),
  itemBuilder: (context, index) {
    return Container(
        decoration: BoxDecoration(
            border: Border.all()
        ), child: ListTile(title: Text('$index'))
    );
  },
);
```

### sample

```dart
GridView.builder(
  gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
      crossAxisCount: 4, childAspectRatio: 2),
  itemBuilder: (context, index) {
    return Container(
        decoration: BoxDecoration(
            border: Border.all()
        ), child: ListTile(title: Text('$index'))
    );
  },
);
```

