# Activity启动流程分析

[TOC]

### 1 Activity--`startActivityForResult`

Activity的启动一般有了两种方式，一种是需要返回结果，一种是不需要返回记过的。但是最终都会调用以下的方法。

```java
public void startActivityForResult(@RequiresPermission Intent intent, int requestCode,
            @Nullable Bundle options) {
    //mParent不为空的情况是在使用TabActivity的情况，但现在多被Fragment所代替
        if (mParent == null) {
            options = transferSpringboardActivityOptions(options);
            Instrumentation.ActivityResult ar =
                mInstrumentation.execStartActivity(
                    this, mMainThread.getApplicationThread(), mToken, this,
                    intent, requestCode, options);
            if (ar != null) {
                mMainThread.sendActivityResult(
                    mToken, mEmbeddedID, requestCode, ar.getResultCode(),
                    ar.getResultData());
            }
            if (requestCode >= 0) {
                mStartedActivity = true;
            }
			//取消即将要执行的时间 ，开始退出动画
            cancelInputsAndStartExitTransition(options);
        } else {
           ....
        }
    }
```

* `mInstrumentation` 类型 `Instrumentation`，是用来监控程序与系统之间的交互操作。
* `mMainThread` 类型`ActivityThread` ，是用来描述应用程序进程的，系统每当启动一个应用程序进程时，都会创建一个`ActivityThread` 对象。
* `mMainThread.getApplicationThread()`获取一个类型为`ApplicationThread`对象，最后交给AMS，这样AMS就可以通过`ApplicationThread`与`Activity`交流了。
* `mToken` 类型 `IBinder`，它是Binder的代理对象，指向AMS中一个类型为`ActivityRecord`对象，用来维护对应`Activity`组件的运行状态以及信息。该对象也交给AMS，让其能获得`Activity`的详细信息。
* `mInstrumentation`、`mMainThread`、`mToken`三个对象均是在`Activity`的`attach()`被赋值的。



#### 1.1 Instrumentation--`execStartActivity`

```java
public ActivityResult execStartActivity(
            Context who, IBinder contextThread, IBinder token, Activity target,
            Intent intent, int requestCode, Bundle options) {
    	IApplicationThread whoThread = (IApplicationThread) contextThread;
      	...
        try {
            intent.migrateExtraStreamToClipData();
            intent.prepareToLeaveProcess(who);
            
            int result = ActivityManagerNative.getDefault()
                .startActivity(whoThread, who.getBasePackageName(), intent,
                        intent.resolveTypeIfNeeded(who.getContentResolver()),
                        token, target != null ? target.mEmbeddedID : null,
                        requestCode, 0, null, options);
            
            checkStartActivityResult(result, intent);
        } catch (RemoteException e) {
            throw new RuntimeException("Failure from system", e);
        }
        return null;
    }
```

* `ActivityManagerNative.getDefault()`返回了一个`ActivityManagerProxy`类型的单例对象，该对象是AMS的代理对象。
* 通过调用代理对象的`startActivity()`方法，通知AMS启动`Activity`。



#### 1.2 ActivityManagerProxy -- `startActivity`

```java
    public int startActivity(IApplicationThread caller, String callingPackage, Intent intent,
            String resolvedType, IBinder resultTo, String resultWho, int requestCode,
            int startFlags, ProfilerInfo profilerInfo, Bundle options) throws RemoteException {
        Parcel data = Parcel.obtain();
        Parcel reply = Parcel.obtain();
 		
        ...
        
        mRemote.transact(START_ACTIVITY_TRANSACTION, data, reply, 0);
        
        reply.readException();
        int result = reply.readInt();
        reply.recycle();
        data.recycle();
        return result;
    }
```

* `mRemote`是AMS的代理对象，调用`transact`方法，向AMS发送一个`START_ACTIVITY_TRANSACTION`的进程间通信请求
* `transact`方法在**Binder**类中实现，`ActivityManagerService`继承**Binder**



#### 1.3 ActivityManagerService-- `transact`

```java
public final boolean transact(int code, Parcel data, Parcel reply,
            int flags) throws RemoteException {
		....
        boolean r = onTransact(code, data, reply, flags);
		...
        return r;
    }
```

* 将通信请求转交给`onTransact`方法处理



