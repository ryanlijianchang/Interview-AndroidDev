### 一、Service是什么? ###

Service是一种可以在后台执行长时间运行操作而没有用户界面的应用组件。它是运行在主线程中，不可以进行耗时操作。如果需要执行长时间耗时操作，需要手动创建子线程进行处理，否则会出现UI线程被堵塞的问题。

### 二、Service和Thread的区别？ ###

Service和Thread是没有任何关系的。Thread是程序执行的最小单元，它是分配CPU的基本单位。而Service是运行在一个进程的主线程中，因此Service不是线程！

什么时候需要用到Thread呢？当我们需要执行耗时操作时，可以开启子线程并在子线程里面进行操作。而我们用到Service的时候，是如果我们一些操作和UI界面没有关系，或者不需要跟Activity的生命周期同步时，我们可以在Service里面执行这些操作，例如监听下载进度，MP3播放进度，网络请求等。

### 三、Service的启动方式 ###

**1. startService()：**
    其他组件调用startService()启动一个Service，一旦启动，Service将一直运行在后台，即使启动这个Service的组件已经被销毁。通常一个被start的Service会在后台执行单独的操作，也不需要给启动它的组件返回结果。只有当Service自己调用stop方法，或者其他组件调用stopService方法，才会终止。

**2. bindService()：**
    其他组件使用bindService启动的Service，这种方式会使Service和组件绑定在一起，当启动它的组件销毁时，Service也会自动进行unBind操作。同一个Service可以被多个组件绑定，只有所有绑定它的组件都进行unBind操作，这个service才会被销毁。
    
###  四、bindService的过程 ###

1. 创建BindService的服务端，继承自Service并在类中创建一个实现IBinder接口的对象，并提供公共方法给客户端调用。
2. BindService从onBind方法回调此IBinder实例。
3. Activity在onServiceConnected方法中获取IBinder实例，并使用提供的方法调用绑定的服务。

### 五、目前能否保证service不被杀死 ###

**1. Service设置成START_STICKY**

kill 后会被重启（等待5秒左右），重传Intent，保持与重启前一样

**2.提升service优先级**

在AndroidManifest.xml文件中对于intent-filter可以通过android:priority = "1000"这个属性设置最高优先级，1000是最高值，如果数字越小则优先级越低，同时适用于广播。
【结论】目前看来，priority这个属性貌似只适用于broadcast，对于Service来说可能无效

**3.提升service进程优先级**

Android中的进程是托管的，当系统进程空间紧张的时候，会依照优先级自动进行进程的回收
当service运行在低内存的环境时，将会kill掉一些存在的进程。因此进程的优先级将会很重要，可以在startForeground()使用startForeground()将service放到前台状态。这样在低内存时被kill的几率会低一些。

【结论】如果在极度极度低内存的压力下，该service还是会被kill掉，并且不一定会restart()

**4.onDestroy方法里重启service**

service +broadcast 方式，就是当service走onDestory()的时候，发送一个自定义的广播，当收到广播的时候，重新启动service
也可以直接在onDestroy()里startService
【结论】当使用类似口口管家等第三方应用或是在setting里-应用-强制停止时，APP进程可能就直接被干掉了，onDestroy方法都进不来，所以还是无法保证

**5.监听系统广播判断Service状态**

通过系统的一些广播，比如：手机重启、界面唤醒、应用状态改变等等监听并捕获到，然后判断我们的Service是否还存活，别忘记加权限
【结论】这也能算是一种措施，不过感觉监听多了会导致Service很混乱，带来诸多不便

**6.在JNI层,用C代码fork一个进程出来**

这样产生的进程,会被系统认为是两个不同的进程.但是Android5.0之后可能不行

**7.root之后放到system/app变成系统级应用**

**8.大招: 放一个像素在前台(手机QQ)**