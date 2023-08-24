## `Flutter`事件处理整体流程

Flutter的整个事件处理，主要围绕一个 `HitTestResult` 列表进行，整个流程如下：

1. 命中测试：找到哪些组件能接受到事件，手指按下时，触发`PointerDownEvent`事件，从根组件对应的 render object 开始，按照深度优先便利整个渲染树，对全部 render object 进行**命中测试**，命中测试通过的会被加入`HitTestResult`列表。
2. 事件分发：对能接受到事件的组件调用事件处理方法，就是调用`HitTestResult`列表中所有组件的`handleEvent`方法。
3. 事件清理：一个手势的结束是一个`PointerUpEvent`，此时清除`HitTestResult`列表

事件处理流程的伪代码在书中有，这里不列出来了。

## 命中测试

整个命中测试通常涉及到三个方法，

- `hitTest()`：父 render object 调用子 render object使用的方法，返回值布尔值，一般代表了是否通过了命中测试，也有例外。

- `hitTestChildren()`：有多个子child的 render object 通常用这个方法调用所有child的hitTest方法。

- `hitTestSelf()`：有子child的 redner object 用来返回自己是否可以通过命中测试的方法，但是最终还是要添加到`HitTestResult`列表中才算通过。

> 注意：命中测试通过的唯一标准就是被加入到了`HitTestResult`列表

### 1. 开始命中测试

调用根节点的`RenderObject`的`hitTest`方法:

```dart
@override
void hitTest(HitTestResult result, Offset position) {
  //从根节点开始进行命中测试
  renderView.hitTest(result, position: position); 
  // 会调用 GestureBinding 中的 hitTest()方法，我们将在下一节中介绍。
  super.hitTest(result, position); 
}
```

这里的`rederView`就是根节点的 `RenderObject`

### 2. 在渲染树中的命中测试过程

renderView 的`hitTest`方法：

```dart
// 发起命中测试，position 为事件触发的坐标（如果有的话）。
bool hitTest(HitTestResult result, { Offset position }) {
  if (child != null)
    child.hitTest(result, position: position); //递归对子树进行命中测试
  //根节点会始终被添加到HitTestResult列表中
  result.add(HitTestEntry(this)); 
  return true;
}
```

根节点会调用自己 child 的`hitTest`方法，有多个孩子的 RenderObject 会调用`hitTestChildren()`

```dart
bool hitTest(HitTestResult result, { @required Offset position }) {
  ...  
  if (_size.contains(position)) { // 判断事件的触发位置是否位于组件范围内
    if (hitTestChildren(result, position: position) || hitTestSelf(position)) {
      result.add(BoxHitTestEntry(this, position));
      return true;
    }
  }
  return false;
}
```

我们看一个典型的 `hitTestChildren`的实现：

```dart
// 子类的 hitTestChildren() 中会直接调用此方法
bool defaultHitTestChildren(BoxHitTestResult result, { required Offset position }) {
   // 遍历所有子组件(子节点从后向前遍历)
  ChildType? child = lastChild;
  while (child != null) {
    final ParentDataType childParentData = child.parentData! as ParentDataType;
    // isHit 为当前子节点调用hitTest() 的返回值
    final bool isHit = result.addWithPaintOffset(
      offset: childParentData.offset,
      position: position,
      //调用子组件的 hitTest方法，
      hitTest: (BoxHitTestResult result, Offset? transformed) {
        return child!.hitTest(result, position: transformed!);
      },
    );
    // 一旦有一个子节点的 hitTest() 方法返回 true，则终止遍历，直接返回true
    if (isHit) return true;
    child = childParentData.previousSibling;
  }
  return false;
}

  bool addWithPaintOffset({
    required Offset? offset,
    required Offset position,
    required BoxHitTest hitTest,
  }) {
    ...// 省略无关代码
    final bool isHit = hitTest(this, transformedPosition);
    return isHit; // 返回 hitTest 的执行结果
  }
```

主要逻辑是遍历调用子组件的 hitTest() 方法，同时提供了一种中断机制：即遍历过程中只要有子节点的 hitTest() 返回了 true 时：

1. 会终止子节点遍历，这意味着该子节点前面的兄弟节点将没有机会通过命中测试。注意，兄弟节点的遍历倒序的。
2. 父节点也会通过命中测试。因为子节点 hitTest() 返回了 true 导父节点 hitTestChildren 也会返回 true，最终会导致 父节点的 hitTest 返回 true，父节点被添加到 HitTestResult 中。