#### 1.4 ActivityManagerService-- `onTransact`

```java
public boolean onTransact(int code, Parcel data, Parcel reply, int flags)
            throws RemoteException {
        switch (code) {
        case START_ACTIVITY_TRANSACTION:
        {
			...
            int result = startActivity(app, callingPackage, intent, resolvedType,
                    resultTo, resultWho, requestCode, startFlags, profilerInfo, options);
            reply.writeNoException();
            reply.writeInt(result);
            return true;
        }
        ...
}
```

* 最终调用`ActivityManagerService`的`startActivity`方法。



#### 1.5 ActivityManagerService-- `startActivity`

```java
 public final int startActivity(IApplicationThread caller, String callingPackage,
            Intent intent, String resolvedType, IBinder resultTo, String resultWho, int requestCode,
            int startFlags, ProfilerInfo profilerInfo, Bundle bOptions) {
        return startActivityAsUser(caller, callingPackage, intent, resolvedType, resultTo,
                resultWho, requestCode, startFlags, profilerInfo, bOptions,
                UserHandle.getCallingUserId());
    }
```

* 然后调用`startActivityAsUser`方法



#### 1.6 ActivityManagerService-- `startActivityAsUser`

```java
  public final int startActivityAsUser(IApplicationThread caller, String callingPackage,
            Intent intent, String resolvedType, IBinder resultTo, String resultWho, int requestCode,
            int startFlags, ProfilerInfo profilerInfo, Bundle bOptions, int userId) {
		....
        
      
        return mActivityStarter.startActivityMayWait(caller, -1, callingPackage, intent,
                resolvedType, null, null, resultTo, resultWho, requestCode, startFlags,
                profilerInfo, null, null, bOptions, false, userId, null, null);
    }

```

* `mActivityStarter` 类型 `ActivityStarter`，它是Activity启动等操作的管理者



#### 1.7 ActivityStarter-- `startActivityMayWait`

```java
    final int startActivityMayWait(IApplicationThread caller, int callingUid,
            String callingPackage, Intent intent, String resolvedType,
            IVoiceInteractionSession voiceSession, IVoiceInteractor voiceInteractor,
            IBinder resultTo, String resultWho, int requestCode, int startFlags,
            ProfilerInfo profilerInfo, IActivityManager.WaitResult outResult, Configuration config,
            Bundle bOptions, boolean ignoreTargetSecurity, int userId,
            IActivityContainer iContainer, TaskRecord inTask) {
		...
        // 是否指定了要启动Activity相关的组件名，如果指定了组件名则为显示启动，否则为隐式启动
        boolean componentSpecified = intent.getComponent() != null;

        // 保存传入的intent信息
        final Intent ephemeralIntent = new Intent(intent);
        // 重新创建intent 防止 传入的intent对象被修改
        intent = new Intent(intent);

 		...
        // 收集Intent中的信息
        ...
        synchronized (mService) {
           	...
            final ActivityRecord[] outRecord = new ActivityRecord[1];
            int res = startActivityLocked(caller, intent, ephemeralIntent, resolvedType,
                    aInfo, rInfo, voiceSession, voiceInteractor,
                    resultTo, resultWho, requestCode, callingPid,
                    callingUid, callingPackage, realCallingPid, realCallingUid, startFlags,
                    options, ignoreTargetSecurity, componentSpecified, outRecord, container,
                    inTask);

            ...
            return res;
        }
    }
```

* 该方法主要是解析收集Intent的内容信息，然后进行相应的判断处理，最后调用`startActivityLocked`



#### 1.8 ActivityStarter-- `startActivityLocked`

