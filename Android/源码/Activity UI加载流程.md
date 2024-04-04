## View是如何被添加到Window上的
这一节就是通过看setContentView()方法的源码，来了解activity的基础layout的创建过程，以及我们传入的layoutRes或者view是如何被inflate到指定的layout指定id的ViewGroup中的。

#### PhoneWindow
首先看Activity的setContentView()方法：
```java
public void setContentView(@LayoutRes int layoutResID) {
    getWindow().setContentView(layoutResID);
    initWindowDecorActionBar();
}
```
可以看到是调用了`getWindow()`的`setContentView()`，那么就继续去看`getWindow()`
```java
private Window mWindow;
/**
 * 很多代码...
 **/
public Window getWindow() {
    return mWindow;
}
```
得知getWindow()就是一个Window对象。
```java
 * <p>The only existing implementation of this abstract class is
 * android.view.PhoneWindow, which you should instantiate when needing a
 * Window.
 */
public abstract class Window {
/**
 * 很多代码...
 **/
}
```
继续查看Window类，发现是一个abstract类，上面的注释说明了其唯一的实现类是PhoneWindow。这里我们得知，getWindow()一定返回的是一个PhoneWindow，那么我们直接去看PhoneWindow的源码，找他的setContentView()方法。
```
@Override
public void setContentView(int layoutResID) {
    // Note: FEATURE_CONTENT_TRANSITIONS may be set in the process of installing the window
    // decor, when theme attributes and the like are crystalized. Do not check the feature
    // before this happens.
    if (mContentParent == null) {
        installDecor();
    } else if (!hasFeature(FEATURE_CONTENT_TRANSITIONS)) {
        mContentParent.removeAllViews();
    }

    if (hasFeature(FEATURE_CONTENT_TRANSITIONS)) {
        final Scene newScene = Scene.getSceneForLayout(mContentParent, layoutResID,
                getContext());
        transitionTo(newScene);
    } else {
        mLayoutInflater.inflate(layoutResID, mContentParent);
    }
    mContentParent.requestApplyInsets();
    final Callback cb = getCallback();
    if (cb != null && !isDestroyed()) {
        cb.onContentChanged();
    }
    mContentParentExplicitlySet = true;
}
```

