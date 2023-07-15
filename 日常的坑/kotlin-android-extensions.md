kotlin-android-extensions的一个问题

使用kotlin-android-extensions可以起到类似于viewBinding的问题，可以直接import layout里的id，从而直接使用控件，而且不需要生成额外的ViewBinding或DataBinding的类。用起来是蛮舒服的，但是有一个问题，他的原理就是`getView().findViewById()`，所以有几种情况不可用，包括getView返回空，或者刚inflate的布局还没有添加到控件树上。

这次遇到的情况就是这样，我们使用到一个PoppupWindow，

1. 先inflate出来一个布局
2. 对inflate的布局中的一些view进行初始化操作
3. 调用PoppupWindow的setContentView把设置进去

然后在第2步的时候使用通过impor导入的view，编译期都没有问题，运行时会发现会空指针，因为通过`getView().findViewById()`没有找到这些view。