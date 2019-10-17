### 1.IntentService是什么 ###

IntentService是Service的一个派生类，顾名思义是一个Service。我们都知道，对于一些不需要跟随Activity生命周期的操作，例如上传、下载等我们在Service里面进行，但是因为Service是在UI线程的，所以如果直接在Service里面进行耗时操作的话，就会导致ANR，所以我们常见的做法是在Service里面创建子线程进行耗时操作。这样的做法就会导致Service里面的线程管理起来比较麻烦，同时还需要创建Looper、Handler实现消息通信，在不使用的时候还需要将Service销毁。这样无疑会增加工作量，同时也并不会是比较优雅的做法。

那么IntentService就是为了解决这个问题而出现的，在IntentService是一个可以执行异步任务的Service，同时在执行完成异步任务后，会自动销毁的Service，实现了代码封装的同时，避免了资源的浪费。

### 2.IntentService的实现原理 ###

如下代码所示，可以看到源码并不是很多，但是整个设计还是非常优雅的，它里面主要是通过HandlerThread创建了一个子线程，HandlerThread又自动生成了一个Looper，所以IntentService就可以创建本地线程的Handler，并传入HandlerThread的Looper，实现消息处理。

因为每次通过Intent打开Service，都会执行onStartComand操作，它在里面执行了onStart方法，onStart方法里面会执行onHandleIntnet这个抽象方法，以供实现类在里面进行耗时操作，进行完onHandleIntent之后，就会调用stopSelf方法停止该Service，释放资源。


```
public abstract class IntentService extends Service {
    private volatile Looper mServiceLooper;
    private volatile ServiceHandler mServiceHandler;
    private String mName;
    private boolean mRedelivery;

    private final class ServiceHandler extends Handler {
        public ServiceHandler(Looper looper) {
            super(looper);
        }

        @Override
        public void handleMessage(Message msg) {
            onHandleIntent((Intent)msg.obj);
            stopSelf(msg.arg1);
        }
    }

    public IntentService(String name) {
        super();
        mName = name;
    }

    public void setIntentRedelivery(boolean enabled) {
        mRedelivery = enabled;
    }

    @Override
    public void onCreate() {
        super.onCreate();
        HandlerThread thread = new HandlerThread("IntentService[" + mName + "]");
        thread.start();

        mServiceLooper = thread.getLooper();
        mServiceHandler = new ServiceHandler(mServiceLooper);
    }

    @Override
    public void onStart(@Nullable Intent intent, int startId) {
        Message msg = mServiceHandler.obtainMessage();
        msg.arg1 = startId;
        msg.obj = intent;
        mServiceHandler.sendMessage(msg);
    }

    @Override
    public int onStartCommand(@Nullable Intent intent, int flags, int startId) {
        onStart(intent, startId);
        return mRedelivery ? START_REDELIVER_INTENT : START_NOT_STICKY;
    }

    @Override
    public void onDestroy() {
        mServiceLooper.quit();
    }

    @Override
    @Nullable
    public IBinder onBind(Intent intent) {
        return null;
    }

    @WorkerThread
    protected abstract void onHandleIntent(@Nullable Intent intent);
}

```