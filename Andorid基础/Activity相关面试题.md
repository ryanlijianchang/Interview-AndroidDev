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

**1.Standard模式**

> - 特点：
> 1.Activity的默认启动模式
> 2.每启动一个Activity就会在栈顶创建一个新的实例。
>     
> - 缺点：当Activity已经位于栈顶时，而再次启动Activity时还需要在创建一个新的实例，不能直接复用。

**2.SingleTop模式**
> - 特点：该模式会判断要启动的Activity是否在栈顶，如果在栈顶，则会直接复用，否则创建新的实例。
> - 缺点：栈中可能存在多个实例。

**3.SingleTask模式**
> - 特点：该模式会判断要启动的Activity是否存在任务栈中，如果存在，则直接复用，并把该Activity之上的所有实例全部出栈。

**4.SingleInstance模式**

> - 特点：该模式的Activity会启动一个新的任务栈来管理Activity实例，并且该实例在整个系统中只有一个。无论从哪个任务栈中启动该Activity，都会是该Activity所在的任务栈转移到前台，从而是Activity显示。主要是为了在不同程序中共享一个Activity实例。

### 六、Activity的跳转方式 ###

在上面Activity任务栈中的介绍我们知道，不同的Activity有可能运行在同一个任务栈，有可能运行在不同的任务栈中，也就是Activity所处的进程有可能是不一样的。因此Android就提供了一种可靠的机制帮助我们在Activity之间传递数据，而Intent就是其中的一种。

Intent负责对操作规则的描述，包括以下属性：

- component(组件)，
- action（动作），
- category（类别），
- data（数据），
- type（数据类型），
- extras（扩展信息），
- flags（标志位），

Android系统会根据这些规则，分析并找出合适的活动去启动。

Intent类型分为**显式Intent（直接类型）**、**隐式Intent（间接类型）**。上面的component属性为直接类型，其他均为间接类型。


**1. 显式Intent**

当我们明确指定打开一个组件时，就会使用到Component这个属性，然后通过Intent传递该属性，即可实现显示Intent跳转Activity：

    Intent intent = new Intent();
    ComponentName component = new ComponentName(MainActivity.this, SecondActivity.class);
    intent.setComponent(component);                
    startActivity(intent);      

它等价于我们常用的：

    Intent intent = new Intent();
	// setClass底层实现就是把对应的参数转化为ComponentName
	intent.setClass(MainActivity.this, SecondActivity.class);                
    startActivity(intent);    

**2. 隐式Intent**

在上面的属性中，除了Component属性，其它均为隐式Intent需要的属性，只要当我们指定这些隐式属性，Android系统会根据`AndroidManifest.xml`文件里面声明的组件列表，然后根据他们的`intent-filter`过滤出符合我们要求的组件。

例如，我们通过Action来启动Activity，我们只要在`AndroidManifest.xml`对应的Activity的`intent-filter`属性中添加对应的Action即可，注意，必须添加DEFAULT的catergory，因为`AndroidManifest.xml`里面的组件必须配置了action和catergory才能响应，否则会报`android.content.ActivityNotFoundException`这个错误，所以系统在`startActivity()`方法实现里面会给我们自动增加Default类型的catergory，如下：

    <activity android:name=".SecondActivity">
        <intent-filter>
            <!--添加action：NEW_ACTION-->
            <action android:name="NEW_ACTION" />

			<!--catergory必须添加DEFAULT类别-->
            <category android:name="android.intent.category.DEFAULT" />
        </intent-filter>
    </activity>

那么我们在Activity中就可以直接指定Action即可打开该Activity：

	Intent intent = new Intent();
	intent.setAction("NEW_ACTION");
	// 因为在startActivity中，系统会自动帮我们添加DEFAULT的catergory
	// 所以我们需要在AndroidManifest.xml中增加DEFAULT的catergory
	startActivity(intent);

***Scheme跳转***

除此之外，一个比较重要的隐式跳转就是Scheme跳转，什么是Scheme跳转呢？

我们都知道，一个普通的URL分为几个部分，`scheme`、`host`、`relativePath`、`query`。比如：`https://www.baidu.com/search/word=abc`，在这个链接里面，`scheme`就是`https`，`host`就是`www.baidu.com`，`relativePath`就是`search`，`query`就是`word=abc`。平时，我们都习惯了`http`类型的`scheme`，它一般就是对应着一个网页的URL，如果我们希望我们应用的每一个页面都对应着一个URL，我们应用内部的跳转就可以通过URL来跳转、其他应用也能通过URL来跳转到我们应用相应的页面，这个就是我们所说的通过Scheme实现应用内Activity的跳转。那么该如何实现呢？

通常业内常用的做法就是定义一个通用的Activity，并设置该Activity的scheme，然后通过服务器下发一个scheme为该Activity定义的scheme的URL，在该Activity内对URL进行解析，分别解析出上面所说的`scheme`、`host`、`relativePath`、`query`，然后再跳转到我们所需要的Activity。