#### installDecor()方法
首先看第一行关键代码`installDecor()`，这个方法比较长，只列出关键部分：
```java
 mForceDecorInstall = false;
if (mDecor == null) {
    mDecor = generateDecor(-1);
    mDecor.setDescendantFocusability(ViewGroup.FOCUS_AFTER_DESCENDANTS);
    mDecor.setIsRootNamespace(true);
    if (!mInvalidatePanelMenuPosted && mInvalidatePanelMenuFeatures != 0) {
        mDecor.postOnAnimation(mInvalidatePanelMenuRunnable);
    }
} else {
    mDecor.setWindow(this);
}
if (mContentParent == null) {
    mContentParent = generateLayout(mDecor);

    //其他代码...
    
}
```
显然`mDecor = generateDecor(-1);`为其第一行关键代码，继续往里看
```java
protected DecorView generateDecor(int featureId) {
    // System process doesn't have application context and in that case we need to directly use
    // the context we have. Otherwise we want the application context, so we don't cling to the
    // activity.
    Context context;
    if (mUseDecorContext) {
        Context applicationContext = getContext().getApplicationContext();
        if (applicationContext == null) {
            context = getContext();
        } else {
            context = new DecorContext(applicationContext, getContext());
            if (mTheme != -1) {
                context.setTheme(mTheme);
            }
        }
    } else {
        context = getContext();
    }
    return new DecorView(context, featureId, this, getAttributes());
}
```
重要的代码就一行，就是new了一个DecorView，如果继续点进去DecorView的构造方法，可以看到一个setWindow()方法算是重要的方法。其实就是在DecorView中存储外部的PhonWindow对象。不算重要，就不列代码了。  
然后再回到`installDecor()`，继续看下一行关键代码:
```java
if (mContentParent == null) {
    mContentParent = generateLayout(mDecor);
```
点进generateLayout继续看
```java
protected ViewGroup generateLayout(DecorView decor) {
/**
 * 省略n多代码，主要内容为根据getWindowStyle()拿到的TypedArray以及根据getAttributes()拿到的layoutParams
 * 进行各种requestFeature和setFlags，来对当前的window进行一些处理。
 **/
 
  // Inflate the window decor.

    int layoutResource;
    int features = getLocalFeatures();
    // System.out.println("Features: 0x" + Integer.toHexString(features));
    if ((features & (1 << FEATURE_SWIPE_TO_DISMISS)) != 0) {
        layoutResource = R.layout.screen_swipe_dismiss;
        setCloseOnSwipeEnabled(true);
    } else if ((features & ((1 << FEATURE_LEFT_ICON) | (1 << FEATURE_RIGHT_ICON))) != 0) {
        if (mIsFloating) {
            TypedValue res = new TypedValue();
            getContext().getTheme().resolveAttribute(
                    R.attr.dialogTitleIconsDecorLayout, res, true);
            layoutResource = res.resourceId;
        } else {
            layoutResource = R.layout.screen_title_icons;
        }
        // XXX Remove this once action bar supports these features.
        removeFeature(FEATURE_ACTION_BAR);
        // System.out.println("Title Icons!");
    } else if ((features & ((1 << FEATURE_PROGRESS) | (1 << FEATURE_INDETERMINATE_PROGRESS))) != 0
            && (features & (1 << FEATURE_ACTION_BAR)) == 0) {
        // Special case for a window with only a progress bar (and title).
        // XXX Need to have a no-title version of embedded windows.
        layoutResource = R.layout.screen_progress;
        // System.out.println("Progress!");
    } else if ((features & (1 << FEATURE_CUSTOM_TITLE)) != 0) {
        // Special case for a window with a custom title.
        // If the window is floating, we need a dialog layout
        if (mIsFloating) {
            TypedValue res = new TypedValue();
            getContext().getTheme().resolveAttribute(
                    R.attr.dialogCustomTitleDecorLayout, res, true);
            layoutResource = res.resourceId;
        } else {
            layoutResource = R.layout.screen_custom_title;
        }
        // XXX Remove this once action bar supports these features.
        removeFeature(FEATURE_ACTION_BAR);
    } else if ((features & (1 << FEATURE_NO_TITLE)) == 0) {
        // If no other features and not embedded, only need a title.
        // If the window is floating, we need a dialog layout
        if (mIsFloating) {
            TypedValue res = new TypedValue();
            getContext().getTheme().resolveAttribute(
                    R.attr.dialogTitleDecorLayout, res, true);
            layoutResource = res.resourceId;
        } else if ((features & (1 << FEATURE_ACTION_BAR)) != 0) {
            layoutResource = a.getResourceId(
                    R.styleable.Window_windowActionBarFullscreenDecorLayout,
                    R.layout.screen_action_bar);
        } else {
            layoutResource = R.layout.screen_title;
        }
        // System.out.println("Title!");
    } else if ((features & (1 << FEATURE_ACTION_MODE_OVERLAY)) != 0) {
        layoutResource = R.layout.screen_simple_overlay_action_mode;
    } else {
        // Embedded, so no decoration is needed.
        layoutResource = R.layout.screen_simple;
        // System.out.println("Simple!");
    }

    mDecor.startChanging();
    mDecor.onResourcesLoaded(mLayoutInflater, layoutResource);

    ViewGroup contentParent = (ViewGroup)findViewById(ID_ANDROID_CONTENT);

    /**
     * 又是省略一堆代码
     **/

    return contentParent;
}
```
这个超长的方法主要就是首先根据getWindowStyle()拿到的TypedArray以及根据getAttributes()拿到的layoutParams进行各种requestFeature和setFlags，来对当前的window进行一些处理。其实就是处理activity的theme主题。  
接下来就是重要的事，针对不同的主题，给layoutResource赋值了不同的布局文件。这个layoutResource就是我们activity的根布局。  
接下来我们看到了另一行关键代码`mDecor.onResourcesLoaded(mLayoutInflater, layoutResource);`  
此方法中最重要的就是下面 两行

