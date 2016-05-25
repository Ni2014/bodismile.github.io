---
layout: post
title: 判断App处于前台的最简方式
category: 移动开发
tags: android
keywords: android,Foreground
---

有时候我们需要知道当前应用程序处于前台还是后台，以此判断是否发送推送消息来提醒用户，尤其在即时通讯类的应用中。

之前的做法一般是定义观察者集合，然后在`BaseActivity`的`onResume`中`添加观察者`，在`onPause`方法中`移除观察者`，这种方式并不太准确。

这里介绍一种判断APP处于前台的最简方式：


大家可以看看这篇文章：[is my Android app currently foreground or background](http://steveliles.github.io/is_my_android_app_currently_foreground_or_background.html)

完整的`Foreground`类如下：

	/**
	 *检测程序是否处于前台/后台，用于判断是否显示通知栏，替换ObseverListener的一种更加简便的实现
	 */
	public class Foreground implements Application.ActivityLifecycleCallbacks {
	
		public static final long CHECK_DELAY = 500;
		public static final String TAG = Foreground.class.getName();
		private static Foreground instance;
		private boolean foreground = false, paused = true;
		private Handler handler = new Handler();
		private List<Listener> listeners = new CopyOnWriteArrayList<Listener>();
		private Runnable check;
	
		/**
		 * Its not strictly necessary to use this method - _usually_ invoking
		 * get with a Context gives us a path to retrieve the Application and
		 * initialise, but sometimes (e.g. in test harness) the ApplicationContext
		 * is != the Application, and the docs make no guarantees.
		 *
		 * @param application
		 * @return an initialised Foreground instance
		 */
		public static Foreground init(Application application){
			if (instance == null) {
				instance = new Foreground();
				application.registerActivityLifecycleCallbacks(instance);
			}
			return instance;
		}
	
		public static Foreground get(Application application){
			if (instance == null) {
				init(application);
			}
			return instance;
		}
	
		public static Foreground get(Context ctx){
			if (instance == null) {
				Context appCtx = ctx.getApplicationContext();
				if (appCtx instanceof Application) {
					init((Application)appCtx);
				}
				throw new IllegalStateException(
						"Foreground is not initialised and " +
								"cannot obtain the Application object");
			}
			return instance;
		}
	
		public static Foreground get(){
			if (instance == null) {
				throw new IllegalStateException(
						"Foreground is not initialised - invoke " +
								"at least once with parameterised init/get");
			}
			return instance;
		}
	
		public boolean isForeground(){
			return foreground;
		}
	
		public boolean isBackground(){
			return !foreground;
		}
	
		public void addListener(Listener listener){
			listeners.add(listener);
		}
	
		public void removeListener(Listener listener){
			listeners.remove(listener);
		}
	
		@Override
		public void onActivityResumed(Activity activity) {
			paused = false;
			boolean wasBackground = !foreground;
			foreground = true;
	
			if (check != null)
				handler.removeCallbacks(check);
	
			if (wasBackground){
				IMLogger.i(TAG, "went foreground");
				for (Listener l : listeners) {
					try {
						l.onBecameForeground();
					} catch (Exception exc) {
						IMLogger.e(TAG, "Listener threw exception!", exc);
					}
				}
			} else {
				IMLogger.i(TAG, "still foreground");
			}
		}
	
		@Override
		public void onActivityPaused(Activity activity) {
			paused = true;
			if (check != null)
				handler.removeCallbacks(check);
			handler.postDelayed(check = new Runnable(){
				@Override
				public void run() {
					if (foreground && paused) {
						foreground = false;
						IMLogger.i(TAG, "went background");
						for (Listener l : listeners) {
							try {
								l.onBecameBackground();
							} catch (Exception exc) {
								IMLogger.e(TAG, "Listener throw exception!", exc);
							}
						}
					} else {
						IMLogger.i(TAG, "still foreground");
					}
				}
			}, CHECK_DELAY);
		}
	
		@Override
		public void onActivityCreated(Activity activity, Bundle savedInstanceState) {}
	
		@Override
		public void onActivityStarted(Activity activity) {}
	
		@Override
		public void onActivityStopped(Activity activity) {}
	
		@Override
		public void onActivitySaveInstanceState(Activity activity, Bundle outState) {}
	
		@Override
		public void onActivityDestroyed(Activity activity) {}
	
	
		public interface Listener {
	
			public void onBecameForeground();
	
			public void onBecameBackground();
	
		}
	
	}

**注：[Foreground](https://gist.github.com/steveliles/11116937#file-foreground-java)用法如下：**

1. 在application里面进行初始化操作：

	Foreground.init((Application)appContext);

2. 通过如下方式获取当前处于前台或者后台：

	Foreground.get().isForeground()
	
	true-表示app处于前台状态
    false-表示处于后台状态，可以发送通知消息