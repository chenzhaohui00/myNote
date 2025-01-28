## 1. activity - AMS

startActivity的源码整体分三部分，第一部分是从启动activity这边自己的流程，就是通过instrumentation交给AMS的，ATMS是从Android 9.0开始引入的新架构，旨在增强对多窗口、分屏等用户交互模式的支持。

```java
Activity.startActivity
	Activity.startActivityForResult
		Instrumentation.execStartActivity //codesnap 1.1
			ActivityTaskManagerService.startActivity //交给了ATMS
```

### code snap 1.1

```java
//获取到ATMS调用startActivity
int result = ActivityTaskManager.getService().startActivity(whoThread,
    who.getOpPackageName(), who.getAttributionTag(), intent,
    intent.resolveTypeIfNeeded(who.getContentResolver()), token,
    target != null ? target.mEmbeddedID : null, requestCode, 0, null, options);
```



## 2. AMS - ActivityStarter - ApplicationThread

这部分主要的流程都在ActivityStarter，它负责从解析Intent、校验权限、确认被加入的task，到任务栈处理，最后交给task自己去resumeActivity，task会创建

```java
ActivityTaskManagerService.startActivity
	ActivityTaskManagerService.startActivityAsUser // code snap 2.0
		ActivityStarter.execute
			Request.resolveActivity //解析Intent和Activity, 获取到resolveIntent和activityInfo
			ActivityStarter.executeRequest // 校验权限, 创建ActivityRecord实例
				ActivityStarter.startActivityUnchecked
					ActivityStarter.startActivityInner
						ActivityStarter.computeLaunchingTaskFlags //计算待启动的activity的flag
						ActivityStarter.computeSourceRootTask //计算task
						Task.startActivityLocked
							Task.positionChildAtTop // 把task移到最前面
						RootWindowContainer.resumeFocusedTasksTopActivities //可能不会立刻resume，但最终会resume
							Task.resumeTopActivityUncheckedLocked
								Task.resumeTopActivityInnerLocked
									TaskFragment.resumeTopActivity
										ActivityTaskSupervisor.startSpecificActivity // code snap 2.1
											ActivityTaskSupervisor.realStartActivityLocked // code snap 2.2
												ClientLifecycleManager.scheduleTransaction
													ApplicationThread.scheduleTransaction // code snap 2.3
```

### code snap 2.0

这一部分是获取到ActivityStarter，通过builder模式，把参数都放到ActivityStarter.mRequest中，然后excute。

```java
return getActivityStartController().obtainStarter(intent, "startActivityAsUser")
        .setCaller(caller)
        .setCallingPackage(callingPackage)
        .setCallingFeatureId(callingFeatureId)
        .setResolvedType(resolvedType)
        .setResultTo(resultTo)
        .setResultWho(resultWho)
        .setRequestCode(requestCode)
        .setStartFlags(startFlags)
        .setProfilerInfo(profilerInfo)
        .setActivityOptions(bOptions)
        .setUserId(userId)
        .execute();
```

### code snap 2.1

这里是获取到待启动的进程的contoller，后面会用来获取其ApplicationThread

```java
final WindowProcessController wpc =
        mService.getProcessController(r.processName, r.info.applicationInfo.uid);
```

### code snap 2.2

这里是获取ClientTransaction实例，传递一些要处理的信息

```java
final boolean isTransitionForward = r.isTransitionForward();
clientTransaction.addCallback(LaunchActivityItem.obtain(new Intent(r.intent),
        System.identityHashCode(r), r.info,
        // TODO: Have this take the merged configuration instead of separate global
        // and override configs.
        mergedConfiguration.getGlobalConfiguration(),
        mergedConfiguration.getOverrideConfiguration(), r.compat,
        r.getFilteredReferrer(r.launchedFromPackage), task.voiceInteractor,
        proc.getReportedProcState(), r.getSavedState(), r.getPersistentSavedState(),
        results, newIntents, r.takeOptions(), isTransitionForward,
        proc.createProfilerInfoIfNeeded(), r.assistToken, activityClientController,
        r.createFixedRotationAdjustmentsIfNeeded(), r.shareableActivityToken,
        r.getLaunchedFromBubble()));

// 根据情况获取不同的生命周期的实例
final ActivityLifecycleItem lifecycleItem;
if (andResume) {
    lifecycleItem = ResumeActivityItem.obtain(isTransitionForward);
} else {
    lifecycleItem = PauseActivityItem.obtain();
}
clientTransaction.setLifecycleStateRequest(lifecycleItem);

// 通过LifeCycleManager把transaction发出去
mService.getLifecycleManager().scheduleTransaction(clientTransaction);
```