这个中断机制的目的，就是保证大多数情况的，因为大多数情况下兄弟节点占用的布局空间是不重合的，因此当用户点击的坐标位置只会有一个节点，所以一旦找到它后（通过了命中测试，hitTest 返回true），就没有必要再判断其他兄弟节点了。

但是也有例外情况，比如在 Stack 布局中，兄弟组件的布局空间会重叠，如果我们想让位于底部的组件也能响应事件，就得有一种机制，能让我们确保：即使找到了一个节点，也不应该终止遍历，也就是说所有的子组件的 hitTest 方法都必须返回 false！

另外，为什么兄弟节点的遍历要倒序？如上所述，兄弟节点一般不会重叠，而一旦发生重叠的话，往往是后面的组件会在前面组件之上，点击时应该是后面的组件会响应事件，而前面被遮住的组件不能响应，所以命中测试应该优先对后面的节点进行测试，因为一旦通过测试，就不会再继续遍历了。

#### IgnorePointer和AbsorbPointer的原理

回到 hitTestChildren 上，如果不重写 hitTestChildren，则默认直接返回 false，这也就意味着后代节点将无法参与命中测试，相当于事件被拦截了，这也正是 IgnorePointer 和 AbsorbPointer 可以拦截事件下发的原理。

如果 hitTestSelf 返回 true，则无论子节点中是否有通过命中测试的节点，当前节点自身都会被添加到 HitTestResult 中。而 IgnorePointer 和 AbsorbPointer 的区别就是，前者的 hitTestSelf 返回了 false，而后者返回了 true。

## 事件分发 & 事件清理

事件分发就是遍历`HitTestResult`列表，并调用每一个render object的`handleEvent`方法。

事件清理就是在 up 事件以后，清空`HitTestResult`列表。

## HitTestBehavior

前面提的到手势监听组件`Listener`的参数中有一个`HitTestBehavior`属性，就是用来控制`Listener`是否能够通过命中测试，以及`hitTest`的返回值的。`HitTestBehavior`有以下几种取值：

```dart
//在命中测试过程中 Listener 组件如何表现。
enum HitTestBehavior {
  // 组件是否通过命中测试取决于子组件是否通过命中测试
  deferToChild,
  // 组件必然会通过命中测试，同时其 hitTest 返回值始终为 true
  opaque,
  // 组件必然会通过命中测试，但其 hitTest 返回值可能为 true 也可能为 false
  translucent,
}
```

查看 Listener 源码，发现它的渲染对象 RenderPointerListener 继承了 RenderProxyBoxWithHitTestBehavior 类：

```dart
abstract class RenderProxyBoxWithHitTestBehavior extends RenderProxyBox {
  //[behavior] 的默认值为 [HitTestBehavior.deferToChild].
  RenderProxyBoxWithHitTestBehavior({
    this.behavior = HitTestBehavior.deferToChild,
    RenderBox? child,
  }) : super(child);

  HitTestBehavior behavior;

  @override
  bool hitTest(BoxHitTestResult result, { required Offset position }) {
    bool hitTarget = false;
    if (size.contains(position)) {
      hitTarget = hitTestChildren(result, position: position) || hitTestSelf(position);
      if (hitTarget || behavior == HitTestBehavior.translucent) //1
        result.add(BoxHitTestEntry(this, position)); // 通过命中测试
    }
    return hitTarget;
  }

  @override
  bool hitTestSelf(Offset position) => behavior == HitTestBehavior.opaque; //2

}
```

具体到代码层面，`HitTestBehavior`的三个值的作用是：

1. behavior 为 deferToChild 时，hitTestSelf 返回 false，当前组件是否能通过命中测试完全取决于 hitTestChildren 的返回值。也就是说只要有一个子节点通过命中测试，则当前组件便会通过命中测试。
2. behavior 为 opaque 时，hitTestSelf 返回 true，hitTarget 值始终为 true，当前组件通过命中测试。
3. behavior 为 translucent 时，hitTestSelf 返回 false，hitTarget 值此时取决于 hitTestChildren 的返回值，但是无论 hitTarget 值是什么，当前节点都会被添加到 HitTestResult 中。

## 兄弟节点拦截事件问题以及HitTestBlocker

前面提到了，默认我们无法让一个`Stack`中的多个重合的子 widget 可以处理同一个事件，因为靠后的`widget`的`hitTest`会返回 true，导致`Stack`的`hitTestChildren`跳过前面的 widget 的命中测试。

这一节提供了一个水印的栗子，展示了这个问题。另外还提供了一个自定义的组件 `HitTestBlocker`，可以通过参数，详细控制命中测试的一些过程，以及 hitTest 的返回值，从而解决了兄弟节点拦截事件的问题。具体可以看书中的栗子，这里不详述了。