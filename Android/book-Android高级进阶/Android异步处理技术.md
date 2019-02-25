# Android异步处理技术

​	移动应用的开发要求我们再去处理好主线程和子线程之间的关系,耗时的操作应该放在子线程中,避免阻塞主线程,导致ANR.异步处理技术是提高应用性能,解决主线程和子线程之间通信问题的关键.

​	在Android中,异步处理技术有很多种,常见的Thread,AsyncTask,Handler&Looper.Executors等,我们需要根据具体业务需求进行选择,一个完整的异步处理技术继承树

​							Thread

​			HandlerThread								Executor

​	AsyncQueryHandler   IntentService 						AsyncTask

​														Loader

## 1.Thread

​	线程是java的概念,它是实际执行任务的基本单元.Thread是Android中异步处理技术的基础,创建线程两种方法

- 集成Thread类重写run方法

  ```java
  public class MyThread extends Thread{
      @Override
      public void run(){
          //实现具体逻辑,文件读写,网络请求等
      }
  }
  
  public void startThread(){
      new MyThread().start();
  }
  ```

- 实现Runnable接口并实现run方法

  ```java
  public class MyRunnable implements Runnable{
      @Override
      public void run(){
          //实现具体逻辑,文件读写,网络请求等
      }
  }
  
  public void startThread(){
      new Thread(new MyRunnable()).start();
  }
  ```

		Android应用中各种类型的线程本质上都基于Linux系统的pthreads,在应用层可以分为三种类型的线程

- 主线程:主线程也成为UI线程,随着应用启动而启动,主线程用来运行Android组件,同时刷新屏幕上的UI元素.Android系统如果检测到非主线程更新UI组件,那么就会抛出异常,只有主线程才能操作UI,是因为Android的UI工具包不是线程安全的.主线程中创建的Handler会顺序执行接收到的消息,包括从其他线程发送的消息.因此,如果消息队列中前面的消息没有很快执行完,那么它可能会阻塞队列中的其他消息的即时处理
- Binder线程:Binder线程用于不同进程之间线程的通信,每个进程都维护了一个线程池,用来处理其他进程中线程发送的消息,这些进程包括系统服务,Intents,ContentProviders和Service等.在大部分情况下,应用不需要关心BInder线程,因为系统会优先将请求转换为使用主线程...一个典型的需要使用BInder线程的场景是AIDL接口绑定service
- 后台线程:在应用中显式创建的线程都是后台线程,也就是当刚创建出来时,这些线程的执行体是空的,需要手动添加人途.在Linux系统层面,主线程和后台线程是应一样的.在Android框架中,通过WindowManager赋予了主线程只能处理UI更新以及后台线程不能直接操作UI的限制.

## 2.HandlerThread

​	HandlerThread是一个集成了Looper和MessageQueue的线程,当启动HandlerThread时,会同时生成Looper和MessageQueue,然后等待消息进行处理,

```java
@Override
public void run(){
    mTid = Process.myTid();
    Looper.prepare();
    synchronized(this){
        mLooper = Looper.myLooper();
        notifyAll();
    }
    Process.setThreadPriority(mPriority);
    onLooperPrepared();
    Looper.loop();
    mTid = -1;
}
```

​	使用HandlerThread的好处是开发者不需要自己去创建和维护Looper,它的用法和普通线程一样

```java
HandlerThread handlerThread = new HandlerThread("workThread");
handlerThread.start();

handler = new Handler(handlerThread.getLooper()){
  @Override
    public void handleMessage(Message msg){
        super.handleMessage(msg);
        //处理接收到的消息
    }
};
```

​	HandlerThread中只有一个消息队列,队列中的消息是顺序执行的,因此是线程安全的,当然吞吐量自然受到一定的影响,队列中的任务可能会被前面没有执行完的任务阻塞.HandlerThread的内部机制确保了在创建Looper和发送消息之间不存在竞态条件,这个是通过将HandlerThread.getLooper实现为一个阻塞操作实现的,只有当HandlerThread准备好接受消息之后才会返回

```java
public Looper getLooper(){
    if(!isAlive()){
        return null;
    }
    
    //如果线程已经启动,那么在Looper准备好之前应先等待
    synchronized(this){
        while(isAlive() && mLooper == null){
            try{
                wait();
            }catch(InterruptException e){
                
            }
        }
    }
    return mLooper;
}
```

​	如果具体业务要求在HandlerThread开始接受消息之前要进行某些初始化操作的话,可以重写HandlerThread的onLooperPrepared函数,例如可以在这个函数中创建与HandlerThread关联的Handler实例,这同时也可以对外隐藏Handler实例

