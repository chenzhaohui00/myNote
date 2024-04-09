## SystemServer - SystemUIService

```java
SystemServer.main // code snap 1.1
	SystemServer.run // code snap 1.2
		SystemServer.startOtherServices
			SystemServer.startSystemUi //就是通过LocalServiceManager获取一下信息，然后start一下
```

### code snap 1.1  Zygote启动

```java
/**
 * The main entry point from zygote.
 */
public static void main(String[] args) {
    new SystemServer().run();
}
```

### code snap 1.2  run方法，启动各种服务

```java
private void run() {
	/*
	 * 其他代码
	 */
	 
    // Start services.
    try {
        t.traceBegin("StartServices");
        startBootstrapServices(t); //包括PMS,WMS,AMS,ATMS等
        startCoreServices(t); //包括SystemConfigService,BatteryService等
        startOtherServices(t); //启动SystemUIService
    } catch (Throwable ex) {
        Slog.e("System", "******************************************");
        Slog.e("System", "************ Failure starting system services", ex);
        throw ex;
    } finally {
        t.traceEnd();
    }
}
```



## SystemUIService - SystemUIApplication

```java
SystemUIService.onCreate
	SystemUIApplication.startServicesIfNeeded //code snap 2.1
```

### code snap 2.1

```java
public void startServicesIfNeeded() {
    String[] names = SystemUIFactory.getInstance().getSystemUIServiceComponents(getResources());
    startServicesIfNeeded(/* metricsPrefix= */ "StartServices", names);
}
```

根据SystemUIFactory获取需要启动的组件名数组，然后一次启动，具体获取需要启动的组件名通过 config.xml 配置：

```java
public String[] getSystemUIServiceComponents(Resources resources) {
    return resources.getStringArray(R.array.config_systemUIServiceComponents);
}
```

### config.xml

读取的配置如下：

```xml
<!-- 这里是配置的SystemUIFactory的组件名，CarSystemUI就是配置了自己的Factory -->
<!-- SystemUIFactory component -->
<string name="config_systemUIFactoryComponent" translatable="false">com.android.systemui.SystemUIFactory</string>

<!-- 下面就是要启动的组建了 -->
<!-- SystemUI Services: The classes of the stuff to start. -->
<string-array name="config_systemUIServiceComponents" translatable="false">
    <item>com.android.systemui.util.NotificationChannels</item>
    <item>com.android.systemui.keyguard.KeyguardViewMediator</item>
    <item>com.android.systemui.recents.Recents</item>
    <item>com.android.systemui.volume.VolumeUI</item>
    <item>com.android.systemui.statusbar.phone.StatusBar</item>
    <item>com.android.systemui.usb.StorageNotification</item>
    <item>com.android.systemui.power.PowerUI</item>
    <item>com.android.systemui.media.RingtonePlayer</item>
    <item>com.android.systemui.keyboard.KeyboardUI</item>
    <item>com.android.systemui.shortcut.ShortcutKeyDispatcher</item>
    <item>@string/config_systemUIVendorServiceComponent</item>
    <item>com.android.systemui.util.leak.GarbageMonitor$Service</item>
    <item>com.android.systemui.LatencyTester</item>
    <item>com.android.systemui.globalactions.GlobalActionsComponent</item>
    <item>com.android.systemui.ScreenDecorations</item>
    <item>com.android.systemui.biometrics.AuthController</item>
    <item>com.android.systemui.SliceBroadcastRelayHandler</item>
    <item>com.android.systemui.statusbar.notification.InstantAppNotifier</item>
    <item>com.android.systemui.theme.ThemeOverlayController</item>
    <item>com.android.systemui.accessibility.WindowMagnification</item>
    <item>com.android.systemui.accessibility.SystemActions</item>
    <item>com.android.systemui.toast.ToastUI</item>
    <item>com.android.systemui.wmshell.WMShell</item>
</string-array>
```



### 最终的 startServicesIfNeeded

```java
private SystemUI[] mServices; //所有的组件都是SystemUI类型

private void startServicesIfNeeded(String metricsPrefix, String[] services) {
    if (mServicesStarted) {
        return;
    }
    mServices = new SystemUI[services.length];

	/*
	 * 其他代码
	 */
  
    final int N = services.length;
    for (int i = 0; i < N; i++) {
        String clsName = services[i];
        if (DEBUG) Log.d(TAG, "loading: " + clsName);
        log.traceBegin(metricsPrefix + clsName);
        long ti = System.currentTimeMillis();
        try {
        		//创建每个组件
            SystemUI obj = mComponentHelper.resolveSystemUI(clsName);
            if (obj == null) {
                Constructor constructor = Class.forName(clsName).getConstructor(Context.class);
                obj = (SystemUI) constructor.newInstance(this);
            }
            mServices[i] = obj;
        } catch (ClassNotFoundException
                | NoSuchMethodException
                | IllegalAccessException
                | InstantiationException
                | InvocationTargetException ex) {
            throw new RuntimeException(ex);
        }

        if (DEBUG) Log.d(TAG, "running: " + mServices[i]);
      	//调用每个组件的start()方法
        mServices[i].start();
        log.traceEnd();

        // Warn if initialization of component takes too long
        ti = System.currentTimeMillis() - ti;
        if (ti > 1000) {
            Log.w(TAG, "Initialization of " + clsName + " took " + ti + " ms");
        }

        if (mBootCompleteCache.isBootComplete()) {
			    	//调用每个组件的onBootCompleted()方法
            mServices[i].onBootCompleted();
        }

        dumpManager.registerDumpable(mServices[i].getClass().getName(), mServices[i]);
    }
    mSysUIComponent.getInitController().executePostInitTasks();
    log.traceEnd();

    mServicesStarted = true;
}
```

