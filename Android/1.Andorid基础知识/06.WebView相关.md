### 一、WebView常见的坑 ###

1. Android API 16以及之前的版本存在远程代码执行的安全漏洞，该漏洞源于程序没有正确限制使用WebView.addJavascriptInteface方法，远程攻击者可以通过使用Java Reflection API利用该漏洞执行任意Java对象的方法。
2. webview在布局文件中使用，webview写入其他容器时，在Activiity的onDestroy时，需要先remove掉webview，再调用webview的destroy方法。
3. jsbridge
4. webviewClient.onPageFinished会在页面跳转时调用很多次，建议使用webChromeClient.onProgressChanged方法。   
5. 后台耗电。在开启webview之后，webview会启动子线程，所以当页面关闭时，需要清理这些残留线程，避免耗电。
6. webview硬件加速导致页面渲染问题。


### 二、Webview的内存泄漏 ###

1. 独立进程，可能涉及进程通信问题
2. 动态添加webview，对传入webview的context使用弱引用，在activity创建时动态添加webview，停止时先remove掉webview再destory掉webview。