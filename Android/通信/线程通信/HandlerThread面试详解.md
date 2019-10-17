### 一、产生背景 ###
- 开启Thread子线程进行耗时操作
- 多次创建和销毁线程是很耗费系统资源的

### 二、HandlerThread是什么 ### 

Handler+Thread+Looper，是一个Thread，内部有Looper。

### 三、特点 ###

- HandlerThread本质是一个线程
- HandlerThread有自己内部的Looper对象，可以进行looper循环
- 通过HandlerThread的Looper对象传递给Handler，可以在handleMessage里面处理异步消息
- 优点是不会有堵塞，减少了性能的消耗，缺点是不能多任务同时操作，需要等待处理
- 与线程池不同，HandlerThread是一个串行队列，HandlerThread背后只有一个线程

### 四、源码解析 ###

如下代码所示，关键点在于构造方法和getLooper方法里面的两个同步代码块，例如在UI线程创建了HanlderThread并运行之后，我们通过getLooper方法获取Looper，它会在同步代码块里面进行wait操作，直到run方法里面创建完looper并notifyAll之后，才会返回looper对象。

```
public class HandlerThread extends Thread {

    @Override
    public void run() {
        mTid = Process.myTid();
        Looper.prepare();
        synchronized (this) {
            mLooper = Looper.myLooper();
            notifyAll();
        }
        Process.setThreadPriority(mPriority);
        onLooperPrepared();
        Looper.loop();
        mTid = -1;
    }

    public Looper getLooper() {
        if (!isAlive()) {
            return null;
        }

        // If the thread has been started, wait until the looper has been created.
        synchronized (this) {
            while (isAlive() && mLooper == null) {
                try {
                    wait();
                } catch (InterruptedException e) {
                }
            }
        }
        return mLooper;
    }

}
```