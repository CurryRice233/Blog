---
title: Espresso 检测 Toast 消息
date: 2019-04-14 20:26:55
categories: Espresso
tags: 
	- Espresso
	- Android
---

[Espresso](https://developer.android.com/training/testing/espresso) 是一个 Google 官方提供的 Android 应用 UI 自动化测试框架。Google 希望，当 Android 的开发者利用 Espresso 写完测试用例后，能一边看着测试用例自动执行，一边享受一杯香醇的 Espresso (浓咖啡)。

在使用 Espresso 的时候，我们经常会需要检测 Toast 消息。谷歌第一条就是以下代码：
```java
onView(withText(R.string.TOAST_STRING)).inRoot(withDecorView(not(is(getActivity().getWindow().getDecorView())))).check(matches(isDisplayed()));
```
如果这行代码对你有用的话，那么恭喜，下面的内容你不用看了。

我遇到的情况十分的神奇，在用了上面的代码后，第一次检测能行，但是第二次的时候就怎么都检测不了了。
在一通谷歌之后，找到了解决方法。
需要自己写一个 `ToastMatcher`
```java
public class ToastMatcher extends TypeSafeMatcher<Root> {

	@Override
	public void describeTo(Description description) {
		description.appendText("is toast");
	}

	@Override
	public boolean matchesSafely(Root root) {
		int type = root.getWindowLayoutParams().get().type;
		if ((type == WindowManager.LayoutParams.TYPE_TOAST)) {
			IBinder windowToken = root.getDecorView().getWindowToken();
			IBinder appToken = root.getDecorView().getApplicationWindowToken();
			if (windowToken == appToken) {
				return true;
			}
		}
		return false;
	}
}
```

Espresso Test 里
```java
onView(withText(textId)).inRoot(new ToastMatcher()).check(matches(isDisplayed()));
```
这样就完美解决了我遇到的问题。