# AndroidView的绘制流程

Android中Activity是作为应用程序的载体存在的,他代表这一个完整的用户界面,提供了一个窗口来绘制各种视图,当Activity启动时,我们会通过setContentView 方法来设置一个内容视图,这个内容视图就是用户看到的界面.内容视图就是以ViewGroup的形式存在的.

## 1.绘制的整体流程

当一个应用启动时,会启动一个主Activity,Android系统会根据Activity的布局来对他进行绘制.绘制会从跟视图ViewRoot的performTraversals方法开始,从上到下遍历整个视图树,每个View控件负责绘制自己,而ViewGroup还需要负责通知自己的子View进行绘制操作.视图绘制的过程可以分为三个步骤,分别是测量,布局和绘制.

performTraversals

```java
private void preformTraversals(){
    //...
    int childWidthMeasureSpec = getRootMeasureSpec(mWidth,lp.width);
    int childHeightMeasureSpec = getRootMeasureSpec(mHeight,lp.height);
    //...
    //执行测量流程
    performMeasure(childWidthMeasureSpec,childHeightMeasureSpec);
    //...
    //执行布局流程
    performLayout(lp,desiredWindowWidth,desiredWindowHeight);
    //...
    //执行绘制流程
    performDraw();
}
```

## 2.MeasureSpec

MeasureSpec表示的是一个32位的整型值,它的高2位表示测量模式SpecMode,低30位表示某种测量模式下的规格大小SpecSize.MeasureSpec是View类的一个静态内部类,用来说明应该如何测量这个View.

```java
public static class MeasureSpec{
    private static final int MODE_SHIFT = 30;
    private static final int MODE_TASK = 0x3 << MODE_SHIFT;
    
 	//不指定测量模式
    public static final int UNSPECIFIED = 0 << MODE_SHIFT;
    
    //精确测量模式
    public static final int EXACTLY = 1 << MODE_SHIFT;
    
    //最大值测量模式
    public static final int AT_MOST = 2 <<MODE_SHIFT;
    
    //根据指定的大小和模式创建一个MeasureSpec
    public static int makeMeasureSpec(int size,int mode){
        if(sUseBrokenMakeMeasureSpec){
            return size + mode;
        } else {
          	return (size & ~MODE_MASK) | (mode & MODE_MASK);  
        }
    }
    
    //微调某个MeasureSpec的大小
    static int adjust(int measureSpec,int delta){
        final int mode = getMode(measureSpec);
        if(mode == UNSPECIFIED){
            //NO need to adjust size for UNSPECIFIED mode.
            return makeMeasureSpec(0,UNSPECIFIED);
        }
        int size = getSize(measureSpec) + delta;
        if(size < 0) {
            Log.e(VIEW_LOG_TAG,"MeasureSpec.adjust: new size would be negative! ("
                 + size + ") spec :" + toString(measureSpec) + " delta: "
                 + delta);
            size = 0;
        }
        return makeMeasureSpec(size,mode);
    }
}
```

我们需要重点关注以下三种测量模式

- UNSPECIFIED:不指定测量模式,父视图没有限制子视图的大小,子视图可以是想要的任何尺寸,通常用于系统内部,开发中很少使用到
- EXACTLY:精确测量模式,当视图layout_width或者layout_height指定为具体数值或者match_parent时生效,表示父视图已经决定了子视图的精确大小,这种模式下View的测量值就是SpecSize的值
- AT_MOST:最大值模式,当该视图的layout_width或者layout_height指定为wrap_content时生效,此时子视图的尺寸可以是不超过父视图允许的最大尺寸的任何尺寸.

对DecorView而言,它的MeasureSpec由窗口尺寸和其自身的LayoutParams共同决定;对于普通的View,它的MeasureSpec由父视图的MeasureSpec和其自身的LayoutParams共同决定

## 3.Measure

Measure操作用来计算View的实际大小,页面的测量流程是从performMeasure方法开始的,

```java
private void performMeasure(int childWidthMeasureSpec,int childHeightMeasureSpec){
    //...
    mView.measure(childWidthMeasureSpec,childHeightMeasureSpec);
    //...
}
```

可以看到,具体的测量操作是分发给ViewGroup 的,由ViewGroup在它的measureChild方法中传递给子view.ViewGroup通过遍历自身所有的子View,并组个调用子View的measure方法实现测量操作.

