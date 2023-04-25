## FittedBox

根据Flutter 的布局协议，父组件会将自身的最大显示空间作为约束传递给子组件，子组件应该遵守父组件的约束，如果子组件原始大小超过了父组件的约束区域，则需要进行一些缩小、裁剪或其他处理，而不同的组件的处理方式是特定的。

但是总有一些场景，我们希望自己根据父组件的约束条件来对子组件进行缩放、裁剪等处理，这时候就可以用`FittedBox`：

```dart
const FittedBox({
  Key? key,
  this.fit = BoxFit.contain, // 适配方式，和Image的fit属性相同，相当于Android中的scaleMode
  this.alignment = Alignment.center, //对齐方式
  this.clipBehavior = Clip.none, //是否剪裁
  Widget? child,
})
```

### FittedBox适配原理

`FittedBox`的使用方法就是嵌套在子组件外来对子组件进行适配，适配的原理是：

1. `FittedBox`忽略父组件传递过来的约束，允许子组件无限大
2. `FittedBox`对子组件布局结束后就可以获得子组件真实的大小
3. 有了父组件的约束和子组件的大小，再通过指定的适配方式适配子组件

