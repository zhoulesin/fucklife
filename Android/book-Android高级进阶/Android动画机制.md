# Android动画机制

- 在Android 3.0之前,我们能使用的动画类型有两种,分别是逐帧动画和补间动画.
- 在Android 3.0发布,SDK又为开发者带来了更加强大灵活的属性动画.使得实现复杂的动画效果更加容易;
- 在Android 4.4中,SDK又带来了android.transition框架.

## 1.逐帧动画Frame Animation

一系列状态不断变化的图片,每一帧对应的图片和持续的时间.

### 1.1XML资源方式

将图片放到res/drawable中,在res/anim目录中新建动画xml文件

```xml
<animation-list xmlns:android="http://schemas.android.com/apk/res/android"
                android:oneshot="false">
    <item
          android:drawable="@drawable/common_loading_01"
          android:duration="120"/>
     <item
          android:drawable="@drawable/common_loading_02"
          android:duration="120"/>
     <item
          android:drawable="@drawable/common_loading_03"
          android:duration="120"/>
</animation-list>
```

其中android.oneshot控制动画是否循环播放  true表示不循环

android:duration指定每一帧的播放持续时间

### 1.2代码方式

在代码中定义逐帧动画

```java
AnimationDrawable animDrawable = new AnimationDrawable();
for(int i=1;i<5;i++){
    int id = getResource().getIdentifier("common_loading_"+i,"drawable",getPackageName());
    Drawable drawable = getResources().getDrawable(id);
    animDrawable.addFrame(drawable,120);
}
imageView.setBackgroundDrawable(animDrawable);
animDrawable.setOneShot(false);
```

定义好逐帧动画之后,可以在符合某个条件时触发或者停止动画的播放.

```java
//获取AnimationDrawable 对象实例,用来控制动画的播放和停止
AnimationDrawable animDrawable = (AnimationDrawable)imageView.getBackground();

animDrawable.start();

animDrawable.stop();
```

- 逐帧动画就是讲imageView的background设置为AnimationDrawable,再对animationDrawable进行动画的操作

## 2.补间动画Tween Animation

补间动画是指开发者无需定义动画过程中的每一帧,只需要定义动画的开始和结束两个关键帧,并制定动画的变化时间和方式等,然后交由Android系统进行计算,通过在两个关键帧之间插入渐变值来实现平滑过渡,从而对view的内容完成一系列的图形变换来实现动画效果.

- 透明度alpha
- 大小scale
- 位移translate
- 旋转rotate

这四种效果可以动态组合,从而实现复杂灵活的动画.

### 2.1插值器interpolator

前面说到Android系统会在补间动画的开始和结束关键帧之间插入渐变值,它依据的便是interpolator,具体来说,interpolator会根据类型的不同,选择不同的算法计算出在补间动画期间所需要动态插入帧的密度和位置,interpolator负责控制动画的变化速度,使得前面所说的四种基本动画效果都能够以匀速,加速,减速,抛物线等多种速度进行变化.

interpolator类其实是一个空接口,他继承自TimeInterpolator,timeinterpolator时间插值器允许动画进行非线性运动变换,如加速和减速等.改接口中只有float getInterpolator(float input)这个方法

```java
public interface Interpolator extends TimeInterpolator{
    
}

public interface TimeInterpolator{
    float getInterpolation(float input);
}
```

Android提供了几个interpolator的实现

| 插值器类型                       | 功能说明                                      |
| -------------------------------- | --------------------------------------------- |
| AccelerateDecelerateInterpolator | 在动画开始与结束时候速率改变慢,在中间加速     |
| AccelerateInterpolator           | 在动画开始速率慢,然后开始加速                 |
| AniticipateInterpolator          | 动画开始先向后,然后向前                       |
| AniticipateOvershootInterpolator | 动画开始先向后,到终点再向前甩一定值再返回终点 |
| BounceInterpolator               | 动画结束时候弹起                              |
| CycleInterpolator                | 动画循环播放特定的次数,速率的改变遵循正弦曲线 |
| DecelerateInterpolator           | 在动画开始的地方速率改变快,然后开始变慢       |
| LinearInterpolator               | 动画以常量速率进行改变                        |
| OvershootInterpolator            | 到终点再向前甩一定值再返回终点                |
| PathInterpolator                 | 通过定义路径坐标,动画可以按照路径坐标来运行,  |

- 也可以通过实现Interpolator接口来编写自己的插值器

Android SDK使用Animation类来表示抽象的动画类,

| 动画类型           | 功能说明                              |
| ------------------ | ------------------------------------- |
| AlphaAnimation     | 改变透明度,时间                       |
| ScaleAnimation     | 改变大小,指定缩放比,持续时间,缩放中心 |
| TranslateAnimation | 改变位置,指定开始结束位置,时间        |
| RotateAnimation    | 旋转,指定角度,旋转中心点,时间         |

### 2.2 AlphaAnimatino

xml方式

