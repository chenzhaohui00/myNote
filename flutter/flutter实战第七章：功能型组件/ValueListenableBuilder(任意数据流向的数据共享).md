`InheritedWidget`可以给子组件共享数据和对象，也就是说他可以做到数据在组件树中从上到下流动。但是如果是从下到上的数据流，或是同级Widget之间的数据共享就无能为力了。此时就需要使用到`ValueListenableBuilder`。

`ValueListenableBuilder`的构造如下：

```dart
const ValueListenableBuilder({
  Key? key,
  required this.valueListenable, // 数据源，类型为ValueListenable<T>
  required this.builder, // builder
  this.child,
}
```

它接收一个数据源和一个创建子widget的builder，当监听到数据源数据变动时，重新执行builder的构建构建子widget。另外这个`child`用来构建其子组件之中不需要根据数据变化而变动的widget。

对于父组件使用子孙组件的数据的通常的使用方式就是：子组件的数据用`ValueNotifier`(`ValueListenable`的子类)包裹，父组件使用`ValueListenableBuilder`观察此数据的变化并据此build组件，sample如下：

```dart
class ValueListenableRoute extends StatefulWidget {
  const ValueListenableRoute({Key? key}) : super(key: key);

  @override
  State<ValueListenableRoute> createState() => _ValueListenableState();
}

class _ValueListenableState extends State<ValueListenableRoute> {
  // 定义一个ValueNotifier，当数字变化时会通知 ValueListenableBuilder
  final ValueNotifier<int> _counter = ValueNotifier<int>(0);
  static const double textScaleFactor = 1.5;

  @override
  Widget build(BuildContext context) {
    // 添加 + 按钮不会触发整个 ValueListenableRoute 组件的 build
    print('build');
    return Scaffold(
      appBar: AppBar(title: Text('ValueListenableBuilder 测试')),
      body: Center(
        child: ValueListenableBuilder<int>(
          builder: (BuildContext context, int value, Widget? child) {
            // builder 方法只会在 _counter 变化时被调用
            return Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: <Widget>[
                child!,
                Text('$value 次',textScaleFactor: textScaleFactor),
              ],
            );
          },
          valueListenable: _counter,
          // 当子组件不依赖变化的数据，且子组件收件开销比较大时，指定 child 属性来缓存子组件非常有用
          child: const Text('点击了 ', textScaleFactor: textScaleFactor),
        ),
      ),
      floatingActionButton: FloatingActionButton(
        child: const Icon(Icons.add),
        // 点击后值 +1，触发 ValueListenableBuilder 重新构建
        onPressed: () => _counter.value += 1,
      ),
    );
  }
}
```