```java
final int startActivityLocked(IApplicationThread caller, Intent intent, Intent ephemeralIntent,
            String resolvedType, ActivityInfo aInfo, ResolveInfo rInfo,
            IVoiceInteractionSession voiceSession, IVoiceInteractor voiceInteractor,
            IBinder resultTo, String resultWho, int requestCode, int callingPid, int callingUid,
            String callingPackage, int realCallingPid, int realCallingUid, int startFlags,
            ActivityOptions options, boolean ignoreTargetSecurity, boolean componentSpecified,
            ActivityRecord[] outActivity, ActivityStackSupervisor.ActivityContainer container,
            TaskRecord inTask) {
        int err = ActivityManager.START_SUCCESS;
		// 获取调用者进程记录对象，每一个进程都使用一个ProcessRecord对象来描述，并且会保存起来
        ProcessRecord callerApp = null;
        if (caller != null) {
            //caller指向为源Activity应用程序所在进程的ApplicationThread对象
            //AMS通过caller获取对应的ProcessRecord对象
            //callerApp指向了源Activity（调用者）所在的应用程序进程
            callerApp = mService.getRecordForAppLocked(caller);
            if (callerApp != null) {
                callingPid = callerApp.pid;
                callingUid = callerApp.info.uid;
            } else {
                //调用者被系统杀死或者意外退出
                Slog.w(TAG, "Unable to find app for caller " + caller
                        + " (pid=" + callingPid + ") when starting: "
                        + intent.toString());
                err = ActivityManager.START_PERMISSION_DENIED;
            }
        }

		...
        //调用者Activity封装    
        ActivityRecord sourceRecord = null;
    	//需要接受返回结果的Activity对象封装
        ActivityRecord resultRecord = null;
        if (resultTo != null) {
            //如果需要返回结果
            //然后查找栈中是否存在对应调用者Activity的ActivityRecord
            sourceRecord = mSupervisor.isInAnyStackLocked(resultTo);
            //如果requestCode大于等于0，也就是需要返回结果给调用者Activity
            //并且调用者Activity没有在等待关闭的队列中，说明sourceRecord就是接受结果的Activity
            if (sourceRecord != null) {
                if (requestCode >= 0 && !sourceRecord.finishing) {
                    // 将调用者的Activity作为接受结果的Activity
                    resultRecord = sourceRecord;
                }
            }
        }

        final int launchFlags = intent.getFlags();

        if ((launchFlags & Intent.FLAG_ACTIVITY_FORWARD_RESULT) != 0 && sourceRecord != null) {
            // Transfer the result target from the source activity to the new
            // one being started, including any failures.
            if (requestCode >= 0) {
                ActivityOptions.abort(options);
                return ActivityManager.START_FORWARD_AND_REQUEST_CONFLICT;
            }
            resultRecord = sourceRecord.resultTo;
            if (resultRecord != null && !resultRecord.isInStackLocked()) {
                resultRecord = null;
            }
            resultWho = sourceRecord.resultWho;
            requestCode = sourceRecord.requestCode;
            sourceRecord.resultTo = null;
            if (resultRecord != null) {
                resultRecord.removeResultsLocked(sourceRecord, resultWho, requestCode);
            }
            if (sourceRecord.launchedFromUid == callingUid) {
                // The new activity is being launched from the same uid as the previous
                // activity in the flow, and asking to forward its result back to the
                // previous.  In this case the activity is serving as a trampoline between
                // the two, so we also want to update its launchedFromPackage to be the
                // same as the previous activity.  Note that this is safe, since we know
                // these two packages come from the same uid; the caller could just as
                // well have supplied that same package name itself.  This specifially
                // deals with the case of an intent picker/chooser being launched in the app
                // flow to redirect to an activity picked by the user, where we want the final
                // activity to consider it to have been launched by the previous app activity.
                callingPackage = sourceRecord.launchedFromPackage;
            }
        }

        if (err == ActivityManager.START_SUCCESS && intent.getComponent() == null) {
            // We couldn't find a class that can handle the given Intent.
            // That's the end of that!
            err = ActivityManager.START_INTENT_NOT_RESOLVED;
        }

        if (err == ActivityManager.START_SUCCESS && aInfo == null) {
            // We couldn't find the specific class specified in the Intent.
            // Also the end of the line.
            err = ActivityManager.START_CLASS_NOT_FOUND;
        }

        if (err == ActivityManager.START_SUCCESS && sourceRecord != null
                && sourceRecord.task.voiceSession != null) {
            // If this activity is being launched as part of a voice session, we need
            // to ensure that it is safe to do so.  If the upcoming activity will also
            // be part of the voice session, we can only launch it if it has explicitly
            // said it supports the VOICE category, or it is a part of the calling app.
            if ((launchFlags & FLAG_ACTIVITY_NEW_TASK) == 0
                    && sourceRecord.info.applicationInfo.uid != aInfo.applicationInfo.uid) {
                try {
                    intent.addCategory(Intent.CATEGORY_VOICE);
                    if (!AppGlobals.getPackageManager().activitySupportsIntent(
                            intent.getComponent(), intent, resolvedType)) {
                        Slog.w(TAG,
                                "Activity being started in current voice task does not support voice: "
                                        + intent);
                        err = ActivityManager.START_NOT_VOICE_COMPATIBLE;
                    }
                } catch (RemoteException e) {
                    Slog.w(TAG, "Failure checking voice capabilities", e);
                    err = ActivityManager.START_NOT_VOICE_COMPATIBLE;
                }
            }
        }

        if (err == ActivityManager.START_SUCCESS && voiceSession != null) {
            // If the caller is starting a new voice session, just make sure the target
            // is actually allowing it to run this way.
            try {
                if (!AppGlobals.getPackageManager().activitySupportsIntent(intent.getComponent(),
                        intent, resolvedType)) {
                    Slog.w(TAG,
                            "Activity being started in new voice task does not support: "
                                    + intent);
                    err = ActivityManager.START_NOT_VOICE_COMPATIBLE;
                }
            } catch (RemoteException e) {
                Slog.w(TAG, "Failure checking voice capabilities", e);
                err = ActivityManager.START_NOT_VOICE_COMPATIBLE;
            }
        }

        final ActivityStack resultStack = resultRecord == null ? null : resultRecord.task.stack;

        if (err != START_SUCCESS) {
            if (resultRecord != null) {
                resultStack.sendActivityResultLocked(
                        -1, resultRecord, resultWho, requestCode, RESULT_CANCELED, null);
            }
            ActivityOptions.abort(options);
            return err;
        }

        boolean abort = !mSupervisor.checkStartAnyActivityPermission(intent, aInfo, resultWho,
                requestCode, callingPid, callingUid, callingPackage, ignoreTargetSecurity, callerApp,
                resultRecord, resultStack, options);
        abort |= !mService.mIntentFirewall.checkStartActivity(intent, callingUid,
                callingPid, resolvedType, aInfo.applicationInfo);

        if (mService.mController != null) {
            try {
                // The Intent we give to the watcher has the extra data
                // stripped off, since it can contain private information.
                Intent watchIntent = intent.cloneFilter();
                abort |= !mService.mController.activityStarting(watchIntent,
                        aInfo.applicationInfo.packageName);
            } catch (RemoteException e) {
                mService.mController = null;
            }
        }

        mInterceptor.setStates(userId, realCallingPid, realCallingUid, startFlags, callingPackage);
        mInterceptor.intercept(intent, rInfo, aInfo, resolvedType, inTask, callingPid, callingUid,
                options);
        intent = mInterceptor.mIntent;
        rInfo = mInterceptor.mRInfo;
        aInfo = mInterceptor.mAInfo;
        resolvedType = mInterceptor.mResolvedType;
        inTask = mInterceptor.mInTask;
        callingPid = mInterceptor.mCallingPid;
        callingUid = mInterceptor.mCallingUid;
        options = mInterceptor.mActivityOptions;
        if (abort) {
            if (resultRecord != null) {
                resultStack.sendActivityResultLocked(-1, resultRecord, resultWho, requestCode,
                        RESULT_CANCELED, null);
            }
            // We pretend to the caller that it was really started, but
            // they will just get a cancel result.
            ActivityOptions.abort(options);
            return START_SUCCESS;
        }

        // If permissions need a review before any of the app components can run, we
        // launch the review activity and pass a pending intent to start the activity
        // we are to launching now after the review is completed.
        if (Build.PERMISSIONS_REVIEW_REQUIRED && aInfo != null) {
            if (mService.getPackageManagerInternalLocked().isPermissionsReviewRequired(
                    aInfo.packageName, userId)) {
                IIntentSender target = mService.getIntentSenderLocked(
                        ActivityManager.INTENT_SENDER_ACTIVITY, callingPackage,
                        callingUid, userId, null, null, 0, new Intent[]{intent},
                        new String[]{resolvedType}, PendingIntent.FLAG_CANCEL_CURRENT
                                | PendingIntent.FLAG_ONE_SHOT, null);

                final int flags = intent.getFlags();
                Intent newIntent = new Intent(Intent.ACTION_REVIEW_PERMISSIONS);
                newIntent.setFlags(flags
                        | Intent.FLAG_ACTIVITY_EXCLUDE_FROM_RECENTS);
                newIntent.putExtra(Intent.EXTRA_PACKAGE_NAME, aInfo.packageName);
                newIntent.putExtra(Intent.EXTRA_INTENT, new IntentSender(target));
                if (resultRecord != null) {
                    newIntent.putExtra(Intent.EXTRA_RESULT_NEEDED, true);
                }
                intent = newIntent;

                resolvedType = null;
                callingUid = realCallingUid;
                callingPid = realCallingPid;

                rInfo = mSupervisor.resolveIntent(intent, resolvedType, userId);
                aInfo = mSupervisor.resolveActivity(intent, rInfo, startFlags,
                        null /*profilerInfo*/);

                if (DEBUG_PERMISSIONS_REVIEW) {
                    Slog.i(TAG, "START u" + userId + " {" + intent.toShortString(true, true,
                            true, false) + "} from uid " + callingUid + " on display "
                            + (container == null ? (mSupervisor.mFocusedStack == null ?
                            Display.DEFAULT_DISPLAY : mSupervisor.mFocusedStack.mDisplayId) :
                            (container.mActivityDisplay == null ? Display.DEFAULT_DISPLAY :
                                    container.mActivityDisplay.mDisplayId)));
                }
            }
        }

        // If we have an ephemeral app, abort the process of launching the resolved intent.
        // Instead, launch the ephemeral installer. Once the installer is finished, it
        // starts either the intent we resolved here [on install error] or the ephemeral
        // app [on install success].
        if (rInfo != null && rInfo.ephemeralResolveInfo != null) {
            intent = buildEphemeralInstallerIntent(intent, ephemeralIntent,
                    rInfo.ephemeralResolveInfo.getPackageName(), callingPackage, resolvedType,
                    userId);
            resolvedType = null;
            callingUid = realCallingUid;
            callingPid = realCallingPid;

            aInfo = mSupervisor.resolveActivity(intent, rInfo, startFlags, null /*profilerInfo*/);
        }

        ActivityRecord r = new ActivityRecord(mService, callerApp, callingUid, callingPackage,
                intent, resolvedType, aInfo, mService.mConfiguration, resultRecord, resultWho,
                requestCode, componentSpecified, voiceSession != null, mSupervisor, container,
                options, sourceRecord);
        if (outActivity != null) {
            outActivity[0] = r;
        }

        if (r.appTimeTracker == null && sourceRecord != null) {
            // If the caller didn't specify an explicit time tracker, we want to continue
            // tracking under any it has.
            r.appTimeTracker = sourceRecord.appTimeTracker;
        }

        final ActivityStack stack = mSupervisor.mFocusedStack;
        if (voiceSession == null && (stack.mResumedActivity == null
                || stack.mResumedActivity.info.applicationInfo.uid != callingUid)) {
            if (!mService.checkAppSwitchAllowedLocked(callingPid, callingUid,
                    realCallingPid, realCallingUid, "Activity start")) {
                PendingActivityLaunch pal =  new PendingActivityLaunch(r,
                        sourceRecord, startFlags, stack, callerApp);
                mPendingActivityLaunches.add(pal);
                ActivityOptions.abort(options);
                return ActivityManager.START_SWITCHES_CANCELED;
            }
        }

        if (mService.mDidAppSwitch) {
            // This is the second allowed switch since we stopped switches,
            // so now just generally allow switches.  Use case: user presses
            // home (switches disabled, switch to home, mDidAppSwitch now true);
            // user taps a home icon (coming from home so allowed, we hit here
            // and now allow anyone to switch again).
            mService.mAppSwitchesAllowedTime = 0;
        } else {
            mService.mDidAppSwitch = true;
        }

        doPendingActivityLaunchesLocked(false);

        try {
            mService.mWindowManager.deferSurfaceLayout();
            err = startActivityUnchecked(r, sourceRecord, voiceSession, voiceInteractor, startFlags,
                    true, options, inTask);
        } finally {
            mService.mWindowManager.continueSurfaceLayout();
        }
        postStartActivityUncheckedProcessing(r, err, stack.mStackId, mSourceRecord, mTargetStack);
        return err;
    }
```