### 七、Activity的通信方式 ###

在Android系统中，Activity之间的通信方式林林种种，具体需要使用哪个方式，我觉得还是得根据具体的场景，例如当我们需要考虑跨进程，我们可以选择intent来传递数据；当考虑发送数据到多个Activity，那么就可以用Broadcast或者是Callback的方式；其他的还可以考虑数据存储实现数据传递等。

**1. Intent传递数据**

Intent是Android中组件连接的桥梁，我们都知道四大组件可以通过指定process属性使它们在不同独立的进程，所以Intent是跨进程的一种数据传递机制，它的使用方式也是非常简单：

	Intent intent = new Intent(MainActivity.this, SecondActivity.class);
	intent.putExtra("data", "data");
	startActivity(intent);

而在SecondActivity中，通过传入的intent就可以获取到数据：

	String data = getIntent().getStringExtra("data");

其实这种方式最终调用的还是`Activity`的`startActivityForResult(Intent intent, int requestCode)`方法，只是给我们设置了默认的请求码，那么`startActivityForResult`是什么呢？

我们都知道，`startActivityForResult(Intent intent, int requestCode)`和`onActivityResult(int requestCode, int resultCode, Intent data)`都是成对出现的，传递数据时比较重要的两个概念就是
`requestCode`和`resultCode`。

- requestCode：请求码。例如当我们的Activity有多个按钮请求打开Activity，我们可以通过请求码来标识是哪个按钮触发的请求。
- resultCode：结果码。即返回当前的上一个Activity的标记，例如当前Activity多个按钮触发打开多个Activity，我们就可以根据结果码判断是哪一个Activity返回到当前Activity。

当我们识别了从哪个地方跳转和从哪个地方返回，那么数据就是从我们的intent里面获取，我们再看一下intent是怎么传递数据的？

通常我们都是通过intent.putExtra(key, value)来传递数据：

	Intent intent = new Intent(MainActivity.this, SecondActivity.class);
	intent.putExtra("data", "data");
	startActivity(intent);

同样，我们也可以通过Bundle来传递：

    Intent intent = new Intent(MainActivity.this, SecondActivity.class);
    Bundle bundle = new Bundle();
    bundle.putInt("data", value);
    intent.putExtras(bundle);
    startActivityForResult(intent, requestCode);

那么，这两种有什么区别呢？这可是面试中真实遇到过的问题，其实只要我们看一眼源码，就非常清楚了，我们看一下Intent的源码：

	public class Intent implements Parcelable, Cloneable {
		// 省略..
	    private Bundle mExtras;
		// 省略..
	
		
		// 省略..
	
	    public @NonNull Intent putExtras(@NonNull Bundle extras) {
	        if (mExtras == null) {
	            mExtras = new Bundle();
	        }
	        mExtras.putAll(extras);
	        return this;
	    }
		
		// 省略..
	
	}

我们可以看到，Intent的每一个put方法，其实都是把传入的数据，再传到自身的成员变量Bundle，所以Intent的内部其实也是通过Bundle来进行传递数据的。那么这两种方式有什么区别呢？

从底层实现上来说是没区别，但是在使用的方便程度上，确实有一定的差距，例如当我们从A->B->C->D，我们需要把几个字符串从A传到D，假如我们使用`intent.putExtra(String, String)`的方式，那么我们就需要在每一个Activity都需要把intent的数据拿出来，再put进去，传递给下一个Activity。然而，如果我们通过`intent.putExtras(Bundle)`的形式，那么我们就可以在每个Activity拿到封装好的Bundle，并传到下一个Activity即可。

了解到Intent是通过Bundle来传递数据，那么Bundle支持什么数据类型呢？通过API我们知道，Bundle可以传递七种Java基本数据类型及其数组、Parcelabe、Serializable等，可以根据自己的实际需要传递不同类型的数据。

**2.广播传递数据**

广播作为Android四大组件之一，我们也是可以使用广播跨进程传递数据。同样的，我们只需要在发送数据的Activity设置广播的Action并传递数据，在接受信息的Activity注册广播接收器，并过滤相应的Action，读取数据，即可完成数据的传递。

**3.通过Callback传递数据**

我们都知道同一个进程是共享内存区的，所以我们如果不考虑跨进程的话，Activity之间的通信实际上是可以通过CallBack的方式来进行，类似于EventBus，在同一块内存区域里面有且仅有一个事件总线，我们可以将Activity注册到事件总线里面，即添加Callback监听，当我们需要传递数据时，往事件总线里面post需要的数据，在需要处理的Activity的Callback里面对数据进行相应即可。这种情况是需要不同Activity都是在同一个线程里面才能实现。

**4.通过数据存储实现传递数据**

数据存储可以实现数据共享，在不同的Activity对存储的数据进行读写即可实现数据传递。在Android中，常见的文件存储有`SharedPreferences`、`文件存储`、`Sql存储`、`ContentProvider存储`，每种方式都有各自的优缺点，根据实际场景择优使用。


