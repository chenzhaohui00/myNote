## 简述

本质上就是一个可以监听LifeCycleOwner的数据包裹类，通过observe方法添加LifeCycleOwner的观察者，数据发生变化时，会通知生命周期处于活跃状态的观察者。

根据上述三点特性就有了三个优点：

- 作为用来观察的数据包裹类 -> 符合数据驱动UI方式。
- 数据变化主动通知观察者 -> 简化View层观察者的代码，不用在生命周期方法里做观察/停止观察的操作了。
- 只通知LifeCycle处于活跃状态的观察者 -> 解决了错误的回调导致的内存泄漏、崩溃。

另外，LiveData一般都放到ViewModel中，而后者的生命周期比View层的activtiy和fragment长，因此也利于配置更改等情况下的数据恢复。activity和fragment重建后，不用重新获取数据，直接observe就会直接收到数据。



## LiveData的使用位置

### 1. 在界面层使用，不要在数据层使用

具体原因有二：

1. LiveData默认不支持异步处理，onChange回调、和map的处理都是在主线程中进行的。这就意味着在数据层很多时候需要在子线程中做数据的转换，都没法在子线程处理了。
2. LiveData类默认提供的数据流组合的功能很有限，没有rxJava/rxKotlin和kotlin flow那么多。

### 2. 具体到界面层，要在ViewModle中使用，不要在Actvitiy/Fragment中使用



## 转换LiveData

通过Transformations类提供的map、switchMap方法，可以把LiveData转换为另一个LiveData，这两者的区别是方法传入的转换方法参数，map的转换方法返回的是转换后的值，switchMap的转换方法转换回的是包裹了转后换数据的LiveData对象。

所以他们的应用场景的区别就是，对于简单的数据转换使用map来做，对于一般的异步的数据请求、处理等，使用swichMap来做。



## 合并多个LiveData数据源

使用MediatorLiveData合并多个LiveData数据源，任意源发生变化都会触发MediatorLiveData回调观察者。



## 扩展LiveData

继承LiveData以后：

1. 重写onActive，当处于激活状态的observer个数从0到1时会被调用，写有观察者以后的逻辑，一般就是获取某个数据并且观察数据变化。
2. 重写onInActive，当处于激活状态的observer个数从1变为0时被调用，写没有观察者以后的逻辑，一般就是取消对数据的观察。

Sample代码如下：

```java
public class MyLiveData extends LiveData<Integer> {
    private static final String TAG = "MyLiveData";
    private static MyLiveData sData;
    private WeakReference<Context> mContextWeakReference;

    public static MyLiveData getInstance(Context context){
        if (sData == null){
            sData = new MyLiveData(context);
        }
        return sData;
    }

    private MyLiveData(Context context){
        mContextWeakReference = new WeakReference<>(context);
    }

    @Override
    protected void onActive() {
        super.onActive();
        registerReceiver();
    }

    @Override
    protected void onInactive() {
        super.onInactive();
        unregisterReceiver();
    }

    private void registerReceiver() {
        IntentFilter intentFilter = new IntentFilter();
        intentFilter.addAction(WifiManager.RSSI_CHANGED_ACTION);
        mContextWeakReference.get().registerReceiver(mReceiver, intentFilter);
    }

    private void unregisterReceiver() {
        mContextWeakReference.get().unregisterReceiver(mReceiver);
    }


    private BroadcastReceiver mReceiver = new BroadcastReceiver() {
        @Override
        public void onReceive(Context context, Intent intent) {
            String action = intent.getAction();
            Log.d(TAG, "action = " + action);
            if (WifiManager.RSSI_CHANGED_ACTION.equals(action)) {
                int wifiRssi = intent.getIntExtra(WifiManager.EXTRA_NEW_RSSI, -200);
                int wifiLevel = WifiManager.calculateSignalLevel(
                        wifiRssi, 4);
                sData.setValue(wifiLevel);
            }
        }
    };
}
```