### SystemUI类

```java
/**
 * A top-level module of system UI code (sometimes called "system UI services" elsewhere in code).
 * Which SystemUI modules are loaded can be controlled via a config resource.
 *
 * @see SystemUIApplication#startServicesIfNeeded()
 */
public abstract class SystemUI implements Dumpable {
    protected final Context mContext;

    public SystemUI(Context context) {
        mContext = context;
    }

    public abstract void start();

    protected void onConfigurationChanged(Configuration newConfig) {
    }

    @Override
    public void dump(@NonNull FileDescriptor fd, @NonNull PrintWriter pw, @NonNull String[] args) {
    }

    protected void onBootCompleted() {
    }

    public static void overrideNotificationAppName(Context context, Notification.Builder n,
            boolean system) {
        final Bundle extras = new Bundle();
        String appName = system
                ? context.getString(com.android.internal.R.string.notification_app_name_system)
                : context.getString(com.android.internal.R.string.notification_app_name_settings);
        extras.putString(Notification.EXTRA_SUBSTITUTE_APP_NAME, appName);

        n.addExtras(extras);
    }
}
```



## CarSystemUI

对于CardSystemUI，首先他复用了SystemUI的代码，通过静态导入的方法，下面是CarSystemUI的Android.bp文件中导入的静态库：

```Android.bp
static_libs: [
    "SystemUI-core",
    "CarNotificationLib",
    "SystemUIPluginLib",
    "SystemUISharedLib",
    "SettingsLib",
    "car-admin-ui-lib",
    "car-ui-lib",
    "android.car.userlib",
    "car-qc-lib",
    "androidx.legacy_legacy-support-v4",
    "androidx.recyclerview_recyclerview",
    "androidx.preference_preference",
    "androidx.appcompat_appcompat",
    "androidx.mediarouter_mediarouter",
    "androidx.palette_palette",
    "androidx.legacy_legacy-preference-v14",
    "androidx.leanback_leanback",
    "androidx.slice_slice-core",
    "androidx.slice_slice-view",
    "androidx.slice_slice-builders",
    "androidx.arch.core_core-runtime",
    "androidx.lifecycle_lifecycle-extensions",
    "SystemUI-tags",
    "SystemUI-proto",
    "dagger2",
    "//external/kotlinc:kotlin-annotations",
],
```

然后，他的config.xml中指定了CarSystemUIFactory，CarSystemUIFactory 继承了SystemUIFactory。另外config.xml还定义了自己要添加的和移除的

```xml
<string name="config_systemUIFactoryComponent" translatable="false">
    com.android.systemui.CarSystemUIFactory
</string>

<!-- The list of components to exclude from config_systemUIServiceComponents. -->
<string-array name="config_systemUIServiceComponentsExclude" translatable="false">
    <item>com.android.systemui.recents.Recents</item>
    <item>com.android.systemui.volume.VolumeUI</item>
    <item>com.android.systemui.statusbar.phone.StatusBar</item>
    <item>com.android.systemui.keyboard.KeyboardUI</item>
    <item>com.android.systemui.shortcut.ShortcutKeyDispatcher</item>
    <item>com.android.systemui.LatencyTester</item>
    <item>com.android.systemui.globalactions.GlobalActionsComponent</item>
    <item>com.android.systemui.SliceBroadcastRelayHandler</item>
    <item>com.android.systemui.statusbar.notification.InstantAppNotifier</item>
    <item>com.android.systemui.accessibility.WindowMagnification</item>
    <item>com.android.systemui.accessibility.SystemActions</item>
    <item>com.android.systemui.toast.ToastUI</item>
</string-array>

<!-- The list of components to append to config_systemUIServiceComponents. -->
<string-array name="config_systemUIServiceComponentsInclude" translatable="false">
    <item>com.android.systemui.car.systembar.CarSystemBar</item>
    <item>com.android.systemui.car.voicerecognition.ConnectedDeviceVoiceRecognitionNotifier</item>
    <item>com.android.systemui.car.window.SystemUIOverlayWindowManager</item>
    <item>com.android.systemui.car.toast.CarToastUI</item>
    <item>com.android.systemui.car.volume.VolumeUI</item>
    <item>com.android.systemui.car.cluster.ClusterDisplayController</item>
</string-array>
```

然后复写了 SystemUIFactory 的 getSystemUIServiceComponents方法

```java
@Override
public String[] getSystemUIServiceComponents(Resources resources) {
    Set<String> names = new HashSet<>();

  	//先通过父类中的方法获取原来的全部组件
    //通过静态包导入的SystemUi中的config.xml会和这个包里面的config.xml合并
    for (String s : super.getSystemUIServiceComponents(resources)) {
        names.add(s);
    }

    //移除配置文件中指定的不需要的组件
    for (String s : resources.getStringArray(R.array.config_systemUIServiceComponentsExclude)) {
        names.remove(s);
    }

  	//添加配置文件中指定的新组件
    for (String s : resources.getStringArray(R.array.config_systemUIServiceComponentsInclude)) {
        names.add(s);
    }

    String[] finalNames = new String[names.size()];
    names.toArray(finalNames);

    return finalNames;
}
```