```xml
<set xmlns:android="http://schemas.android.com/apk/res/android"
     android:itnerpolator="@android:anim/accelerate_decelerate_interpolator">
	<translate
               android:duration="200"
               android:fromYDelta="100%p"
               android:toYDelta="0"/>

<alpha
               android:duration="200"
               android:fromAlpha="0.0"
               android:toAlpha="1.0"/>

</set>
```

AlphaAnimation构造函数只有两个参数

```java
public AlphaAnimation(float fromAlpha,float toAlpha){
    mFromAlpha = fromAlpha;
    mToAlpha = toAlpha;
}
```

在代码中实现透明度动画

```java
public void alpha(){
    AlphaAnimation anim = new AlphaAnimation(0,1);
    anim.setDuration(150);
    anim.setFillAfter(true);	//动画结束保留结束状态
    mImageView.setAnimation(anim);
}
```

### 2.3ScaleAnimation

xml实现

```xml
<set xmlns:android="http://schemas.android.com/apk/res/android"
     android:interpolator="@android:anim/accelerate_interpolator">
	<scale
           android:duration="2000"
           android:fromXScale="0.2"
           android:toXScale="1.5"
           android:fromYScale="0.2"
           android:toYScale="1.5"
           android:pivotX="50%p"
           android:pivotY="50%p"/>
</set>
```

代码实现中,scaleAnimation可用的构造函数有3个

```java
public ScaleAnimation(float fromX,float toX,float fromY,float toY){}

public ScaleAnimation(float fromX,float toX,float fromY,float toY,float pivotX,float pivotY){}

public ScaleAnimation(float fromX,float toX,float fromY,float toY,int pivotXType,float pivotXValue,int pivotYType,float pivotYValue){}
```

```java
public void scale(){
    ScaleAnimation anim = new ScaleAnimation(1.0f,4.0f,1.0f,4.0f,
                                            Animation.RELATIVE_TO_SELF,0.0f,
                                            Animation.RELATIVE_TO_SELF,0.0f);
    anim.setDuration(2000);
    anim.setFillAfter(true);
    imageView.startAnimation(anim);
}
```

### 2.4TranslateAnimatino

xml实现

```xml
<set xmlns:android="http://schemas.android.com/apk/res/android">
	<translate
               android:duration="200"
               android:fromXDelta="0"
               android:toXDelta="0"
                android:fromYDelta="0"
               android:toYDelta="1000"/>
</set>
```

代码实现

```java
public TranslateAnimation(float fromXDelta,float toXDelta,float fromYDelta,float toYDelta){}

public TranslateAnimation(int fromXType,float fromXValue,int toXType,float toXValue,
                         int fromYType,float fromYValue,int toYType,float toYValue){}
```

| 参数      | 含义说明                                                     |
| --------- | ------------------------------------------------------------ |
| fromXType | 动画开始在X轴 位移模式,Animation.ABSOLUTE,Animation.RELATIVE_TO_SELF,Animation.RELATIVE_TO_PARENT |
| ...       | ...                                                          |

translateAnimation使用

```java
public void translate(){
    TranslateAnimation anim = new TranslateAnimation(Animation.RELATIVE_TO_SELF,0f,
                                                    Animation.RELATIVE_TO_SELF,2f,
                                                    Animation.RELATIVE_TO_SELF,0f,
                                                    Animation.RELATIVE_TO_SELF,2f);
    anim.setDuration(3000);
    anim.setFillAfter(true);
    imageView.setAnimation(anim);
}
```

### 2.5RotateAnimation

xml实现

```xml
<set xmlns:android="http://schemas.android.com/apk/res/android">
	<rotate
            android:duration="1000"
            android:fromDegrees="0"
            android:pivotX="50%"
            android:pivotY="50%"
            android:startOffset="0"
            android:repeatCount="-1"
            android:repeatMode="restart"
            android:toDegrees="360"/>
</set>
```

在代码中

```java
public RotateAnimation(float fromDegrees,float toDegrees){}

public RotateAnimation(float fromDegrees,float toDegrees,float pivotX,float pivotY){}

public RotateAnimation(float fromDegrees,float toDegrees,int pivotXType,flaot pivotXValue,
                      int pivotYType,float pivotYValue){}
```

使用

```java
public void rotate(){
    RotateAnimation anim = new RotateAnimation(0,-720,
                                              RotateAnimation.RELATIVE_TO_SELF,0.5f,
                                              RotateAnimation.RELATIVE_TO_SELF,0.5f);
    anim.setDuration(200);
    anim.setFillAfter(true);
    imageView.startAnimation(anim);
}
```

### 2.6自定义补间动画

在实际项目中,往往遇到上述四种动画无法实现的动画需求,

继承Animation,applyTransformation.

```java
public class MyAnimation extends Animation{
    @Override
    protected void applyTransformation(float interpolatedTime,Transformation transformation){
        super.applyTransformation(interpolatedTime,transformation);
    }
}
```

其中interpolatedTime表示动画的时间进行比,无论动画的实际持续时间是多少,这个参数都会从0到1,transformation表示补间动画在不同时刻对view的变形程度.

