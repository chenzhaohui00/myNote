## `Stream`的创建

#### 1. Stream类的构造

比如常用的一个命名构造`periodic`：

```dart
final _stream = Stream.periodic(const Duration(seconds: 1), (i) => i);
```

但使用`periodic`构造出来的steam不能进行细致的控制，比如说自己控制什么时候发具体的什么时间，或者是关闭这个steam。

#### 2. 通过StreamContoller

`StreamController`可以自己控制往流里面添加event，也需要自己在需要的时候手动关闭steam。

`StreamContoller`内部有两个属性，`stream`和`sink`。前者就是用来`listen`或者在`StreamBuilder`中使用的，后者用来具体控制这个`stream`。

#### 3. 返回Stream的Api

比如读文件时使用`File.openRead()`就会返回一个Stream

#### 4. 使用异步生成器(async*)

我们知道方法声明中使用`async`会返回`Future`，而使用`async*`则会返回一个`Stream`，如下：

```dart
Stream<int> timedCounter(Duration interval, [int? maxCount]) async* {
  int i = 0;
  while (true) {
    await Future.delayed(interval);
    yield i++;
    if (i == maxCount) break;
  }
}
```

就是`Generator`的写法，通过`yield`来发出一个事件。



## `StreamController`

### `StreamController.sink`

前面说了，`StreamContoller`内部有两个属性，`stream`和`sink`。前者就不用说了，用来`listen`或者在`StreamBuilder`中使用就好了。重点记下这个`sink`，他类似于水槽的概念，主要可以通过几个方法控制`Stream`：

- `add` ： 往水槽里添加一个事件
- `addError`：往水槽里添加一个`error`
- `close`：关闭这个`stream`



## Stream的一些常用方法

类似于rxJava的操作符，一般用来生成另一个`Stream`

- `where`：用于过滤流返回的事件
- `map`：用来做数据转换
- `disctinct`：用来对重复数据做去重



## `Stream`的一些特性

- Stream是冷流，意味着需要有人`listen`，他才会发事件出来。
- 一个`Stream`只能被一个人`listen`，否则就会报错。
- 一个`Stream`要想被多个人`listen`，就需要将这个stream转换为一个广播，具体就是在创建`Stream`的时候，通过`StreamController.broadcast()`构造来创建一个广播`Stream`。



## Sample

```dart
import 'dart:async';

import 'package:flutter/material.dart';

class StreamBuilderSample extends StatefulWidget {
  const StreamBuilderSample({super.key});

  @override
  State<StreamBuilderSample> createState() {
    return StreamBuilderSampleState();
  }
}

class StreamBuilderSampleState
    extends State<StreamBuilderSample> {
  final _stream = Stream.periodic(const Duration(seconds: 1), (i) => i);
  final _controller = StreamController.broadcast();

  @override
  void initState() {
    // _stream.listen((event) {
    //   print(event);
    // });
    _controller.stream.listen((event) {
      debugPrint("receive: $event");
    }, onError:  (error, stackTrace){
      debugPrint("receive error: $error");
    });
    super.initState();
  }

  @override
  void dispose() {
    _controller.sink.close();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text(
          "FutureBuilderAndStreamBuilder",
          style: TextStyle(fontSize: 25),
        ),
      ),
      body: Center(
        child: DefaultTextStyle(
          style: const TextStyle(fontSize: 32, color: Colors.black),
          child: Column(
            children: [
              TextButton(
                  onPressed: () {
                    _controller.sink.add(10);
                  },
                  child: const Text("10")),
              TextButton(
                  onPressed: () {
                    _controller.sink.add("hei");
                  },
                  child: const Text("hei")),
              TextButton(
                  onPressed: () {
                    _controller.sink.addError("oops");
                  },
                  child: const Text("error")),
              TextButton(
                  onPressed: () {
                    _controller.sink.close();
                  },
                  child: const Text("close")),
              StreamBuilder(
                stream: _controller.stream
                    .where((event) => event is int)
                    .map((event) => event * 2)
                    .distinct(),
                builder: (BuildContext context, AsyncSnapshot snapshot) {
                  debugPrint("build");
                  switch (snapshot.connectionState) {
                    case ConnectionState.none:
                      return const Text("none: 无数据流");
                    case ConnectionState.waiting:
                      return const Text("waiting：等待数据流");
                    case ConnectionState.active:
                      if (snapshot.hasError) {
                        return Text("active，error：${snapshot.error}");
                      } else {
                        return Text("active，data：${snapshot.data}");
                      }
                    case ConnectionState.done:
                      return const Text("done：数据流已关闭");
                  }
                },
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```