### code snap 2.3

```java
void scheduleTransaction(ClientTransaction transaction) throws RemoteException {
    //获取到了被启动Activity的app的ApplicatinThread
    final IApplicationThread client = transaction.getClient();
    //交给它去执行对应方法
    transaction.schedule();
    if (!(client instanceof Binder)) {
        // If client is not an instance of Binder - it's a remote call and at this point it is
        // safe to recycle the object. All objects used for local calls will be recycled after
        // the transaction is executed on client in ActivityThread.
        transaction.recycle();
    }
}
```



## 3. ApplicaitonThread

最后这部分就是被启动的Application自己这边的流程，就是收到要处理的Activity的生命周期方法，通过Handler从ApplicationThread发给ActivityThread，然后调用不同的方法。

对于lanchActivity，就是要创建Activity实例，Application实例（如果需要的话），attach activity，调用activity的onCreate，结束！

```java
ApplicationThread.scheduleTransaction
	ClientTransactionHandler.scheduleTransaction // 通过Handler去sendMessage, codeSnap 3.1
		ActivityThread.H.handleMessage
			TransactionExecutor.execute
				TransactionExecutor.executeCallbacks
					TransactionExecutor.cycleToPath
						TransactionExecutor.performLifecycleSequence //分发lifecycle处理方法, codeSnap 3.2
							ActivityThread.handleLaunchActivity
								ActivityThread.performLaunchActivity
									Instrumentation.newActivity // 创建Activity实例
									LoadedApk.makeApplication //创建Applicatin实例
										Instrumentation.newApplication //还是Instrumentation来创建
										Instrumentation.callApplicationOnCreate //调用application的onCreate方法
									Activity.attach //很多重要的参数设置进来的地方
										Activity.attachBaseContext //设置activity的context
										mWindow = new PhoneWindow //phoneWindow创建
										mWindow.setWindowManager //设置WindowManager
                  Instrumentation.callActivityOnCreate //调用Activity的onCreate
```

### codeSnap 3.1

```java
void scheduleTransaction(ClientTransaction transaction) {
    transaction.preExecute(this);
    sendMessage(ActivityThread.H.EXECUTE_TRANSACTION, transaction);
}
```

### codeSnap 3.2

根据不同的transaction实例，处理应该处理的生命周期方法

```java
private void performLifecycleSequence(ActivityClientRecord r, IntArray path,
        ClientTransaction transaction) {
    final int size = path.size();
    for (int i = 0, state; i < size; i++) {
        state = path.get(i);
        if (DEBUG_RESOLVER) {
            Slog.d(TAG, tId(transaction) + "Transitioning activity: "
                    + getShortActivityName(r.token, mTransactionHandler)
                    + " to state: " + getStateName(state));
        }
        switch (state) {
            case ON_CREATE:
                mTransactionHandler.handleLaunchActivity(r, mPendingActions,
                        null /* customIntent */);
                break;
            case ON_START:
                mTransactionHandler.handleStartActivity(r, mPendingActions,
                        null /* activityOptions */);
                break;
            case ON_RESUME:
                mTransactionHandler.handleResumeActivity(r, false /* finalStateRequest */,
                        r.isForward, "LIFECYCLER_RESUME_ACTIVITY");
                break;
            case ON_PAUSE:
                mTransactionHandler.handlePauseActivity(r, false /* finished */,
                        false /* userLeaving */, 0 /* configChanges */, mPendingActions,
                        "LIFECYCLER_PAUSE_ACTIVITY");
                break;
            case ON_STOP:
                mTransactionHandler.handleStopActivity(r, 0 /* configChanges */,
                        mPendingActions, false /* finalStateRequest */,
                        "LIFECYCLER_STOP_ACTIVITY");
                break;
            case ON_DESTROY:
                mTransactionHandler.handleDestroyActivity(r, false /* finishing */,
                        0 /* configChanges */, false /* getNonConfigInstance */,
                        "performLifecycleSequence. cycling to:" + path.get(size - 1));
                break;
            case ON_RESTART:
                mTransactionHandler.performRestartActivity(r, false /* start */);
                break;
            default:
                throw new IllegalArgumentException("Unexpected lifecycle state: " + state);
        }
    }
}
```