```java
public class MyHandlerThread extends HandlerThread{
    private Handler mHandler;
    public MyHandlerThread(){
        super("MyHandlerThread",Process.THREAD_PRIORITY_BACKGROUND);
    }
    
    @Override
    protected void onLooperPrepared(){
        super.onLooperPrepared();
        mHandler = new Handler(getLooper()){
            @Override
            public void handerMessage(Message msg){
                switch(msg.what){
                    case 1:
                        //Handler message
                        break;
                    case 2:
                        //Hanlder message
                        break;
                }
            }
        });
    }
}

public void publishedMethod1(){
    mHandler.sendEmptyMessage(1);
}

public void publishedMethod2(){
    mHanlder.sendEmptyMessage(2);
}
```

## 3.AsyncQueryHandler

AsyncQueryHandler是用于在ContentProvider上面执行异步的CRUD操作的工具类,CRUD操作会被放到一个单独的子线程中执行,当操作结束获取到结果后,将通过消息的方式传递给调用AsyncQueryHandler的线程,通常就是主线程.AsyncQueryHandler是一个抽象类,继承自Handler,通过封装ContentResolver,HandlerThread,Handler等实现对ContentProvider的异步操作.

AsyncQueryHandler封装了四个方法来操作ContentProvider,

```java
final void startDelete(int token,Object cookie,Uri uri,String selection,String[] selectionArgs);

final void startInsert(int token,Object cookie,Uri uri,ContentValues initialValues);

final void startQuery(int token,Object cookie,Uri uri,String[] projection,String selection,String[] selectionArgs,String orderBy);

final void startUpdate(int token,Object cookie,Uri uri,ContentValues values,String selection,String[] selectionArgs);
```

AsyncQueryHandler的子类可以根据实际需求实现下面的回调函数,从而得到上面的CRUD操作的返回结果

```java
@Override
protected void onDeleteComplete(int token,Object cookie,int result){
    //...
}

@Override
protected void onUpdateComplete(int token,Object cookie,int result){
    //....
}

@Override
protected void onInsertComplete(int token,Object cookie,Uri result){
    //...
}

@Override
protected void onQueryComplete(int token,Object cookie,Cursor result){
    //...
}
```

## 4.IntentService

我们知道service的各个生命周期函数是运行在主线程的,因此它本身并不是一个异步处理技术.为了能够在service中实现在子线程中处理耗时任务,Android引入了一个service的子类IntentService.IntentService具有Service一样的生命周期,同时也提供了在后台线程中处理异步任务的机制.与HandlerThread类似,IntentService也是在一个后台线程中顺序执行所有的任务,我们通过给Content.startService传递一个Intent类型的参数可以启动IntentService的异步执行,如果此时IntentService正在运行中,那么这个新的Intent将会进入队列进行排队,直到后台线程处理完队列前面的任务;如果此时IntentService没有在运行,那么将会启动一个新的IntentService,当后台线程队列中所有任务处理完成之后,IntentService将会结束它的生命周期,因此IntentService不需要开发者手动结束.

IntentService本身是一个抽象类,因此使用前需要继承它并实现onHandlerIntent方法,在这个方法中实现具体的后台处理业务逻辑,同时在子类的构造方法中需要调用super(String name)传入子类的名字.

```java
public class SimpleIntentService extends IntentService{
    public SimpleIntentService(){
        super(SimpleIntentService.calss.getName());
        setIntentRedelivery(true);
    }
    
    @Override
    protected void onHandleIntent(Intent intent){
        //这个方法是在后台线程中调用的
    }
}
```

上面代码中的setIntentRedelivery方法如果设置为true,那么IntentService的onStartCommand方法将会返回START_REDELIVERY_INTENT,这时,如果onHandlerIntent方法返回之前进程死掉了,那么进程将会重新启动,intent将会重新投递.

```java
<service android:name=".SimpleIntentService" />
```

IntentService是通过HandlerThread来实现后台任务的处理的

```java
public abstract class IntentService extends Service{
    private volatile Looper mServiceLooper;
    private volatile ServiceHandler mServiceHandler;
    private String mName;
    private boolean mRedelivery;
    
    private final class ServiceHandler extends Handler{
        public ServiceHandler(Looper looper){
            super(looper);
        }
        
        @Override
        public void handleMessage(Message msg){
            onHandleIntent((Intent)msg.obj);
            stopSelf(msg.arg1);
        }
    }
    
    public IntentService(String name){
        super();
        mName = name;
    }
    
    @Override
    public void onCreate(){
        super.onCreate();
        HandlerThread thread = new HandlerThread("IntentService ["+mName+"]");
        thread.start();
        
        mServiceLooper = thread.getLooper();
        mServiceHandler = new ServieHandler(mServiceLooper);
    }
    
    @Override
    public void onStart(Intent intent,int startId){
        Message msg = mServiceHandler.obtainMessage();
        msg.arg1 = startId;
        msg.obj = intent;
        mServiceHandler.sendMessage(msg);
    }
    
    @Override
    public int onStartCommand(Intent intent,int flags,int startId){
        onStart(intent,startId);
        return mRedelivery ? START_REDELIVERY_INTENT : START_NOT_STICKY;
    }
}

```

