## 1. activity - AMS

```java
Activity.startActivity
	Activity.startActivityForResult //skip some simple steps.
		Instrumentation.execStartActivity //codesnap 1.1
			ActivityTaskManagerService.startActivity //ATMS is new name of AMS since Android8.0
```

### code snap 1.1

```java
int result = ActivityTaskManager.getService().startActivity(whoThread,
    who.getOpPackageName(), who.getAttributionTag(), intent,
    intent.resolveTypeIfNeeded(who.getContentResolver()), token,
    target != null ? target.mEmbeddedID : null, requestCode, 0, null, options);
```



## 2. AMS - ActivityStarter - ApplicationThread

```java
ActivityTaskManagerService.startActivity
	ActivityTaskManagerService.startActivityAsUser // code snap 2.0
		ActivityStarter.execute
			Request.resolveActivity //resove Intent and Activity, get resolveIntent and activityInfo and so on
			ActivityStarter.executeRequest // check permission, create ActivityRecord instance
				ActivityStarter.startActivityUnchecked
					ActivityStarter.startActivityInner // task handle
						ActivityStarter.computeLaunchingTaskFlags
						ActivityStarter.computeSourceRootTask
						Task.startActivityLocked
							Task.positionChildAtTop //move task to top
						RootWindowContainer.resumeFocusedTasksTopActivities
							Task.resumeTopActivityUncheckedLocked
								Task.resumeTopActivityInnerLocked
									TaskFragment.resumeTopActivity
										ActivityTaskSupervisor.startSpecificActivity // code snap 2.2
											ActivityTaskSupervisor.realStartActivityLocked // code snap 2.2
												ClientLifecycleManager.scheduleTransaction
													ApplicationThread.scheduleTransaction
```

### code snap 2.0

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

```java
final WindowProcessController wpc =
        mService.getProcessController(r.processName, r.info.applicationInfo.uid);
```

### code snap 2.2

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

// Set desired final state.
final ActivityLifecycleItem lifecycleItem;
if (andResume) {
    lifecycleItem = ResumeActivityItem.obtain(isTransitionForward);
} else {
    lifecycleItem = PauseActivityItem.obtain();
}
clientTransaction.setLifecycleStateRequest(lifecycleItem);

// Schedule transaction.
mService.getLifecycleManager().scheduleTransaction(clientTransaction);
```

### code snap 2.3

```java
void scheduleTransaction(ClientTransaction transaction) throws RemoteException {
    //
    final IApplicationThread client = transaction.getClient();
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

```java
ApplicationThread.scheduleTransaction
	ClientTransactionHandler.scheduleTransaction // sendMessage, codeSnap 3.1
		ActivityThread.H.handleMessage
			TransactionExecutor.execute
				TransactionExecutor.executeCallbacks
					TransactionExecutor.cycleToPath
						TransactionExecutor.performLifecycleSequence //dispatch lifecycle handle method, codeSnap 3.2
							ActivityThread.handleLaunchActivity
								ActivityThread.performLaunchActivity
									Instrumentation.newActivity
									LoadedApk.makeApplication
										Instrumentation.newApplication
										Instrumentation.callApplicationOnCreate
									Activity.attach
										Activity.attachBaseContext
										mWindow = new PhoneWindow
										mWindow.setWindowManager
                                    Instrumentation.callActivityOnCreate
```

### codeSnap 3.1

```java
void scheduleTransaction(ClientTransaction transaction) {
    transaction.preExecute(this);
    sendMessage(ActivityThread.H.EXECUTE_TRANSACTION, transaction);
}
```

### codeSnap 3.2

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

