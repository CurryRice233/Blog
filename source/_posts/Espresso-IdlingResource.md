---
title: Espresso 等待外部资源
date: 2019-04-14 19:23:05
categories: Espresso
tags: 
	- Espresso
	- Android
	- Firebase
---

[Espresso](https://developer.android.com/training/testing/espresso) 是一个 Google 官方提供的 Android 应用 UI 自动化测试框架。Google 希望，当 Android 的开发者利用 Espresso 写完测试用例后，能一边看着测试用例自动执行，一边享受一杯香醇的 Espresso (浓咖啡)。

在使用 Espresso 的时候，很多场景下我们需要暂停 Espresso 进程，等待应用获取一个外部资源。可能是一张图片，一段文字。我的情况是应用需要在 [Firebase](https://firebase.google.com/) 获取一个对象。需要在 Espresso 进行到一半的时候暂停它，然后应用向 Firebase 发送请求，等待响应，之后再继续 Espresso 测试。
Espresso 提供了一个 `IdlingResource` 的接口来解决上面的问题。让我们来实现它:
```java
public class SimpleIdlingResource implements IdlingResource {
	@Nullable
	private volatile ResourceCallback mCallback;
	// Idleness is controlled with this boolean.
	private AtomicBoolean mIsIdleNow = new AtomicBoolean(true);
	@Override
	public String getName() {
		return this.getClass().getName();
	}
	@Override
	public boolean isIdleNow() {
		return mIsIdleNow.get();
	}
	@Override
	public void registerIdleTransitionCallback(ResourceCallback callback) {
		mCallback = callback;
	}
	/**
	 * Sets the new idle state, if isIdleNow is true, it pings the {@link ResourceCallback}.
	 * @param isIdleNow false if there are pending operations, true if idle.
	 */
	public void setIdleState(boolean isIdleNow) {
		mIsIdleNow.set(isIdleNow);
		if (isIdleNow && mCallback != null) {
			mCallback.onTransitionToIdle();
		}
	}
}
```
然后在 Activity 里声明：
```java
public class MainActivity extends AppCompatActivity {
	public static SimpleIdlingResource mIdlingResource;
	
	@VisibleForTesting
	public SimpleIdlingResource getIdlingResource() {
		if(mIdlingResource == null)
			mIdlingResource = new SimpleIdlingResource();
		return mIdlingResource;
	}
}
```
之后在你的代码准备获取外部资源前加入
```java
if(MainActivity.mIdlingResource!=null)
	MainActivity.mIdlingResource.setIdleState(false);
```
来暂停 Espresso，等获取到了外部资源之后再 `setIdleState(true)` Espresso 就会继续执行了。

最后一步就是在 EspressoTest 里注册了
```java
@RunWith(AndroidJUnit4.class)
@LargeTest
public class EspressoTest {
	private IdlingResource idlingResource;

	@Rule
	public ActivityTestRule<MainActivity> mActivityRule = new ActivityTestRule<>(MainActivity.class);
	
	@Before
	public void registerActivity(){
		idlingResource = mActivityRule.getActivity().getIdlingResource();
		IdlingRegistry.getInstance().register(idlingResource);
	}
	@After
	public void unregisterActivity(){
		IdlingRegistry.getInstance().unregister(idlingResource);
	}
	
	@Test
	public void test(){
		// your test
	}
}
```