## 5.Executor Framework

​	我们知道,创建和销毁对象,是存在开销的,如果应用中频繁出现线程的创建和销毁,那么会影响到应用的性能.使用Java Executor框架可以通过线程池等机制解决这个问题,改善用户体验.Executor框架为开发者提供了如下能力

- 创建工作线程池,同时通过队列来控制能够在这些线程执行的任务的个数

- 检测导致线程意外终止的错误

- 等待线程执行完成并获取执行结果

- 批量执行线程,并通过固定的顺序获取执行结果

- 在合适的时机启动后台线程,从而保证线程执行结果可以很快反馈给用户

  Executor框架的基础是一个名为Executor的接口定义,Executor的主要目的是分离任务的创建和它的执行,最

终实现上述功能

```java
public interface Executor{
    void execute(Runnable command);
}
```

开发者可以通过实现Executor接口并重写execute方法从而实现自己的Executor类,最简单的是直接在这个方法中创建一个线程来执行Runnable

```java
public class SimpleExecutor implements Executor{
    @Override
    public void execute(Runnable runnable){
        new Thread(runnable).start();
    }
}
```

当然,实际应用中execute方法很少是这么简单的,通常需要增加类似队列,任务优先级等功能,最终实现一个线程池.线程池是任务队列和工作线程的集合,这两者组合起来实现生产者消费者模式.Executor框架为开发者提供了预定义的线程池实现

- 固定大小的线程池:通过Executors.newFixedThreadPool(n)创建,其中n表示线程池中线程的个数
- 可变大小的线程池:通过Executors.newCachedThreadPool()创建,当有新的任务需要执行时,线程池会创建新的线程来处理它,空闲的线程会等待60秒来执行新任务,当没有任务可执行时就自动销毁,因此可变大小线程池会根据任务队列的大小而变化
- 单个线程的线程池:通过Executors.newSingleThreadExecutor()创建,这个线程池中永远只有一个线程来串行执行任务队列中的任务

预定义的线程池都是基于ThreadPoolExecutor类之上构建的,而通过ThreadPoolExecutor开发者可以自定义线程池的一些行为,我们主要来看看这个类的构造函数定义.

```java
ThreadPoolExecutor executor = new ThreadPoolExecutor(
	int corePoolSize,
	int maximumPoolSize,
	long keepAliveTime,
	TimeUnit unit,
	BlockingQueue<Runnable> workQueue);
```

- corePoolSize:核心线程数,核心线程会一直存在于线程中,即时当前没有任务需要处理;当线程数小于核心线程数时,即时当前有空闲的线程,线程池也会优先创建新的线程来处理
- maximumPoolSize:最大线程数,当线程数大于核心线程数,且任务队列已经满了,这时线程池就会创建新的线程,直到线程数量达到最大线程数为止
- keepAliveTime:线程的空闲存活时间,当线程的空闲时间超过这个之时,线程会被销毁,知道线程数等于核心线程数
- unit:keepAliveTime的单位
- workQueue:线程池所使用的任务缓冲队列

## 6.AsyncTask

AsyncTask是在Executor框架基础上进行的封装,它实现将耗时任务移动到工作线程中执行,同时提供方便的接口实现工作线程和主线程的通信,使用AsyncTask一般会用到

```java
public class FullTask extends AsyncTask<Params,Progress,Result>{
    @Override
    protected void onPreExecute(){}
    
    @Override
    protected Result doInBackground(Params...params){}
    
    @Override
    protected void onProgressUpdate(Progress... progress){}
    
    @Override
    protected void onPostExecute(Result result){}
    
    @Override
    protected void onCancelled(Result result){}
}
```

其中,除了doInBackground方法是在工作线程中执行,其他的都是在主线程中执行,AsyncTask需要注意的地方.

| API level | execute 方法 | executeOnExecutor方法 |
| --------- | ------------ | --------------------- |
| 1-3       | 串行执行     | 无                    |
| 4-10      | 并行执行     | 无                    |
| 11-12     | 并行执行     | 串行或并行            |
| 13+       | 串行执行     | 串行或并行            |

可以看到,如果使用AsyncTask执行的任务需要并行执行的话,那么在API Level大于13的版本上建议只用executeOnExecutor

