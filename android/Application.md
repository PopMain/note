



#### Application类初始化流程

2021-03-23 10:06:29.383 21153-21153/com.zzzzzx.app W/WZX: java.lang.Throwable: application attachBaseContext
        at com.zzzzzx.app.util.SLog.logStack(SLog.java:10)
        at com.zzzzzx.app.ZzzzxApplication.attachBaseContext(ZzzzxApplication.kt:17)
        at android.app.Application.attach(Application.java:376)
        at android.app.Instrumentation.newApplication(Instrumentation.java:1156)
        at android.app.LoadedApk.makeApplication(LoadedApk.java:1222)
        at android.app.ActivityThread.handleBindApplication(ActivityThread.java:6572)
        at android.app.ActivityThread.access$1500(ActivityThread.java:233)
        at android.app.ActivityThread$H.handleMessage(ActivityThread.java:1914)
        at android.os.Handler.dispatchMessage(Handler.java:107)
        at android.os.Looper.loop(Looper.java:224)
        at android.app.ActivityThread.main(ActivityThread.java:7561)
        at java.lang.reflect.Method.invoke(Native Method)
        at com.android.internal.os.RuntimeInit$MethodAndArgsCaller.run(RuntimeInit.java:539)
        at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:995)
2021-03-23 10:06:29.414 21153-21153/com.zzzzzx.app W/WZX: java.lang.Throwable: application onCreate
        at com.zzzzzx.app.util.SLog.logStack(SLog.java:10)
        at com.zzzzzx.app.ZzzzxApplication.onCreate(ZzzzxApplication.kt:12)
        at android.app.Instrumentation.callApplicationOnCreate(Instrumentation.java:1189)
        at android.app.ActivityThread.handleBindApplication(ActivityThread.java:6601)
        at android.app.ActivityThread.access$1500(ActivityThread.java:233)
        at android.app.ActivityThread$H.handleMessage(ActivityThread.java:1914)
        at android.os.Handler.dispatchMessage(Handler.java:107)
        at android.os.Looper.loop(Looper.java:224)
        at android.app.ActivityThread.main(ActivityThread.java:7561)
        at java.lang.reflect.Method.invoke(Native Method)
        at com.android.internal.os.RuntimeInit$MethodAndArgsCaller.run(RuntimeInit.java:539)
        at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:995)



```java
private void handleBindApplication(AppBindData data) {
    ...
    final InstrumentationInfo ii;
    ...
    // 创建 mInstrumentation 实例
    if (ii != null) {
        ...
        //创建ContextImpl
        final ContextImpl instrContext = ContextImpl.createAppContext(this, pi);

        try {
            //创建mInstrumentation实例
            final ClassLoader cl = instrContext.getClassLoader();
            mInstrumentation = (Instrumentation)
                cl.loadClass(data.instrumentationName.getClassName()).newInstance();
        } catch (Exception e) {
            ...
        }
        ...
    } else {
        mInstrumentation = new Instrumentation();
    }
    ...
    Application app;
    ...
    try {
        ...
        // 创建 Application 实例
        app = data.info.makeApplication(data.restrictedBackupMode, null);
        mInitialApplication = app;
         if (!data.restrictedBackupMode) {
                if (!ArrayUtils.isEmpty(data.providers)) {
                    //启动ContentProvider
                    installContentProviders(app, data.providers);
                    mH.sendEmptyMessageDelayed(H.ENABLE_JIT, 10*1000);
                }
            }
        try {
            //调用Application的onCreate
            mInstrumentation.callApplicationOnCreate(app);
        } catch (Exception e) {
            ...
        }
    } finally {
        ...
    }
    ...
}

```



```java
 public Application newApplication(ClassLoader cl, String className, Context context)
            throws InstantiationException, IllegalAccessException, 
            ClassNotFoundException {
        Application app = getFactory(context.getPackageName())
                .instantiateApplication(cl, className);
        app.attach(context);
        return app;
    }
```



ActivityThread -> main()  -> attach() -> bindApplication(IPC调用通知ActivityManager绑定，返回AppBindData) -> ApplicaitonThread(系统进程IPC调用到App进程) -> handleBindApplicaiton -> LoadedApk.makeApplication(初始化Applicaiton实例) -> Instrumentation.newApplication(调用了Applicaiton.attach,attach中调用attachBaseContext) -> Instrumentation.callApplicationOnCreate(调用Application.onCreate)

执行完 **ApplicationThread# handleBindApplication ()** 之后，这时候新进程已经启动。
回到AMS# attachApplicationLocked(),代码逻辑来到了上篇文章的注释5处，调用了ActivityStackSupervisor# attachApplicationLocked,在里面又执行了realStartActivityLocked()