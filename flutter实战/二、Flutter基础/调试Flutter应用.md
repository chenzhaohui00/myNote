## 日志与断点

#### 编程式断点

导包：`import 'dart:developer';`。然后就可以使用`debuger()`语句，该语句接收一个bool表达式，当真时程序中断，如下

```dart
void someFunction(double offset) {
  debugger(when: offset > 30.0);
  // ...
}
```

说实话我没看明白有啥用，就是程序中断了，然后也不知道能干啥。

#### 输出

- `print()`输出到控制台，可以使用`flutter logs`查看。
- 如果你一次输出太多，那么Android有时会丢弃一些日志行。为了避免这种情况，我们可以使用Flutter的`foundation`库中的`debugPrint()`，它封装了`print`，将一次输出的内容长度限制在一个级别（内容过多时会分批输出），避免被Android内核丢弃。
- 树中的一些类也具有`toStringDeep`实现，从该点返回整个子树的多行描述。
- 一些具有详细信息`toString`的类会实现一个`toStringShort`，它只返回对象的类型或其他非常简短的描述。

#### 断言

assert仅在调试模式会运行。

#### 断点

使用Android Studio正常的debug功能即可。

## 调试应用程序层

#### 1. dump Widget树

调用`debugDumpApp()`，只要app已经build过后，并且不在build方法内调用即可。结果会看到很多在你的应用源代码中没有出现的widget，因为它们是被框架中widget的`build()`函数插入的。如果我们编写自己的widget，则可以通过覆盖`debugFillProperties()` 来添加信息。

我试了一下，很简单的几层嵌套Widget，出来一大堆Widget，基本没法看的那种，控制台都输出不下了。而且在新路由中dump的，连首页的Widget树都打印出来了，而且排在前面。很难用的感觉。

#### 2. dump render树

调用`debugDumpRenderTree()`转储渲染树。 正如`debugDumpApp()`，除布局或绘制阶段外，我们可以随时调用此函数。作为一般规则，从 frame 回调 或事件处理器中调用它是最佳解决方案。

还是信息量过大的问题，不太好用。

#### 3. dump layer树

使用`debugDumpLayerTree()`，这个输出的信息就很少了，但是问题是和Widget很难对应起来。也不太好用。

#### 4. 帧开始和结束日志

找出相对于帧的开始/结束事件发生的位置，可以切换`debugPrintBeginFrameBanner`和`debugPrintEndFrameBanner`布尔值以将帧的开始和结束打印到控制台。

```dart
debugPrintBeginFrameBanner = true;
debugPrintEndFrameBanner = true;
```

to be continue..