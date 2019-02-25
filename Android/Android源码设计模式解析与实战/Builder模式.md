# Builder模式

## Builder模式介绍

​	Builder模式是一步一步创建一个复杂对象的创建型模式,它允许用户在不知道内部构建细节的情况下,可以更精细地控制对象的构造流程,该模式是为了将构建复杂对象的过程和它的部件解耦,使得构建过程和部件的表示隔离开来

​	因为一个复杂的对象有很多大量组成部分,如汽车,有车轮,方向盘,发动机还有各种小零件等,如何将这些部件装配成一辆汽车,这个装配过程很漫长,也很复杂,对于这种情况,为了在构建过程中对外部隐藏实现细节,就可以使用Builder模式将部件和组件过程分离,使得构建过程和部件都可以自由扩展,两者之间的耦合也降到最低

## Builder模式的定义

​	将一个复杂对象的构建与它的表示分离,使得同样的构建过程可以创建不同的表示

## Builder模式的使用场景

- 相同的方法,不同的执行顺序,产生不同的事件结果的
- 多个部件或零件,都可以装配到一个对象中,但是产生的运行结果又不相同
- 产品类非常复杂,或者产品类中的调用顺序不同产生了不同的作用,这个时候使用建造者模式非常合适
- 初始化一个对象特别复杂,如参数多,且很多参数都有默认值时

## Builder模式的uml类图

- Product产品类   ---  产品的抽象类
- Builder ----抽象Builder类,规范产品的组建,一般是由子类实现具体的组建过程
- ConcreteBuilder ---具体的Builder类
- Director --统一组装过程

## Builder模式简单实现

​	计算机的组装过程较为复杂,并且组装顺序是不固定的,为了易于理解,我们把计算机组装的过程简化为构建主机,设置操作系统,设置显示器3个部分,然后通过Director和具体的Builder来构建计算机对象

```java
public abstract class Computer{
    protected String mBoard;
    protected String mDisplay;
    protected String mOs;
    
    protected Computer{}
    public void setBoard(String board){
        mBoard = board;
    }
    public void setDisplay(String display){
        mDisplay = display;
    }
    public abstract void setOs();
}
```

​	具体的Computer类,Mac

```java
public class Mac extends Computer{
    protected Mac{}
    public void setOs(){
        mOs = "Mac os xxx";
    }
}
//抽象Builder类
public abstract class Builder{
    //设置主机
    public abstract void buildBoard(String board);
    public abstract void buildDisplay(String display);
    public abstract void buildOs();
    public abstract Computer create();
}
//具体的Builder类
public class MacBuilder extends Builder{
    private Computer mComputer = new Mac();
    public void buildBoard(String board){
        mComputer.setBoard(board);
    }
    public void buildDisplay(String display){
        mComputer.setDisplay(display);
    }
    public void buildOs(){
        mComputer.setOs();
    }
    public Computer create(){
        return mComputer;
    }
}
//Director类,负责构造Computer
public class Director{
    Builder mBuilder = null;
    
    public Director(Builder builder){
        mBuilder = builder;
    }
    
    public void construct(String board,String display){
        mBuilder.buildBoard(board);
        mBuilder.buildDisplay(display);
        mBuilder.buildOs();
    }
}
//测试代码
public class Test{
    public static void main(String[] args){
        Builder builder = new MacBuilder();
        Director director = new Director(builder);
        director.construct("inter","retina");
        
        syso(builder.create().toString());
    }
}
```

​	上述实例中,通过具体的MacBuilder构建Mac对象,而Director封装了构建复杂产品对象的过程,对外隐藏构建细节.Builder与Director一起将一个复杂对象的构建与它的表示分离,使得同样的构建过程可以创建不同的对象.

​	值得注意的是,在现实开发过程中,Director角色经常会被省略.而直接适用一个Buidler来进行对象的组装,这个Builder通常为链式调用,它的关键点是每个setter方法都返回自身,也就是return this,这样就使得setter方法可以链式调用

```java
new TestBuilder().setA('A').setB('B').create();
```

# Android中的Builder模式

​	在Android源码中,最常用到的Builder模式就是AlertDialog.Builder,适用该Builder来构建复杂的AlertDialog对象.在开发过程中,我们经常用到AlertDialog.

```java
private void showDialog(Context context){
    AlertDialog.Builder builder = new AlertDialog.Builder(context);
    builder.setIcon(R.drawable.icon);
    builder.setTitle("Title");
    builder.setMessage("Message");
    builder.setPositiveButton("Button1",new DialogInterface.OnClickListener(){
        public void onClick(DialogInterface dialog,int whichButton){
            syso("点击了Button1");
        }
    });
    builder.setNeutralButton("Button2",new DialogInterface.OnClickListener(){
        public void onClick(DialogInterface dialog,int whichButton){
            syso("点击了Button2")
        }
    });
    builder.setNegativeButton("Button3",new DialogInterface.OnClickListener(){
        public void onClick(DialogInterface dialog,int whichButton){
            syso("点击了Button3")
        }
    });
    builder.create().show();
}
```

​	从类名就可以看出这就是一个Builder模式,通过Builder对象来组装Dialog的各个部分,如title,buttons,message等,将Dialog的构造和表示进行分离.

```java
public class AlertDialog extends Dialog implementes DialogInterface{
    private AlertController mAlert;
    protected AlertDialog(Context context,int theme){
        this(context,theme,true);
    }
    AlertDialog(Context context,int theme,boolean createContextWrapper){
        super(context,resolveDialogTheme(context,theme),createContextWrapper);
        mWindow.alwaysReadCloseOnTouchAttr();
        
        mAlert = new AlertController(getContext(),this,getWindow());
    }
    
    public void setTitle(CharSequence title){
        super.setTitle(title);
        mAlert.setTitle(title);
    }
    
    public void setCustomTitle(View customTitleView){
        mAlert.setCustomTitle(customTitleView);
    }
    
    public void setMessage(CharSequence message){
        mAlert.setMessage(message);
    }
    
    //.....
    
    public static class Builder{
        private final AlertController.AlertParams P;
        public Builder(Context context){
            this(context,resolveDialogTheme(context,0));
        }
        
        public Builder(Context context,int theme){
            P = new AlertController.AlertParams(new ContextThemeWrapper(context,resolveDialogTheme(context,theme)));
            mTheme = theme;
        }
        
        public Builder setTitle(CharSequence title){
            P.mTitle = title;
            return this;
        }
        
        public Builder setMessage(CharSequence message){
            P.mMessage = message;
            return this;
        }
        
        public Builder setView(View view){
            P.mView = view;
            P.mViewSpacingSpecified = false;
            return this;
        }
        
        public AlertDialog create(){
            
            final AlertDialog dialog = new AlertDialog(P.mContext,mTheme,false);
            P.apply(dialog.mAlert);
            dialog.setCancelable(P.mCancelable);
            if(P.mCancelable){
                dialog.setCanceldOnTouchOutsize(true);
            }
            dialog.setOnCancelListener(P.mOnCancelListener);
            if(P.mOnKeyListener != null){
                dialog.setOnKeyListener(P.mOnKeyListener);
            }
            return dialog;
        }
    }
}
```





