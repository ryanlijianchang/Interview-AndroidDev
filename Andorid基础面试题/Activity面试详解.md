### 一. 什么是Activity？ ###
Activity是Android提供的一个与用户直接打交道的组件，Android提供了一系列的界面让用户点击和各种滑动操作，这就是Activity。

### 二. Activity的四种状态 ###

1. running：Activity处于活动状态，处于栈顶的状态。
2. paused：表明Activity失去焦点的时候，或者被非全屏Activity占据的时候，或者说被透明Acitivity放在栈顶的时候，它会处于paused状态。内存不足时，会被系统回收。
3. stopped：表明Activity被另一个Activity覆盖的时候，会处于stopped状态，不再是可见。内存不足时，会被系统回收。
4. killed：表明Activity已经被系统回收掉。

### 三. Activity生命周期分析 ###

1. **Activity启动：onCreate() --> onStart() --> onResume()**
	- *onCreate()*：生命周期第一个调用方法，创建Activity时必须重写的方法，我们需要在这个方法里面设置布局文件(setContentView)。它是我们的Activity还没可见的状态，所以我们一些初始化的操作可以在这里面进行，例如图片的预加载，数据加载之类的。
	- *onStart()*：表明Acitivity正在启动，这时候Acitivity已经可见，但是还不能和用户进行交互（点击、滑动等）。
    - *onResume()*：表明Activity已经可见，并且和用户进行交互。
2. **点击Home键回到主界面（Activity不可见）： --> onPause() --> onStop()**
    - *onPause()*：表明整个Activity是处于停止状态， Activity可见但是不可触摸的状态。
    - *onStop()*：整个Activity已经被覆盖，不可见，处于后台运行，如果内存不足，会被销毁。
3. **再次回到原Activity：--> onRestart() --> onStart() --> onResume()**
   - *onRestart()*：不可见状态到可见状态会调用此方法。
4. **退出当前Activity：  --> onPause() --> onStop() --> onDestroy()**
   - *onDestroy()*：可以做资源回收，资源释放。

### 四. Android进程优先级 ###
1. 前台进程：用户可见，可交互。
2. 可见进程：用户可见，不可交互。一般是处于onPause状态，优先级低于前台进行，但是由于用户随时可能切回来，所以系统不会轻易销毁它。
3. 服务进程：一个服务进程就是一个Service，用户不可见。
4. 后台进程：切换到后台，不可见，根据内存进程决定是否被回收。
5. 空进程：这是一种系统缓存机制，其实就是个进程的外壳，当有新进程创建的时候，这个空进程可以加快进程创建速度，当系统内存不足的时候，首先销毁空进程。

优先级：前台进程 > 可见进程 > 服务进程 > 后台进程 > 空进程


### 五、Activity任务栈 ###

