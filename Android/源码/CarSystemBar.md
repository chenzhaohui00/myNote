## CarSystemBar.start

```java
private final CommandQueue mCommandQueue;
private final IStatusBarService mBarService;

public void start() {
	// Connect into the status bar manager service
    mCommandQueue.addCallback(this);
    
    RegisterStatusBarResult result = null;
    try {
        result = mBarService.registerStatusBar(mCommandQueue);
    } catch (RemoteException ex) {
        ex.rethrowFromSystemServer();
    }
    
    createSystemBar(result);
}
```



## createSystemBar

```java
//CarSystemBar
private void createSystemBar(RegisterStatusBarResult result) {
    buildNavBarWindows();
    buildNavBarContent();
    attachNavBarWindows();
}
```

### buildNavBarWindows

```java
private void buildNavBarWindows() {
    mTopSystemBarWindow = mCarSystemBarController.getTopWindow();
    mBottomSystemBarWindow = mCarSystemBarController.getBottomWindow();
    mLeftSystemBarWindow = mCarSystemBarController.getLeftWindow();
    mRightSystemBarWindow = mCarSystemBarController.getRightWindow();
}

// CarSystemBarController
/** Gets the top window if configured to do so. */
@Nullable
public ViewGroup getTopWindow() {
    return mShowTop ? mCarSystemBarViewFactory.getTopWindow() : null;
}

// CarSystemBarViewFactory
/** Gets the top window. */
public ViewGroup getTopWindow() {
    return getWindowCached(Type.TOP);
}

private final ArrayMap<Type, ViewGroup> mCachedContainerMap = new ArrayMap<>();

private ViewGroup getWindowCached(Type type) {
    if (mCachedContainerMap.containsKey(type)) {
        return mCachedContainerMap.get(type);
    }

    ViewGroup window = (ViewGroup) View.inflate(mContext,
            R.layout.navigation_bar_window, /* root= */ null);
    mCachedContainerMap.put(type, window);
    return mCachedContainerMap.get(type);
}
```

### buildNavBarContent

```java
private void buildNavBarContent() {
    mTopSystemBarView = mCarSystemBarController.getTopBar(isDeviceSetupForUser());
    if (mTopSystemBarView != null) {
        mSystemBarConfigs.insetSystemBar(SystemBarConfigs.TOP, mTopSystemBarView);
        mHvacController.registerHvacViews(mTopSystemBarView);
        mTopSystemBarWindow.addView(mTopSystemBarView);
    }
    
    mBottomSystemBarView = mCarSystemBarController.getBottomBar(isDeviceSetupForUser());
    if (mBottomSystemBarView != null) {
        mSystemBarConfigs.insetSystemBar(SystemBarConfigs.BOTTOM, mBottomSystemBarView);
        mHvacController.registerHvacViews(mBottomSystemBarView);
        mBottomSystemBarWindow.addView(mBottomSystemBarView);
    }

    mLeftSystemBarView = mCarSystemBarController.getLeftBar(isDeviceSetupForUser());
    if (mLeftSystemBarView != null) {
        mSystemBarConfigs.insetSystemBar(SystemBarConfigs.LEFT, mLeftSystemBarView);
        mHvacController.registerHvacViews(mLeftSystemBarView);
        mLeftSystemBarWindow.addView(mLeftSystemBarView);
    }

    mRightSystemBarView = mCarSystemBarController.getRightBar(isDeviceSetupForUser());
    if (mRightSystemBarView != null) {
        mSystemBarConfigs.insetSystemBar(SystemBarConfigs.RIGHT, mRightSystemBarView);
        mHvacController.registerHvacViews(mRightSystemBarView);
        mRightSystemBarWindow.addView(mRightSystemBarView);
    }
}

public CarSystemBarView getTopBar(boolean isSetUp) {
    if (!mShowTop) {
        return null;
    }

    mTopView = mCarSystemBarViewFactory.getTopBar(isSetUp);
    setupBar(mTopView, mTopBarTouchListener, mNotificationsShadeController,
            mHvacPanelController, mHvacPanelOverlayViewController);

    return mTopView;
}

// CarSystemBarViewFactory
public CarSystemBarView getTopBar(boolean isSetUp) {
    return getBar(isSetUp, Type.TOP, Type.TOP_UNPROVISIONED);
}

private CarSystemBarView getBar(boolean isSetUp, Type provisioned, Type unprovisioned) {
    CarSystemBarView view = getBarCached(isSetUp, provisioned, unprovisioned);

    if (view == null) {
        String name = isSetUp ? provisioned.name() : unprovisioned.name();
        Log.e(TAG, "CarStatusBar failed inflate for " + name);
        throw new RuntimeException(
                "Unable to build " + name + " nav bar due to missing layout");
    }
    return view;
}

private CarSystemBarView getBarCached(boolean isSetUp, Type provisioned, Type unprovisioned) {
    Type type = isSetUp ? provisioned : unprovisioned;
    if (mCachedViewMap.containsKey(type)) {
        return mCachedViewMap.get(type);
    }

    @LayoutRes int barLayout = sLayoutMap.get(type);
    CarSystemBarView view = (CarSystemBarView) View.inflate(mContext, barLayout,
            /* root= */ null);

    view.setupHvacButton();
    view.setupQuickControlsEntryPoints(mQuickControlsEntryPointsController, isSetUp);
    view.setupReadOnlyIcons(mReadOnlyIconsController);

    // Include a FocusParkingView at the beginning. The rotary controller "parks" the focus here
    // when the user navigates to another window. This is also used to prevent wrap-around.
    view.addView(new FocusParkingView(mContext), 0);

    mCachedViewMap.put(type, view);
    return mCachedViewMap.get(type);
}

//
protected void insetSystemBar(@SystemBarSide int side, CarSystemBarView view) {
    if (mSystemBarConfigMap.get(side) == null) return;

    int[] paddings = mSystemBarConfigMap.get(side).getPaddings();
    view.setPadding(paddings[2], paddings[0], paddings[3], paddings[1]);
}
```



