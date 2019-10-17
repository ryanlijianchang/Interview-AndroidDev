### 使用简介 ###


```
// 同步获取数据
private void okHttpGet() {
    OkHttpClient client = new OkHttpClient();
    Request request = new Request.Builder()
            .url("https://test.com/testdata.txt")
            .build();
    Response res = client.newCall(request).execute();
}

private void okHttpGetSync() {
    OkHttpClient client = new OkHttpClient();
    Request request = new Request.Builder()
        .url("https://test.com/testdata.txt")
        .build();
    client.newCall(requst).enqueue(new Callback() {
        @override
        public void onFailure(Call call, IOException e) {
            
        }
    
        @override
        public void onResponse(Call call, Response response)  {
           
        }
    });
}

```

### 设计架构图 ###

![image](https://upload-images.jianshu.io/upload_images/5982616-c9b805378329b087.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

![image](https://upload-images.jianshu.io/upload_images/5982616-a44688d678695fd0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

### 常见问题 ###

- **为什么okHttp3好用呢？**

    OkHttp是一个精巧的网络请求库,有如下特性: 
    - 1)支持http2，对一台机器的所有请求共享同一个socket
    - 2)内置连接池，支持连接复用，减少延迟
    - 3)支持透明的gzip压缩响应体
    - 4)通过缓存避免重复的请求
    - 5)请求失败时自动重试主机的其他ip，自动重定向
    - 6)好用的API