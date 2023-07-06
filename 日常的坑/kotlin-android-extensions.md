kotlin-android-extensions的一个问题

使用kotlin-android-extensions可以起到类似于viewBinding的问题，而且不需要生成额外的ViewBinding或DataBinding的类。用起来是蛮舒服的，但是有一个问题，他的原理就是`getView().findViewById()`，所以有几种情况不可用，包括getView返回空，或者刚inflate的布局还没有添加到控件树上，此时就使用这个布局里的view。

这次遇到的情况就是这样，我们使用到一个PoppupWindow，

1. 先inflate出来一个布局
2. 对inflate的布局中的一些view进行初始化操作
3. 调用PoppupWindow的setContentView把设置进去。