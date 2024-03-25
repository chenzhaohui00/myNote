## MVP

#### Presenter

1. 要符合SRP：不要所有东西都丢到这里， Presenter 是用来处理业务逻辑的，与此无关的东西都不应该放到这里，UI相关的事情都交给 View 层来处理，网络请求的细节应该放到 Model 或者 Repository中。另外，与Presetenter直接处理的业务没有直接关联的功能类也不应该放到这里。
2. 更改状态而不是调用View层的某个行为：基于数据驱动UI的原则，Presenter 调用 View 层的方法不应该是执行某一个动作，比如`showLoading()`、`hideLoading()`这种。而应该只是告诉View层自己的状态，比如给一个表示isLoading的可观察的数据，或者比如回调`isLoadingChanged(isLoading:  = false)`也好。
3. Presenter 尽量与任何Android解耦。不要调用任何Android的类，包括Log这种，如果和Android类解耦的很好，会很容易做测试。不需要mock那些没有意义的Android的类。另外如果有需要，这种Presenter类甚至可以复用到Android以外的其他业务平台。
4. 生命周期处理：Presenter 处理声明周期View的生命周期的问题，之前我用过的方法是BasePresenter定义一个onDestroy方法，在BaseView的onDestroy的时候调用。还有一种是直接去绑定 LifeCycleOwner，然后在BasePresenter里监听到 LifeCycleOwner的 onDestroy()方法，就直接断开所有未执行完的任务。



## MVVM