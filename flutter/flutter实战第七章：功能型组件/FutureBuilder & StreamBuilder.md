### 基本用法

dart 提供了`Future`和`Stream`给我们处理异步请求以及流式的异步消息，Flutter 则提供了`FutureBuilder`和`StreamBuilder`两个组件来帮我们接收`Future`和`Stream`并生成一个Widget。

以下是`FutrueBuilder`和`StreamBuilder`的构造，可以一起看：

```dart
FutureBuilder({
  this.future,
  this.initialData,
  required this.builder,
})
```

```dart
StreamBuilder({
  Stream<T> stream,
  this.initialData,
  required this.builder,
}) 
```

可以看到是很相似的，就是接收一个future或stream，还可以接受一些init的数据，然后根据一个builder方法，用对应的`future`和`stream`的信息来生成Widget。

builder的签名如下：

```dart
Function (BuildContext context, AsyncSnapshot snapshot) 
```

这个`snapshot`的就是`future`或`stream`的状态快照，其中的一些重要的属性如下：

```dart
final ConnectionState connectionState; //当前的异步状态
bool get hasData => data != null; //有结果数据
final T? data; //结果数据
bool get hasError => error != null; //有错误
final Object? error; //错误
```

所以就可以在build中根据这些状态和数据返回Widget了，sample如下：

```dart
import 'package:flutter/material.dart';

class FutureBuilderAndStreamBuilderRoute extends StatelessWidget {
  const FutureBuilderAndStreamBuilderRoute({super.key});

  Future<String> mockNetRequest() {
    return Future.delayed(const Duration(seconds: 3), () => '网络请求完成');
  }

  Stream<int> streamCounter() {
    return Stream.periodic(const Duration(seconds: 1), (i) => i);
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('FutureBuilderAndStreamBuilderRoute'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            FutureBuilder(
              future: mockNetRequest(),
              builder: (BuildContext context, AsyncSnapshot snapshot) {
                if (snapshot.connectionState == ConnectionState.done) {
                  if (snapshot.hasError) {
                    return Text(
                      'error:${snapshot.error}',
                      textScaleFactor: 2,
                    );
                  } else {
                    return Text(
                      'result: ${snapshot.data}',
                      textScaleFactor: 2,
                    );
                  }
                } else {
                  return const CircularProgressIndicator();
                }
              },
            ),
            StreamBuilder(stream: streamCounter(), builder: (ctx, snapshot) {
              switch (snapshot.connectionState) {
                case ConnectionState.none:
                  return const Text('none', textScaleFactor: 2,);
                case ConnectionState.waiting:
                  return const Text('waiting', textScaleFactor: 2,);
                case ConnectionState.active:
                  return Text('${snapshot.data}', textScaleFactor: 2);
                default:
                  return const Text('done', textScaleFactor: 2,);
              }
            })
          ],
        ),)
      ,
    );
  }
}
```

### initialData和Future.sync()、Future.value()的冲突

initialData会给一个初始化数据，比如组件的一些初始的数据，这个值会给到`snapshop`的`data`属性中，但是不会改变`connectionState`，这就意味着如果我们builder中先判断`connectionState`，会发现还是`waiting`，但这时候已经有`data`了。

所以可能更合适的写法是，不实用`connectionState`，直接判断 `hasData` 和 `hasError` ，如果两者都是false，就是waiting状态。

### 总结

总结一下，就是使用这两个组件，可以传入一个Future或Stream做业务处理，然后这个组件会在Future或Stream回调时调用我们的build方法，我们就在build方法中监听前者的状态，并返回不同的widget即可。