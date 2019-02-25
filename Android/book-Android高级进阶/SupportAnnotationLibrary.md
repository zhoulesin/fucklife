## 1.Nullness注解

此注解包含如下内容

- @Nullable作用于函数参数或者返回值,标记参数或者返回值可以为空
- @NonNull作用域函数参数或者返回值,标记参数或者返回值不可以为空

当出现这种违反注解标记的代码时,As会给出提示,同时使用Android Lint进行静态代码扫描,也会显示出错提示.

2.资源类型注解

我们知道在Android中资源是以整型值表示的,并保存在R.java文件中.

这意味着一个需要传入layout资源值的函数,如果传入string资源值不会再编译期报错,只有在运行时执行到响应代码才能发现问题,使用资源类型注解可以防止这种情况

资源类型的注解作用于函数参数,返回值即类的白能量,

- AnimatorRes:标记整型值是android.R.animator类型
- AnimRes:android.R.anim
- AnyRes:任何一种资源类型
- ArrayRes:android.R.array
- AttrRes:android.R.attr
- BoolRes:整型值是布尔类型
- ColorRes;android.R.color
- DrawableRes:android.R.drawable
- FactionRes;标记是fraction类型
- IdRes:android.R.id
- IntegerRes:android.R.Integer
- InterpolatorRes:android.R.Interpolator
- LayoutRes:android.R.layout
- MenuRes:android.R.menu
- PluralsRes:android.R.plurals
- RawRes
- StringRes
- StyleableRes
- StyleRes
- TransitionRes
- XmlRes

## 3.类型定义注解

在Android开发中,整型值不止经常用来代表资源引用值,而且经常用来代替枚举值

@IntDef注解用来创建一个整型类型定义的新注解,我们可以使用这个新注解来标记自己编写的API.

```java
import android.support.annotation.IntDef;

public abstract class ActionBar{
    //告知编译器不要在.class文件中存储注解数据
    @Retention(RetentionPolicy.SOURCE)
    //定义可以接受的常量列表
    @IntDef({NAVIGATION__MODE_STANDARD,NAVIGATION_MODE_LIST,NAVIGATION_MODE_TABS})
    //定义NavigationMode注解
    public @interface NavigationMode{}
    
    //常量定义
    psfi NAVIGATION_MODE_STANDARD = 0;
    psfi NAVIGATION_MODE_LIST = 1;
    psfi NAVIGATION_MODE_TABS = 2;
    
    @Navigation
    public abstract int getNavigationMode();
    
    public abstract void setNavigationMode(@NavigationMode int mode);
    //...
    
}
```

在使用setNavigationMode时,如果传入的参数mode不是这三个常量之一,那么as就会给出警告

除了类似上面定义,我们可以定义一个flag表示位,来识别函数参数或者返回值是否符合某一种模式,语句如下.

```java
@IntDef(flag=true,value={
    NAVIGATION_MODE_STANDARD,
    NAVIGATION_MODE_LIST,
    NAVIGATION_MODE_TABS
})
@Retention(RetentionPolicy.SOURCE)
public @interface NavigationMode{}

@NavigationMode
public abstract int getNavigationMode();

public abstract void setNavigationMode(@NavigationMode int mode);

```

## 4.线程注解

​	Android应用开发过程中,经常会涉及到多线程的使用,界面相关操作必须在主线程,而耗时操作例如文件下载等则需要放到后台线程中.

- @UiThread:标记运行在UI线程,一个UI线程是Activity运行所在的主窗口,对于一个应用而言,可能存在多个UI线程,每个UI线程对应不同的主窗口
- @MainThread:标记运行在主线程,一个应用只有一个主线程,主线程也是@UiThread线程.通常情况下,我们使用@MainThread来注解生命周期函数,使用@UiThread来注解视图相关函数,
- @WorkerThread:标记运行在后台线程
- @BinderThread:标记运行在Binder线程

AsyncTask

```java
@MainThread
protected void onPreExecute(){}

@WorkerThread
protected abstract Result doInBackground(Params... params);

@MainThread
protected void onProgressUpdate(Progress... values);
```

## 5.RGB颜色值注解

在资源类型注解中我们使用@ColorRes来标记参数类型需要传入颜色类型的资源id,

@ColorInt注解则是标记参数类型需要传入RGB或者ARGB颜色整型值.在TextView的源码中

```java
public void setTextColor(@ColorInt int color){
    mTextColor = ColorStateList.valueOf(color);
    updateTextColors();
}
```

