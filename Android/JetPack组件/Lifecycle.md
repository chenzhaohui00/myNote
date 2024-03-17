## 使用Lifecycle的意义
### 解耦，避免activity或fragment太臃肿

很多三方库或者View或者组件是需要根据activity或fragment的生命周期来进行初始化或销毁的，把这些逻辑抽出去到一个类中，避免activity和fragment的生命周期方法中的代码越臃肿和难以维护，也符合单一职责原则。

### 根据生命周期回调调整一些业务逻辑

1. 调整更新粒度：一个位置信息类，根据应用是否在前台，调整获取位置信息的粒度。前台时细粒度更新，后台时粗粒度更新。
2. 开始和停止缓存：视频或音频，观察到onCreate就开始缓存，但是onResume的时候再开始播放，onDestory时停止缓存。
3. 开始和停止网络连接：后台停止网络请求。
4. 停止和开始动画：暂停时停止动画，继续时继续动画。



## 几个重要的类和接口
- Lifecycle 
    Lifecycle是一个持有组件生命周期状态（如Activity或Fragment）的信息的类，并允许其他对象观察此状态。
- Event 
    从框架和Lifecycle类派发的生命周期事件。这些事件映射到活动和片段中的回调事件。
- State 
    由Lifecycle对象跟踪的组件的当前状态。
- LifecycleOwner （重要）Lifecycle持有者 
    实现该接口的类持有生命周期(Lifecycle对象)，该接口的生命周期(Lifecycle对象)的改变会被其注册的观察者LifecycleObserver观察到并触发其对应的事件。
- LifecycleObserver（重要）Lifecycle观察者 
    实现该接口的类，通过注解的方式，可以通过被LifecycleOwner类的addObserver(LifecycleObserver o)方法注册,被注册后，LifecycleObserver便可以观察到LifecycleOwner的生命周期事件。



## State流转

![img](./lifecycle-states.svg)



## 具体使用

1. 让需要观察组件生命周期的类实现LifecycleObserver或DefaultLifecycleObserver。
    - 如果实现DefultLifecyceObserver接口，需要重写里面生命周期方法；
    - 如果直接实现LifecycleObserver接口，就需要通过注解的方式来接收生命周期的变化。 
    
    

实例代码：
```
public class MainPresenter implements IPresenter {
    private static final String TAG = "MainPresenter";
    public MainPresenter(Context context){

    }

    @Override
    public void onCreate(LifecycleOwner owner) {
        Log.d(TAG, "onCreate: ");
    }

    @Override
    public void onStart(LifecycleOwner owner) {
        Log.d(TAG, "onStart: ");
    }

    @Override
    public void onResume(LifecycleOwner owner) {
        Log.d(TAG, "onResume: ");
    }

    @Override
    public void onPause(LifecycleOwner owner) {
        Log.d(TAG, "onPause: ");
    }

    @Override
    public void onStop(LifecycleOwner owner) {
        Log.d(TAG, "onStop: ");
    }

    @Override
    public void onDestroy(LifecycleOwner owner) {
        Log.d(TAG, "onDestroy: ");
    }
}
```
```
public interface IPresenter extends DefaultLifecycleObserver{

}
```
2. 在拥有生命周期的组件(如activity和fragment)中，注册观察者。方法就是在已经实现了LifecycleOwner的类中调用```getLifecycle().addObserver(observer);```  

实例代码：
```
public class MainActivity extends AppCompatActivity {
    private static final String TAG = "MainActivity";
    private IPresenter mPresenter;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        Log.d(TAG, "onCreate: ");
        setContentView(R.layout.activity_main);
        mPresenter = new MainPresenter(this);
        getLifecycle().addObserver(mPresenter);
    }
}
```
这里要注意的是：Support Library 26.1.0 及其以后的版本，Activity 和Fragment 已经实现了LifecycleOwner 接口，所以，我们可以直接在Activity 和Fragment中使用getLifecycle()方法来获取lifecycle对象，来添加观察者监听。



## 实现原理

其实就是观察者模式，LifecycleOwner会在生命周期对应的方法中，通知已经注册过的LifecycleObserver。  
具体的代码分析，可以随便去网上搜一下就有了。比如：https://blog.csdn.net/zhuzp_blog/article/details/78871374