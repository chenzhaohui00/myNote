## SystemServer - SystemUIService

```java
SystemServer.main // code snap 1.1
	SystemServer.run // code snap 1.2
		SystemServer.startOtherServices
			SystemServer.startSystemUi
```

### code snap 1.1

```java
/**
 * The main entry point from zygote.
 */
public static void main(String[] args) {
    new SystemServer().run();
}
```

### code snap 1.2

```java
private void run() {
	/*
	 * other ocde
	 */
	 
    // Start services.
    try {
        t.traceBegin("StartServices");
        startBootstrapServices(t);
        startCoreServices(t);
        startOtherServices(t);
    } catch (Throwable ex) {
        Slog.e("System", "******************************************");
        Slog.e("System", "************ Failure starting system services", ex);
        throw ex;
    } finally {
        t.traceEnd(); // StartServices
    }
}
```



## SystemUIService - SystemUIApplication

```
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



```java
public String[] getSystemUIServiceComponents(Resources resources) {
    return resources.getStringArray(R.array.config_systemUIServiceComponents);
}
```



### config.xml

```xml
<!-- SystemUIFactory component -->
<string name="config_systemUIFactoryComponent" translatable="false">com.android.systemui.SystemUIFactory</string>

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



### startServicesIfNeeded

```java
private SystemUI[] mServices;

private void startServicesIfNeeded(String metricsPrefix, String[] services) {
    if (mServicesStarted) {
        return;
    }
    mServices = new SystemUI[services.length];

	/*
	 * other ocde
	 */
    
    final int N = services.length;
    for (int i = 0; i < N; i++) {
        String clsName = services[i];
        if (DEBUG) Log.d(TAG, "loading: " + clsName);
        log.traceBegin(metricsPrefix + clsName);
        long ti = System.currentTimeMillis();
        try {
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
        mServices[i].start();
        log.traceEnd();

        // Warn if initialization of component takes too long
        ti = System.currentTimeMillis() - ti;
        if (ti > 1000) {
            Log.w(TAG, "Initialization of " + clsName + " took " + ti + " ms");
        }
        if (mBootCompleteCache.isBootComplete()) {
            mServices[i].onBootCompleted();
        }

        dumpManager.registerDumpable(mServices[i].getClass().getName(), mServices[i]);
    }
    mSysUIComponent.getInitController().executePostInitTasks();
    log.traceEnd();

    mServicesStarted = true;
}
```



## CarSystemUI



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



```java
    @Override
    public String[] getSystemUIServiceComponents(Resources resources) {
        Set<String> names = new HashSet<>();

        for (String s : super.getSystemUIServiceComponents(resources)) {
            names.add(s);
        }

        for (String s : resources.getStringArray(R.array.config_systemUIServiceComponentsExclude)) {
            names.remove(s);
        }

        for (String s : resources.getStringArray(R.array.config_systemUIServiceComponentsInclude)) {
            names.add(s);
        }

        String[] finalNames = new String[names.size()];
        names.toArray(finalNames);

        return finalNames;
    }
```





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

