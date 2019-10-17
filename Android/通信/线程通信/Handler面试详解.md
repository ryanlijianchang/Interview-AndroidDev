### 一、什么是Handler ###

Android消息机制的上层接口，Handler通过发送和处理Message和Runnable对象来关联响应线程的MessageQueue。
1. 可以让对应的Message和Runnable在未来某个时间点进行响应的处理
2. 让自己想要处理的耗时操作放到子线程，让更新UI放到主线程

### 二、Handler使用 ###

1. post(Runnable)
2. sendMessage(Message)

### 三、Handler机制 ###

1. Handler handler = new Handler(Looper.getMainLooper())；因为主线程在初始化的时候已经进行了Looper.prepare和Looper.loop，所以在主线程不需要再进行该操作，但是如果在子线程初始化Looper的时候，就需要我们主动调用这两个方法。在初始化Handler的时候传入了线程对应的Looper，该Looper会绑定一个该线程的MessageQueue，Handler也会把该MeesageQueue初始化。
2. 在需要处理任务的时候，调用handler.sendMessage传入Message，把消息加入到MeesageQueue里面。如果调用handler.post(Runnable)，该Runnable会转换成Meesage，最后还是调用sendMessage方法。
3. Looper会不断循环MessageQueue，然后调用对应线程的Handler.dispatchMessage方法，回调到对应Handler的handleMessage，从而实现了消息机制。


### 四、Handler引起的内存泄露原因以及最佳解决方案 ###

>Handler 允许我们发送延时消息，如果在延时期间用户关闭了 Activity，那么该 Activity 会泄露。
>这个泄露是因为 Message 会持有 Handler，而又因为 Java 的特性，内部类会持有外部类，使得 Activity 会被 Handler 持有，这样最终就导致 Activity 泄露。
>解决该问题的最有效的方法是：将 Handler 定义成静态的内部类，在内部持有 Activity 的弱引用，并及时移除所有消息。


```
private static class SafeHandler extends Handler {

    private WeakReference<HandlerActivity> ref;

    public SafeHandler(HandlerActivity activity) {
        this.ref = new WeakReference(activity);
    }

    @Override
    public void handleMessage(final Message msg) {
        HandlerActivity activity = ref.get();
        if (activity != null) {
            activity.handleMessage(msg);
        }
    }
}

并且再在 Activity.onDestroy() 前移除消息，加一层保障：

@Override
protected void onDestroy() {
  safeHandler.removeCallbacksAndMessages(null);
  super.onDestroy();
}
```

### 五、为什么我们能在主线程直接使用 Handler，而不需要创建 Looper? ### 

注意：通常我们认为 ActivityThread 就是主线程。事实上它并不是一个线程，而是主线程操作的管理者，所以吧，我觉得把 ActivityThread 认为就是主线程无可厚非，另外主线程也可以说成 UI 线程。

在ActivityThread 里 调用了 Looper.prepareMainLooper() 方法创建了 主线程的 Looper ,并且调用了 loop() 方法，所以我们就可以直接使用 Handler 了。

##### 六、Handler 里藏着的 Callback 能干什么？ #####

```
来看看 Handler.dispatchMessage(msg)方法：

public void dispatchMessage(Message msg) {
  //这里的 callback 是 Runnable
  if (msg.callback != null) {
    handleCallback(msg);
  } else {
    //如果 callback 处理了该 msg 并且返回 true， 就不会再回调 handleMessage
    if (mCallback != null) {
      if (mCallback.handleMessage(msg)) {
        return;
      }
    }
    handleMessage(msg);
  }
}
```

可以看到 Handler.Callback 有优先处理消息的权利，当一条消息被 Callback 处理并拦截（返回 true），那么 Handler 的 handleMessage(msg)方法就不会被调用了；如果Callback处理了消息，但是并没有拦截，那么就意味着一个消息可以同时被 Callback 以及 Handler 处理。

这个就很有意思了，这有什么作用呢？

我们可以利用 Callback 这个拦截机制来拦截 Handler 的消息！

场景：Hook ActivityThread.mH，在 ActivityThread 中有个成员变量 mH，它是个 Handler，又是个极其重要的类，几乎所有的插件化框架都使用了这个方法。