```java
void onResourcesLoaded(LayoutInflater inflater, int layoutResource) {
    //此处省略backgroud的处理
    final View root = inflater.inflate(layoutResource, null);
    //此处省略DecorCaptinoView的处理
    addView(root, 0, new ViewGroup.LayoutParams(MATCH_PARENT, MATCH_PARENT));
}
```
可以看到其实就是将layoutResource作为根View，然后添加到DecorView中。这里提一下，DecorView本质上就是一个FrameLayout。所以就是将这个根布局填充到了DecorView中。  
继续generateLayout方法，在通过onResourcesLoaded创建了基础的layout布局以后，通过`ViewGroup contentParent = (ViewGroup)findViewById(ID_ANDROID_CONTENT);`找到固定id的content并在方法的最终返回。

```java
/**
 * The ID that the main layout in the XML layout file should have.
 */
public static final int ID_ANDROID_CONTENT = com.android.internal.R.id.content;
```
这个id所指向的View是一个FrameLayout，他就是我们在setContentView中指定的布局的父布局。  
最终在`installDecor()`方法中，将返回的`contentParent`赋给了`mContentParent`这个变量。

#### installDecor()方法总结
至此我们`installDecor()`就算是看完了，总结一下，通过`generateDecor()`创建`DecorView`实例，然后通过`generateLayout()`根据不同的theme创建不同的根布局，并找到其中的`contentParent`进行返回，最终赋给了`mContentParent`。

#### 添加我们指定的View到contentParent中
继续看`setContentView()`的源码，重要的就只剩下一行：
```java
@Override
public void setContentView(int layoutResID) {
    // Note: FEATURE_CONTENT_TRANSITIONS may be set in the process of installing the window
    // decor, when theme attributes and the like are crystalized. Do not check the feature
    // before this happens.
    if (mContentParent == null) {
        installDecor();
    } else if (!hasFeature(FEATURE_CONTENT_TRANSITIONS)) {
        mContentParent.removeAllViews();
    }

    if (hasFeature(FEATURE_CONTENT_TRANSITIONS)) {
        final Scene newScene = Scene.getSceneForLayout(mContentParent, layoutResID,
                getContext());
        transitionTo(newScene);
    } else {
        mLayoutInflater.inflate(layoutResID, mContentParent);
    }
    mContentParent.requestApplyInsets();
    final Callback cb = getCallback();
    if (cb != null && !isDestroyed()) {
        cb.onContentChanged();
    }
    mContentParentExplicitlySet = true;
}
```
就是这行`mLayoutInflater.inflate(layoutResID, mContentParent);`  
可以看到就是将我们指定的layout，`inflate`并添加到前面赋过值的mContentParent中。

#### 总结
当我们执行`setContentView()`时，首先会在PhoneWindow中创建一个DecorView，然后在DecorView根据不同的theme等创建不同的根布局，并找到其中的contentParent，最终将我们指定的布局添加到contentParent中。  
这就是View被添加到Window的过程。

## View的绘制流程

