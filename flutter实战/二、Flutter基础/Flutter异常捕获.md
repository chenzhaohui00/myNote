## Dart单线程模型

Dart和JS一样，都是单线程模型。它的运行机制大致如下：

![img](../../pictures/2-21.eb7484c9.png)

可以看到，有两个队列，一个是“微任务队列” **microtask queue**，另一个叫做“事件队列” **event queue**。微任务队列优先级高于事件队列。在Dart中，所有的外部事件任务都在事件队列中，如IO、计时器、点击、以及绘制事件等，而微任务通常来源于Dart内部，并且微任务非常少可以通过`Future.microtask(…)`方法向微任务队列插入一个任务。

Dart线程运行过程，如上图中所示，入口函数 main() 执行完后，消息循环机制便启动了。首先会按照先进先出的顺序逐个执行微任务队列中的任务，事件任务执行完毕后程序便会退出，但是，在事件任务执行的过程中也可以插入新的微任务和事件任务，在这种情况下，整个线程的执行过程便是一直在循环，不会退出，而Flutter中，主线程的执行过程正是如此，永不终止。

在事件循环中，当某个任务发生异常并没有被捕获时，程序并不会退出，而直接导致的结果是**当前任务**的后续代码就不会被执行了，也就是说一个任务中的异常是不会影响其他任务执行的。

## Flutter异常捕获

#### 1. Flutter框架已捕获的异常

Flutter 框架为我们在很多关键的方法进行了异常捕获，比如build Widget时发生异常时，会自动弹出一个错误页面，相关源码如下：

```dart
@override
void performRebuild() {
 ...
  try {
    //执行build方法  
    built = build();
  } catch (e, stack) {
    // 有异常时则弹出错误提示  
    built = ErrorWidget.builder(_debugReportException('building $this', e, stack));
  } 
  ...
}      
```

可以看到，catch到异常后返回了一个`ErrorWidget`，我们进入`_debugReportException()`方法看看：

```dart
FlutterErrorDetails _debugReportException(
  String context,
  dynamic exception,
  StackTrace stack, {
  InformationCollector informationCollector
}) {
  //构建错误详情对象  
  final FlutterErrorDetails details = FlutterErrorDetails(
    exception: exception,
    stack: stack,
    library: 'widgets library',
    context: context,
    informationCollector: informationCollector,
  );
  //报告错误 
  FlutterError.reportError(details);
  return details;
}
```

#### 2. 其他异常

在Dart中，异常分两类：同步异常和异步异常，同步异常可以通过`try/catch`捕获，而异步异常则比较麻烦，如下面的代码是捕获不了`Future`的异常的：

```dart
try{
    Future.delayed(Duration(seconds: 1)).then((e) => Future.error("xxx"));
}catch (e){
    print(e)
}
```

但是在 Dart 中有一个方法`runZoned(...)`，可以捕获到所有指定方法内所有的异常，可以做到类似Java中`defaultUncaughtException`的作用。通过给这个方法指定一个Zone，可以捕获、拦截或修改一些代码行为，如Zone中可以捕获日志输出、Timer创建、微任务调度的行为，同时Zone也可以捕获所有未处理的异常。下面我们看看`runZoned(...)`方法定义：

```dart
R runZoned<R>(R body(), {
    Map zoneValues, 
    ZoneSpecification zoneSpecification,
}) 
```

- `zoneValues`: Zone 的私有数据，可以通过实例`zone[key]`获取，可以理解为每个“沙箱”的私有数据。
- `zoneSpecification`：Zone的一些配置，可以自定义一些代码行为，比如拦截日志输出和错误等，举个例子：

```dart
runZoned(
  () => runApp(MyApp()),
  zoneSpecification: ZoneSpecification(
    // 拦截print 蜀西湖
    print: (Zone self, ZoneDelegate parent, Zone zone, String line) {
      parent.print(zone, "Interceptor: $line");
    },
    // 拦截未处理的异步错误
    handleUncaughtError: (Zone self, ZoneDelegate parent, Zone zone,
                          Object error, StackTrace stackTrace) {
      parent.print(zone, '${error.toString()} $stackTrace');
    },
  ),
);
```

最终的异常捕获和上报代码大致如下

```dart
void collectLog(String line){
    ... //收集日志
}
void reportErrorAndLog(FlutterErrorDetails details){
    ... //上报错误和日志逻辑
}

FlutterErrorDetails makeDetails(Object obj, StackTrace stack){
    ...// 构建错误信息
}

void main() {
  var onError = FlutterError.onError; //先将 onerror 保存起来
  FlutterError.onError = (FlutterErrorDetails details) {
    onError?.call(details); //调用默认的onError
    reportErrorAndLog(details); //上报
  };
  runZoned(
  () => runApp(MyApp()),
  zoneSpecification: ZoneSpecification(
    // 拦截print
    print: (Zone self, ZoneDelegate parent, Zone zone, String line) {
      collectLog(line);
      parent.print(zone, "Interceptor: $line");
    },
    // 拦截未处理的异步错误
    handleUncaughtError: (Zone self, ZoneDelegate parent, Zone zone,
                          Object error, StackTrace stackTrace) {
      reportErrorAndLog(details);
      parent.print(zone, '${error.toString()} $stackTrace');
    },
  ),
 );
}
```

## 总结

- Dart是以单线程配合消息循环机制来运行的，微任务队列的执行优先级高于事件队列。
- 在事件循环中，当某个任务发生异常并没有被捕获时，程序并不会退出，而直接导致的结果是**当前任务**的后续代码就不会被执行了
- Flutter框架对一些异常已经进行了捕获，并且提供了一个处理入口给我们，就是`Flutter.onError`，可以做日志收集、上传等操作。
- 对于其他未捕获异常，通过`runZoned()`监听和处理。