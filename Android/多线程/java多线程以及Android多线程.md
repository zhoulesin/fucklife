# Java多线程-Android多线程

https://www.cnblogs.com/zoe-mine/p/7954605.html

## Java多线程

线程和进程的区别

- 线程和进程的本质:由CPU进行调度的并发式执行任务,多个任务被快速轮换执行,使得宏观上具有多个线程或者进程同时执行的效果
- 进程:在操作系统来说,一个运行的程序或者说一个动态的指令集合通常对应一个进程Process,它是系统进行资源分配和调度的





...







使用Callback和Future创建线程:Callable类似于Runnable,提供一个call方法作为线程执行体,并且可以有返回值,以及抛出异常,那么我们如何拿到返回值,java提供了Future接口,在接口定义了一些公共的方法来控制关联它的Callable任务,然后java给出了FutureTask类,该类实现了Future接口和Runnable接口,所以FutureTask的实例就可以作为Thread的target,所以通常做法是创建Callable接口实现类,并对该实现类的实例使用FutureTask来包装

```java
public class TThread{
    public static void main(String[] args){
        FutureTask<Integer> task = new FutureTask<Integer>((Callable<Integer>)()->{
            int i=0;
            for(;i<100;i++){
                syso(Thread.currentThread().getName()+"" +i);
            }
            return i;
        });
        
        new Thread(task,"由返回值的线程").start();
        try{
        	syso("子线程返回值:" + task.get());    
        }catch(){
        }
    }
}
```

线程生命周期

- 线程的生命周期包括:新建new,就绪runnable,运行running,阻塞blocked,死亡dead

- 新建和就绪:程序使用new关键字后,该线程处于新建状态,jvm为其分配内存,并初始化成员变量,程序调用start方法之后,该线程就处于就绪状态

- 运行和阻塞:如果就绪状态的线程获得了CPU,那么程序就处于运行状态
  当发生如下情况,线程将进入阻塞状态

  - 线程调用sleep方法,主动放弃所占用的CPU资源
  - 线程调用阻塞式IO方法,在该方法返回之前,该线程被阻塞
  - 线程试图获得一个同步监视器,但是该同步监视器被其他线程所持有
  - 线程等待某个通知notify,notify通常与wait配合使用
  - 线程调用suspend,挂起,该方法容易造成死锁,不建议使用

  当发生如下情况时,线程进入就绪状态,但是线程什么时候进入运行状态,需要根据系统调度来决定

  - sleep方法的线程经过了指定的sleep时间
  - 阻塞式io方法返回值
  - 成功获得了同步监视器
  - 线程获得了其他线程发出的通知,被唤醒
  - 挂起的线程调用了resume方法恢复