```java
//遍历测量ViewGroup中所有的View
protected void measureChildren(int widthMeasureSpec,int heightMeasureSpec){
    final int size = mChildrenCount;
    final View[] children = mChildren;
    for(int i=0;i<size;i++){
        final View child = children[i];
        //当View的可见性处于GONE状态时,不对其进行测量
        if((child.mViewFlags & VISIBILITY_MASK) != GONE){
            measureChild(child,widthMeasureSpec,heightMeasureSpec);
        }
    }
}

//测量某个指定的view
protected void measureChild(View child,int parentWidthMeasureSpec,int parentHeightMeasureSpec){
    final LayoutParams lp = child.getLayoutParams();
    
    //根据父容器的MeasureSpec和子view的LayoutParams等信息计算子View的MeasureSpec
    final int childWidthMeasureSpec = getChildMeasureSpec(parentWidthMeasureSpec,
                                               mPaddingLeft + mPaddingRight,lp.width);
    final int childHeightMeasureSpec = getchildMeasureSpec(parentHeightMeasureSpec,
                                               mPaddingTop + mPaddingBottom,lp.height);
    child.measure(childWidthMeasureSpec,childHeightMeasureSpec);
}
```

View(ViewGroup)的measure方法,最终的测量是通过回调onMeasure方法实现的,这个通常由View的特定子类自己实现,开发者也可通过重写这个方法实现自定义View

```java
public final void measure(int widthMeasureSpec,int heightMeasureSpec){
    //..
    onMeasure(widthMeasureSpec,heightMeasureSpec);
    //..
}

//如果需要自定义测量过程,则子类可以重写这个方法
protected void onMeasure(int widthMeasureSpec,int heightMeasureSpec){
    //setMeasureDimension方法用于设置View的测量宽高
    setMeasureDimension(getDefaultSize(getSuggestedMinimumWidth(),widthMeasureSpec),
                       getDefaultSize(getSuggestedMinimumHeight(),heightMeasureSpec));
}

//如果View没有重写onMeasure方法,则默认会直接调用getDefaultSize来获得View的宽高
public static int getDefaultSize(int size,int measureSpec){
    int result = size;
    int specMode = MeasureSpec.getMode(measureSpec);
    int specSize = MeasureSpec.getSize(measureSpec);
    
    switch(specMode){
        case MeasureSpec.UNSPECIFIED:
            result = size;
            break;
        case MeasureSpec.AT_MOST:
        case MeasureSpec.EXACTLY:
            result = specSize;
           	break;
    }
    return result;
}
```

## 4.Layout

Layout过程用来确定View子父容器中的布局位置,它是由父容器获取子View的位置参数后,调用子View的layout方法并将位置参数传入实现的,ViewRootImpl的performLayout

```java
//ViewRootImpl.java
private void performLayout(WindowManager.LayoutParams lp,int desiredWindowWidth,
                           int desiredWindowHeight){
    //...
    host.layout(0,0,host.getMeasureWidth(),host.getMeasureHeight());
    //...
}

//View.java
public void layout(int l,int t,int r,int b){
    //...
    onLayout(changed,l,t,r,b);
    //...
}

//空方法,子类如果是ViewGroup类型,则重写这个方法,实现ViewGroup中所有View空间布局流程
protected void onLayout(boolean changed,int left,int top,int right,int bottom){
    
}

```

## 5.Draw

Draw操作用来将控件绘制出来,绘制的流程从performDraw方法开始

```java
private performDraw(){
    //...
    draw(fullRedrawNeeded);
    //..
}

private void draw(boolean fullRedrawNeeded){
    //...
    if(!drawSoftware(surface,mAttachInfo,xOffset,yOffset,scalingRequired,dirty)){
        return;
    }
    //...
}

private boolean drawSoftware(Surface surface,AttachInfo attachInfo,int xoff,int yoff,
                            boolean scalingRequired,Rect dirty){
    //...
    mView.draw(canvas);
    //...
}
```

可以看到最终调用每个View的draw方法绘制每个具体的View,绘制基本上可以分为六个步骤

```java
public void draw(Canvas canvas){
    //...
    //步骤1,绘制View的背景
    drawBackground(canvas);
    
    //...
    //步骤2,如果需要的话,保存canvas的图层,为fading做准备
    saveCount = canvas.getSaveCount();
    
    //...
    canvas.saveLayer(left,top,right,top+length,null,flags);
    //...
    //步骤3.绘制View 的内容
    onDraw(canvas);
    
    //步骤4,绘制View的子View
    dispatchDraw(canvas);
    
    //步骤5.如果需要的话,绘制View的fading边缘并恢复图层
    canvas.drawRect(left,top,right,top+length,p);
    
    canvas.restoreToCount(saveCount);
    
    //步骤6.绘制View的装饰(如滚动条)
    onDrawScrollBars(canvas);
    
}
```