```java
private void attachNavBarWindows() {
    mSystemBarConfigs.getSystemBarSidesByZOrder().forEach(this::attachNavBarBySide);
}

private void attachNavBarBySide(int side) {
    switch (side) {
        case SystemBarConfigs.TOP:
            if (mTopSystemBarWindow != null) {
                mWindowManager.addView(mTopSystemBarWindow,
                        mSystemBarConfigs.getLayoutParamsBySide(SystemBarConfigs.TOP));
            }
            break;
        case SystemBarConfigs.BOTTOM:
            if (mBottomSystemBarWindow != null && !mBottomNavBarVisible) {
                mBottomNavBarVisible = true;

                mWindowManager.addView(mBottomSystemBarWindow,
                        mSystemBarConfigs.getLayoutParamsBySide(SystemBarConfigs.BOTTOM));
            }
            break;
        case SystemBarConfigs.LEFT:
            if (mLeftSystemBarWindow != null) {
                mWindowManager.addView(mLeftSystemBarWindow,
                        mSystemBarConfigs.getLayoutParamsBySide(SystemBarConfigs.LEFT));
            }
            break;
        case SystemBarConfigs.RIGHT:
            if (mRightSystemBarWindow != null) {
                mWindowManager.addView(mRightSystemBarWindow,
                        mSystemBarConfigs.getLayoutParamsBySide(SystemBarConfigs.RIGHT));
            }
            break;
        default:
            return;
    }
}

//SystemBarConfigs
private final List<@SystemBarSide Integer> mSystemBarSidesByZOrder = new ArrayList<>();

protected List<Integer> getSystemBarSidesByZOrder() {
    return mSystemBarSidesByZOrder;
}

//config.xml
<!-- Configure the relative z-order among the system bars. When two system bars overlap (e.g.
     if both top bar and left bar are enabled, it creates an overlapping space in the upper left
     corner), the system bar with the higher z-order takes the overlapping space and padding is
     applied to the other bar.-->
<!-- NOTE: If two overlapping system bars have the same z-order, SystemBarConfigs will throw a
     RuntimeException, since their placing order cannot be determined. Bars that do not overlap
     are allowed to have the same z-order. -->
<!-- NOTE: If the z-order of a bar is 10 or above, it will also appear on top of HUN's.    -->
<integer name="config_topSystemBarZOrder">1</integer>
<integer name="config_leftSystemBarZOrder">0</integer>
<integer name="config_rightSystemBarZOrder">0</integer>
<integer name="config_bottomSystemBarZOrder">10</integer>
```



### SystemBarConfigs

