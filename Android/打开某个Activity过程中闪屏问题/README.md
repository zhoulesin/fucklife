# 打开Activity过程中闪屏问题

## 问题介绍

​	app运行中,打开某个activity,会出现短暂闪现白屏现象.

## 问题分析

​	由于activity 的打开关闭添加了过度动画,所以activity的主题背景色会产生一定影响,在动画过程中,屏幕会默认显示activity的主题背景色

## 问题解决

​	将主题变为默认黑色,并添加windowIsTranslucent="true"属性

​	//old

```xml
<style name="AppBaseTheme" parent="Theme.AppCompat.Light.NoActionBar">
	
</style>
```

​	//new

```xml
<style name="AppBaseTheme" parent="Theme.AppCompat.NoActionBar">
	<item name="android:windowIsTranslucent">true</item>
</style>
```

