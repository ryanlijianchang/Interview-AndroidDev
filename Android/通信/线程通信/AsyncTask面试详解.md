### 一、什么是AsyncTask ###

它的本质就是封装了线程池和Handler的异步框架。

### 二、机制原理 ###

1. AsyncTask的本质是一个静态的线程池，AsyncTask派生出的子类可以实现不同的异步任务，这些任务都是提交到静态的线程池中执行。
2. 线程池中的工作线程执行doInBackground方法执行异步任务。
3. 当任务状态改变后，工作线程会向UI线程发送消息，AsyncTask内部的InternalHandler响应这些消息，并调用相关的回调函数。

### 三、内存泄漏 ###

解决：将AsyncTask设置为静态，并内部持有Activity的弱引用。

### 四、声明周期 ###

AsyncTask的生命周期和Activity生命周期不同步，在不需要时需要调用AsyncTask.cancel方法。如果在Activity结束时不取消AsyncTask有可能会导致崩溃。

### 五、结果丢失 ###

AsyncTask由于持有Activity的引用，如果Activity由于换语言或者横竖屏切换导致重建，会造成AsyncTask引用的Activity不再存在，从而导致结果丢失。

### 六、串行or并行 ###

- 1.6之前，串行
- 1.6-2.3，并行
- 2.3之后，串行，保持稳定