```java
//SystemBarConfigs
private static final int[] BAR_TYPE_MAP = {
        InsetsState.ITYPE_STATUS_BAR,
        InsetsState.ITYPE_NAVIGATION_BAR,
        InsetsState.ITYPE_CLIMATE_BAR,
        InsetsState.ITYPE_EXTRA_NAVIGATION_BAR
};

private void readConfigs() {
    mTopNavBarEnabled = mResources.getBoolean(R.bool.config_enableTopSystemBar);
    mBottomNavBarEnabled = mResources.getBoolean(R.bool.config_enableBottomSystemBar);
    mLeftNavBarEnabled = mResources.getBoolean(R.bool.config_enableLeftSystemBar);
    mRightNavBarEnabled = mResources.getBoolean(R.bool.config_enableRightSystemBar);

    if (mTopNavBarEnabled) {
        SystemBarConfig topBarConfig =
                new SystemBarConfigBuilder()
                        .setSide(TOP)
                        .setGirth(mResources.getDimensionPixelSize(
                                R.dimen.car_top_system_bar_height))
                        .setBarType(mResources.getInteger(R.integer.config_topSystemBarType))
                        .setZOrder(mResources.getInteger(R.integer.config_topSystemBarZOrder))
                        .setHideForKeyboard(mResources.getBoolean(
                                R.bool.config_hideTopSystemBarForKeyboard))
                        .build();
        mSystemBarConfigMap.put(TOP, topBarConfig);
    }

    if (mBottomNavBarEnabled) {
        SystemBarConfig bottomBarConfig =
                new SystemBarConfigBuilder()
                        .setSide(BOTTOM)
                        .setGirth(mResources.getDimensionPixelSize(
                                R.dimen.car_bottom_system_bar_height))
                        .setBarType(mResources.getInteger(R.integer.config_bottomSystemBarType))
                        .setZOrder(
                                mResources.getInteger(R.integer.config_bottomSystemBarZOrder))
                        .setHideForKeyboard(mResources.getBoolean(
                                R.bool.config_hideBottomSystemBarForKeyboard))
                        .build();
        mSystemBarConfigMap.put(BOTTOM, bottomBarConfig);
    }

    if (mLeftNavBarEnabled) {
        SystemBarConfig leftBarConfig =
                new SystemBarConfigBuilder()
                        .setSide(LEFT)
                        .setGirth(mResources.getDimensionPixelSize(
                                R.dimen.car_left_system_bar_width))
                        .setBarType(mResources.getInteger(R.integer.config_leftSystemBarType))
                        .setZOrder(mResources.getInteger(R.integer.config_leftSystemBarZOrder))
                        .setHideForKeyboard(mResources.getBoolean(
                                R.bool.config_hideLeftSystemBarForKeyboard))
                        .build();
        mSystemBarConfigMap.put(LEFT, leftBarConfig);
    }

    if (mRightNavBarEnabled) {
        SystemBarConfig rightBarConfig =
                new SystemBarConfigBuilder()
                        .setSide(RIGHT)
                        .setGirth(mResources.getDimensionPixelSize(
                                R.dimen.car_right_system_bar_width))
                        .setBarType(mResources.getInteger(R.integer.config_rightSystemBarType))
                        .setZOrder(mResources.getInteger(R.integer.config_rightSystemBarZOrder))
                        .setHideForKeyboard(mResources.getBoolean(
                                R.bool.config_hideRightSystemBarForKeyboard))
                        .build();
        mSystemBarConfigMap.put(RIGHT, rightBarConfig);
    }
}

// InsetsState
static final int FIRST_TYPE = 0;

public static final int ITYPE_STATUS_BAR = FIRST_TYPE;
public static final int ITYPE_NAVIGATION_BAR = 1;
public static final int ITYPE_CAPTION_BAR = 2;
public static final int ITYPE_TOP_GESTURES = 3;

//config.xml
<!-- Configure the type of each system bar. Each system bar must have a unique type. -->
<!--    STATUS_BAR = 0-->
<!--    NAVIGATION_BAR = 1-->
<!--    STATUS_BAR_EXTRA = 2-->
<!--    NAVIGATION_BAR_EXTRA = 3-->
<integer name="config_topSystemBarType">0</integer>
<integer name="config_leftSystemBarType">2</integer>
<integer name="config_rightSystemBarType">3</integer>
<integer name="config_bottomSystemBarType">1</integer>

    
protected WindowManager.LayoutParams getLayoutParamsBySide(@SystemBarSide int side) {
	return mSystemBarConfigMap.get(side) != null
        ? mSystemBarConfigMap.get(side).getLayoutParams() : null;
}

//SystemBarConfig
private WindowManager.LayoutParams getLayoutParams() {
    WindowManager.LayoutParams lp = new WindowManager.LayoutParams(
            isHorizontalBar(mSide) ? ViewGroup.LayoutParams.MATCH_PARENT : mGirth,
            isHorizontalBar(mSide) ? mGirth : ViewGroup.LayoutParams.MATCH_PARENT,
            mapZOrderToBarType(mZOrder),
            WindowManager.LayoutParams.FLAG_NOT_FOCUSABLE
                    | WindowManager.LayoutParams.FLAG_NOT_TOUCH_MODAL
                    | WindowManager.LayoutParams.FLAG_WATCH_OUTSIDE_TOUCH
                    | WindowManager.LayoutParams.FLAG_SPLIT_TOUCH,
            PixelFormat.TRANSLUCENT);
    lp.setTitle(BAR_TITLE_MAP.get(mSide));
    lp.providesInsetsTypes = new int[]{BAR_TYPE_MAP[mBarType], BAR_GESTURE_MAP.get(mSide)};
    lp.setFitInsetsTypes(0);
    lp.windowAnimations = 0;
    lp.gravity = BAR_GRAVITY_MAP.get(mSide);
    return lp;
}

private int mapZOrderToBarType(int zOrder) {
    return zOrder >= HUN_ZORDER ? WindowManager.LayoutParams.TYPE_NAVIGATION_BAR_PANEL
            : WindowManager.LayoutParams.TYPE_STATUS_BAR_ADDITIONAL;
}
private static final int HUN_ZORDER = 10;
```



