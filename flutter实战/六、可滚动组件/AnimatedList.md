## 简介

AnimatedList 可以在列表中插入或删除节点时执行一个动画。AnimatedList 是一个 StatefulWidget，它对应的 State 类型为 AnimatedListState，添加和删除元素的方法位于 AnimatedListState 中：

```dart
void insertItem(int index, { Duration duration = _kDuration });

void removeItem(int index, AnimatedListRemovedItemBuilder builder, { Duration duration = _kDuration }) ;
```

## Sample

```dart
class AnimatedListRoute extends StatefulWidget {
  const AnimatedListRoute({super.key});

  @override
  State<StatefulWidget> createState() => AnimatedListRouteState();
}

class AnimatedListRouteState extends State<AnimatedListRoute> {
  GlobalKey<AnimatedListState> listKey = GlobalKey();
  final _data = [];

  @override
  void initState() {
    for (int i = 0; i < 5; i++) {
      _data.add('$i');
    }
    super.initState();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('AnimatedListRoute'),
      ),
      body: Stack(
        children: [
          AnimatedList(
            key: listKey,
            initialItemCount: _data.length,
            itemBuilder: (context, index, animation) {
              return FadeTransition(
                opacity: animation,
                child: buildItem(context, index),
              );
            },
          ),
          Positioned(
            bottom: 30,
            left: 0,
            right: 0,
            child: FloatingActionButton(
                child: const Icon(Icons.add, color: Colors.white),
                onPressed: () {
                  //添加子item
                  //1.添加数据
                  var lastNum = int.parse(_data[_data.length - 1]);
                  _data.add((++lastNum).toString());
                  //2.insertItem
                  listKey.currentState!.insertItem(_data.length - 1);
                }),
          )
        ],
      ),
    );
  }

  Widget buildItem(BuildContext context, int index) {
    String char = _data[index];
    return ListTile(
      key: ValueKey(char),
      title: Text(char),
      trailing: IconButton(
        icon: const Icon(Icons.delete),
        onPressed: () => onDelete(context, index),
      ),
    );
  }

  void onDelete(BuildContext context, int index) {
    listKey.currentState!.removeItem(index, (context, animation) {
      var item = buildItem(context, index);
      _data.removeAt(index);
      return FadeTransition(
        opacity: CurvedAnimation(
            parent: animation,
            //让透明度变化的更快一些
            curve: const Interval(0.5, 1.0)),
        // 不断缩小列表项的高度
        child: SizeTransition(
          sizeFactor: animation,
          axisAlignment: 0.0,
          child: item,
        ),
      );
    });
  }
}
```