一个应用中使用的所有AsyncTask实例会共享全局的属性,也就是说如果AsyncTask中的任务是型,那么应用中所有的AsyncTask都会进行排队,只有等前面的任务执行完成之后,才会接着执行下一个AsyncTask中的任务,在executorOnExecutor(AsyncTask.SERIAL_EXECUTOR)或者API Level大于13的系统上面执行execute方法,都会是这个效果.如果AsyncTask是异步执行,那么在四核CPU系统上,最多也只有五个任务可以同时进行,其他任务需要在队列中进行排队,等待空闲的线程.之所以会出现折冲情况是由于AsyncTask中的ThreadPoolExecutor指定核心线程数是系统CPU核数 + 1

```java
public abstract class AsyncTask<Params,Progress,Result>{
    private static final int CPU_COUNT = Runtime.getRuntime().avaliableProcessors();
    private static final int CORE_POOL_SIZE = CPU_COUNT +1;
    private static final int MAXIMUM_POOL_SIZE = CPU_COUNT *2 +1;
    private static final int KEEP_ALIVE = 1;
    
    public static final Executor THREAD_POOL_EXECUTOR = new ThreadPoolExecutor(CORE_POOL_SIZE,
               MAXIMUM_POOL_SIZE,KEEP_ALIVE,TimeUnit.SECONDS,sPoolWorkQueue,sThreadFactory);
}
```

## 7.Loader

Loader是Android3.0开始引入的一个异步数据加载框架,它使得在Activity或者Fragment中异步加载数据变得很简单,同时它在数据源发生变化时,能够及时发出消息通过.Loader框架设计的API

- Loader:加载器框架的基类,封装了实现异步数据加载的接口,当一个加载器被激活后,他就会开始监视数据源并在数据发生改变时发送新的结果
- AsyncTaskLoader:Loader的子类,顾名思义,它是基于AsyncTask实现异步数据加载,他是一个抽象类,子类必须实现loadInBackground方法,在其中进行具体的数据加载操作.
- CursorLoader:AsyncTaskLoader的子类,封装了堆ContentResolver的query操作,实现从ContentProvider中查询数据的功能
- LoaderManager:抽象类,Activity和Fragment默认都会关联一个LoaderManager的对象,开发者只需要通过getLoaderManager即可获取.LoaderManager是用来管理一个或者多个加载器对象的
- LoaderManager.LoaderCallbacks:LoaderManager的会掉接口,主要有三个方法
  - onCreateLoader:初始化并返回一个新的Loader实例
  - onLoadFinished:当一个加载器完成加载过程之后会回调这个方法
  - onLoadReset:当一个加载器被重置并且数据无效时会回调这个方法

```java
public class ContactActivity extends ListActivity implements LoaderManager.LoaderCallbacks<Cursor>{
    private static final int CONTACT_NAME_LOADER_ID = 0;
    
    //这里的PROJECTION只获取ID和用户名两个字段
    static final String[] CONTACTS_SUMMARY_PROJECTION = new String[]{
        ContactsContract.Contacts._ID,
        ContactsContract.Contacts.DIAPLAY_NAME
    };
    
    SimpleCursorAdapter mAdapter;
    
    public void onCreate(Bundle savedInstanceState){
        super.onCreate(savedInstanceState);
        initAdapter();
        //通过LoaderManager初始化Loader,这会回调到onCreateLoader
        getLoaderManager().initLoader(CONTACT_NAME_LOADER_ID,null,this);
    }
    
    private void initAdapter(){
        mAdapter = new SimpleCursorAdapter(this,
                             android.R.layout.simple_list_item1,null,
                             new String[]{ContactsContract.Contacts.DISPLAY_NAME},
                             new int[]{android.R.id.text1}, 0);
        setListAdapter(mAdapter);
    }
    
    @Override
    public Loader<Cursor> onCreateLoader(int id,Bundle args){
        //实际创建Loader的地方,此处使用CursorLoader
        return new CursorLoader(this,ContactsContract.Contacts.CONTENT_URI,
                                CONTACTS_SUMMARY_PROJECTION,null,null,
                                ContactsContract.Contacts.DISPLAY_NAME + " ASC");
    }
    
    @Override
    public void onLoadFinished(Loader<Cursor> loader,Cursor c){
        //后台线程中加载完数据后,回调这个方法将数据传递给主线程
        mAdapter.swapCursor(c);
    }
    
    @Override
    public void onLoaderReset(Loader<Cursor> loader){
        //Loader被重置后的回调,在这里可以重新刷新页面数据
        mAdapter.swapCursor(null);
    }
    
}
```

## 总结

Android平台提供了如此多的异步处理技术,我们在进行选择的时候需要根据具体的业务需求而定,综合考量以下几个因素

- 尽量使用更少的系统资源,例如CPU和内存等
- 为应用提供更好的性能和响应度
- 实现和使用起来不复杂
- 写出来的代码是否符合好的设计,是否易于理解和维护