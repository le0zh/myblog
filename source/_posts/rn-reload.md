title: ReactNative ReloadJs的实现
tags:
  - react-native
categories:
  - javascript
date: 2016-11-21 14:28:12
---

#### 写在前面

RN的Dev菜单有一个Reload，我们经常使用，效果是刷新当前app的js代码，类似于Web中的“刷新”或者"F5”。
这个特性其实可以拿来做热更新：

![img](http://7xl2vf.com1.z0.glb.clouddn.com//blog/reload-update.png)

<!--more-->

#### 代码实现

在检测到有新版本的jsbundle后，调用relaodjs方法，刷新页面（将该工作放在app启动后主UI显示前，可避免用户丢失当前state）
在走读了RN代码和网上搜索后，总结了方法的实现如下（已经过测试）：
```

public void reloadJs() {
    UiThreadUtil.runOnUiThread(new Runnable() {
        @Override
        public void run() {
            try {
                Activity activity = getCurrentActivity();
                Application application = activity.getApplication();
                ReactInstanceManager instanceManager = ((ReactApplication) application).getReactNativeHost().getReactInstanceManager();
 
                if (instanceManager.getClass().getSimpleName().equals("XReactInstanceManagerImpl")) {
                    // DEBUG
                    JSBundleLoader loader = JSBundleLoader.createRemoteDebuggerBundleLoader(
                            instanceManager.getDevSupportManager().getJSBundleURLForRemoteDebugging(),
                            instanceManager.getDevSupportManager().getSourceUrl()
                    );
 
                    // LOAD FROM FILE
                    // JSBundleLoader loader = JSBundleLoader.createFileLoader(application, UpdateContext.getBundleUrl(application));
 
                    Field jsBundleField = instanceManager.getClass().getDeclaredField("mBundleLoader");
                    jsBundleField.setAccessible(true);
                    jsBundleField.set(instanceManager, loader);
                } else {
                    Field jsBundleField = instanceManager.getClass().getDeclaredField("mJSBundleFile");
                    jsBundleField.setAccessible(true);
                    jsBundleField.set(instanceManager, instanceManager.getDevSupportManager().getSourceUrl());
 
                    // jsBundleField.set(instanceManager, UpdateContext.getBundleUrl(application));
                }
 
                final Method recreateMethod = instanceManager.getClass().getMethod("recreateReactContextInBackground");
 
                final ReactInstanceManager finalizedInstanceManager = instanceManager;
 
                recreateMethod.invoke(finalizedInstanceManager);
 
                activity.recreate();
            } catch (Throwable err) {
                Log.e("hot-update", "Failed to restart application", err);
            }
        }
    });
}
```

#### 一些说明

1. ReactInstanceManager中的 recreateReactContextInBackground 方法就是用来reload js的，但由于该方法是private的，只用用反射的方法调用了。

2. recreateReactContextInBackground方法必须运行在UI线程，所以使用了RN的UiThreadUtil.runOnUiThread方法

3. 通过JSBundleLoader.createFileLoader可以指定待加载的bundle文件，这里就可以是我们下载的最新的文件

4. 上面的DEBUG注释下的代码是为了测试reaload方法，使用开发用的bundle文件