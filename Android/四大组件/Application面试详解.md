- ### 一.如何判断应用被强杀

  在Application中定义一个static常量，赋值为－1，在欢迎界面改为0，如果被强杀，application重新初始化，在父类Activity判断该常量的值。

  ### 二、应用被强杀如何解决

  如果在每一个Activity的onCreate里判断是否被强杀，冗余了，封装到Activity的父类中，如果被强杀，跳转回主界面，如果没有被强杀，执行Activity的初始化操作，给主界面传递intent参数，主界面会调用onNewIntent方法，在onNewIntent跳转到欢迎页面，重新来一遍流程。

  ### 三、如何退出终止App

  **1、容器式**

  建立一个全局容器，把所有的Activity存储起来，退出时循环遍历finish所有Activity。

  这种方法比较简单，但是可以看到activityStack持有这Activity的强引用，也就是说当某个Activity异常退出时，Activity生命周期方法没有被执行，activityStack没有及时释放掉引用，就会导致内存问题。

  ```
  import java.util.ArrayList;
  import java.util.List;
  
  import android.app.Activity;
  import android.os.Bundle;
  
  public class BaseActivity extends Activity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
      super.onCreate(savedInstanceState);
      // 添加Activity到堆栈
      AtyContainer.getInstance().addActivity(this);
    }
  
    @Override
    protected void onDestroy() {
      super.onDestroy();
      // 结束Activity&从栈中移除该Activity
      AtyContainer.getInstance().removeActivity(this);
    }
  
  }
  
  class AtyContainer {
  
    private AtyContainer() {
    }
  
    private static AtyContainer instance = new AtyContainer();
    private static List<Activity> activityStack = new ArrayList<Activity>();
  
    public static AtyContainer getInstance() {
      return instance;
    }
  
    public void addActivity(Activity aty) {
      activityStack.add(aty);
    }
  
    public void removeActivity(Activity aty) {
      activityStack.remove(aty);
    }
  
    /**
     * 结束所有Activity
     */
    public void finishAllActivity() {
      for (int i = 0, size = activityStack.size(); i < size; i++) {
        if (null != activityStack.get(i)) {
          activityStack.get(i).finish();
        }
      }
      activityStack.clear();
    }
  
  }
  ```

  **2、广播式**

  通过在BaseActivity中注册一个广播，当退出时发送一个广播，finish退出。

  ```
  public class BaseActivity extends Activity {
  
    private static final String EXITACTION = "action.exit";
  
    private ExitReceiver exitReceiver = new ExitReceiver();
  
    @Override
    protected void onCreate(Bundle savedInstanceState) {
      super.onCreate(savedInstanceState);
      IntentFilter filter = new IntentFilter();
      filter.addAction(EXITACTION);
      registerReceiver(exitReceiver, filter);
    }
  
    @Override
    protected void onDestroy() {
      super.onDestroy();
      unregisterReceiver(exitReceiver);
    }
  
    class ExitReceiver extends BroadcastReceiver {
  
      @Override
      public void onReceive(Context context, Intent intent) {
        BaseActivity.this.finish();
      }
  
    }
  
  }
  ```

  **3、直接杀进程**

  通过直接杀死当前应用的进程来结束应用，非常简单粗暴，但是有很多问题。

  ActivityManager 的killBackgroundProcesses是AMS提供的杀死后台进程的方法,不能杀死自己。

  Process.killProcess 和 System.exit(0)两个都会kill掉当前进程。

  如果是在第一个 Activity调用Process.killProcess或System.exit(0) 都会 kill掉当前进程。但是如果不是在第一个Activity中调用，如ActivityA启动ActivityB，你在ActivityB中调用Process.killProcess 或 System.exit(0)当前进程确实也被kill掉了，但app会重新启动，又创建了一个新的进程，APP生命周期重新执行。其实Process.killProcess 或 System.exit(0) 都不建议直接调用， 进程是由 os 底层进行管理的，android 系统会自己进行处理回收进程。

  杀进程  

  ```
  android.os.Process.killProcess(android.os.Process.myPid());
  ```

  退出虚拟机

  ```
  System.exit(0);
  ```

  ActivityManager杀死后台进程

  ```
  ActivityManager manager = (ActivityManager) getSystemService(ACTIVITY_SERVICE);
  manager.killBackgroundProcesses(getPackageName());
  ```

  需要添加权限

  ```
  <uses-permission android:name="android.permission.KILL_BACKGROUND_PROCESSES" />
  ```

  **4、 Receiver+singleTask**

  我们知道Activity有四种加载模式，而singleTask就是其中的一种，使用这个模式之后，当startActivity时，它先会在当前栈中查询是否存在Activity的实例，如果存在，则将其至于栈顶，并将其之上的所有Activity移除栈。我们打开一个app，首先是一个splash页面，然后会finish掉splash页面。跳转到主页。然后会在主页进行N次的跳转，期间会产生数量不定的Activity，有的被销毁，有的驻留在栈中，但是栈底永远是我们的HomeActivity。这样就让问题变得简单很多了。我们只需两步操作即可优雅的实现app的退出。

  （1）、在HomeActivity注册一个退出广播，和第二个广播式一样，但是这里只需要在HomeActivity一个页面注册即可。

  （2）、设置HomeActivity的启动模式为singleTask。
  当我们需要退出的时候只需要

  ```
  startActivity(this,HomeActivity,class)
  ```

  再发送一个退出广播。上面代码首先会把栈中HomeActivity之上的所有Activity移除出栈，然后接到广播finish自己。

  **5、SingleTask改版式**

  这种方式是第四种方式的改进版，思路也很简单：
  （1）. 设置MainActivity的加载模式为singleTask
  （2）. 重写MainActivity中的onNewIntent方法
  （3）. 需要退出时在Intent中添加退出的tag

  第一步 设置MainActivity的加载模式为singleTask

  ```
  android:launchMode="singleTask"
  ```

  第二步 重写onNewIntent()方法

  ```
  private static final String TAG_EXIT = "exit";
   @Override
   protected void onNewIntent(Intent intent) {
     super.onNewIntent(intent);
     if (intent != null) {
       boolean isExit = intent.getBooleanExtra(TAG_EXIT, false);
       if (isExit) {
         this.finish();
       }
     }
   }
  ```

  第三步 退出

  ```
  Intent intent = new Intent(this,MainActivity.class);
  intent.putExtra(MainActivity.TAG_EXIT, true);
  startActivity(intent);
  ```

  **6、MainActivity界面 双击返回键退出**

  这种方式更加简单，只需要如下两步操作

  ```
  Intent intent = new Intent(this,MainActivity.class);
  intent.putExtra(MainActivity.TAG_EXIT, true);
  startActivity(intent);
  ```

  我们可以看到很多应用都是双击两次home键退出应用，就是基于这样的方式来实现的，这里在贴一下如何处理连续两次点击退出的源码  

  ```
  private boolean mIsExit;
  @Override
    /**
     * 双击返回键退出
     */
    public boolean onKeyDown(int keyCode, KeyEvent event) {
  
      if (keyCode == KeyEvent.KEYCODE_BACK) {
        if (mIsExit) {
          this.finish();
  
        } else {
          Toast.makeText(this, "再按一次退出", Toast.LENGTH_SHORT).show();
          mIsExit = true;
          new Handler().postDelayed(new Runnable() {
            @Override
            public void run() {
              mIsExit = false;
            }
          }, 2000);
        }
        return true;
      }
      return super.onKeyDown(keyCode, event);
    }
  ```

  ### 四、启动速度优化

  

  

  

  

  ### 拓展阅读 ###

  - [面试官：今日头条启动很快，你觉得可能是做了哪些优化？](https://juejin.im/post/5d95f4a4f265da5b8f10714b)