#### 绘制入口
在加载Activity时，会在ActivityThread中的Handler类H中，通过handleMessage()执行一个方法：`handleResumeActivity()`:
```java
@Override
public void handleResumeActivity(ActivityClientRecord r, boolean finalStateRequest,boolean isForward, String reason) {
		...
    if (r.window == null && !a.mFinished && willBeVisible) {
          r.window = r.activity.getWindow();
          View decor = r.window.getDecorView();
          decor.setVisibility(View.INVISIBLE);
      		//1.这里获取了activity的windowManager，它在activity#attach时创建的
          ViewManager wm = a.getWindowManager();
        	...
          if (a.mVisibleFromClient) {
              if (!a.mWindowAdded) {
                  a.mWindowAdded = true;
                  //2.将decor添加到windowManager中
                  wm.addView(decor, l);
              } else {
                  // The activity will get a callback for this {@link LayoutParams} change
                  // earlier. However, at that time the decor will not be set (this is set
                  // in this method), so no action will be taken. This call ensures the
                  // callback occurs with the decor set.
                  a.onWindowAttributesChanged(l);
              }
          }
    }
  	
}
```
1处的wm在activity被launch的时候(即ActivityThread#performLaunchActivity)，调用了Activity的attach方法，在其中获取到的：

```java
@UnsupportedAppUsage(maxTargetSdk = Build.VERSION_CODES.R, trackingBug = 170729553)
final void attach(Context context, ActivityThread aThread,
						Instrumentation instr, IBinder token, int ident,
            Application application, Intent intent, ActivityInfo info,
            CharSequence title, Activity parent, String id,
            NonConfigurationInstances lastNonConfigurationInstances,
            Configuration config, String referrer, IVoiceInteractor voiceInteractor,
            Window window, ActivityConfigCallback activityConfigCallback, IBinder assistToken,
            IBinder shareableActivityToken) {
      attachBaseContext(context);

      mFragments.attachHost(null /*parent*/);

  		//创建phoneWindow对象
      mWindow = new PhoneWindow(this, window, activityConfigCallback);
  		//此方法内部创建了一个WindowManagerImpl实例
      mWindow.setWindowManager(
                (WindowManager)context.getSystemService(Context.WINDOW_SERVICE),
                mToken, mComponent.flattenToString(),
                (info.flags & ActivityInfo.FLAG_HARDWARE_ACCELERATED) != 0);
        if (mParent != null) {
            mWindow.setContainer(mParent.getWindow());
        }
  			//获取到这个对象，存为activity自己的成员变量
        mWindowManager = mWindow.getWindowManager();
}
```

通过1处的分析现在我们知道了这个wm对象是一个WindowManagerImpl实例，接下来去看2处的wm.addView方法，WindowManagerImpl内部调用了一个单例对象WIndowManagerGlobal的addView方法：

```java
public void addView(View view, ViewGroup.LayoutParams params,
            Display display, Window parentWindow, int userId) {
  	...
    ViewRootImpl root;
  	synchronized (mLock) {
      	...
        //1.创建root对象
        root = new ViewRootImpl(view.getContext(), display);

        view.setLayoutParams(wparams);

      	//不用管，这里就是这个单例对象里面存了所有通过addView方法add的view、root和params
        mViews.add(view);
        mRoots.add(root);
        mParams.add(wparams);

        // do this last because it fires off messages to start doing things
        try {
          	//2.调用ViewRootImpl#setView
            root.setView(view, wparams, panelParentView, userId);
        } catch (RuntimeException e) {
            // BadTokenException or InvalidDisplayException, clean up.
            if (index >= 0) {
                removeViewLocked(index, true);
            }
            throw e;
        }
    }
}
```

最后看ViewRootImpl#setView：

```java
public void setView(View view, WindowManager.LayoutParams attrs, View panelParentView, int userId) {
  	...
  	int res; /* = WindowManagerImpl.ADD_OKAY; */
  	
  	//1.这里触发绘制
  	// Schedule the first layout -before- adding to the window
    // manager, to make sure we do the relayout before receiving
    // any other events from the system.
    requestLayout();
  
  	try {
          ...
          //2.真正的把window加到wms中
          res = mWindowSession.addToDisplayAsUser(mWindow, mWindowAttributes,
                  getHostVisibility(), mDisplay.getDisplayId(), userId,
                  mInsetsController.getRequestedVisibility(), inputChannel, mTempInsets,
                  mTempControls);
          ...
    }
}
```

到此为止，把window通过addView真正添加到wms中，并且通过requestLayout请求了绘制，我们的UI页面要显示出来了。
总结起来就是：  

```java
ActivityThread.handleResumeActivity()
->WindowMangerImple.addView(decorView, LayoutParams)
->WindowManagerGlobal.addView()
->ViewRootImpl.setView()
->requestLayout()
```

#### 绘制的类及方法
在`WindowManagerGlobal.addView()`中初始化了一个ViewRoot，然后将decorView添加到了ViewRoot中：
```java
ViewRootImpl.setView(decorView, layoutParams, parentView);
```
接下来，在setView中调用了如下方法进行绘制：
```java
ViewRootImpl.requestLayout()->scheduleTraversals()->doTraversals()->performTraversals();
```

#### 绘制的三大步骤
最后在`performTraversals()`中，通过一下的三个步骤，进行了绘制：
```java
ViewRootImpl.performMeasure() //测量
ViewRootImpl.performLayout() //布局
ViewRootImpl.performDraw() //绘制
```