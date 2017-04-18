Android 开源项目源码解析细节点
====================================
> 本文为 [Android 开源项目源码解析](http://a.codekk.com) 中 细节点 部分  
> 分析者：[Trinea](https://github.com/trinea)，校对者：[Trinea](https://github.com/trinea)，校对状态：未完成

### 1. Volley 细节点
#### (1) HttpURLConnection 与 HttpClient

### 2. Android Universal Image Loader 细节点
#### (1) FlushedInputStream.java
为了解决早期 Android 版本`BitmapFactory.decodeStream(…)`在慢网络情况下 decode image 异常的 Bug。  
主要通过重写`FilterInputStream`的 skip(long n) 函数解决，确保 skip(long n) 始终跳过了 n 个字节。如果返回结果即跳过的字节数小于 n，则不断循环直到 skip(long n) 跳过 n 字节或到达文件尾。  
见：http://code.google.com/p/android/issues/detail?id=6066

### (2). BaseImageDownloader.getStreamFromNetwork(String imageUri, Object extra)
通过`HttpURLConnection`从网络获取图片的`InputStream`。支持 response code 为 3xx 的重定向。这里有个小细节代码如下：  
```java
try {
    imageStream = conn.getInputStream();
} catch (IOException e) {
    // Read all data to allow reuse connection (http://bit.ly/1ad35PY)
    IoUtils.readAndCloseStream(conn.getErrorStream());
    throw e;
}
```
在发生异常时会调用`conn.getErrorStream()`继续读取 Error Stream，这是为了利于网络连接回收及复用。但有意思的是在 Froyo(2.2) 之前，HttpURLConnection 有个重大 Bug，调用 close() 函数会影响连接池，导致连接复用失效，不少库通过在 2.3 之前使用 AndroidHttpClient 解决这个问题。  
见：http://docs.oracle.com/javase/6/docs/technotes/guides/net/http-keepalive.html  

#### (3). ViewAware.ViewAware(View view, boolean checkActualViewSize)
构造函数。  
`view`表示需要显示图片的对象。  
`checkActualViewSize`表示通过`getWidth()`和`getHeight()`获取图片宽高时返回真实的宽和高，还是`LayoutParams`的宽高，true 表示返回真实宽和高。  
如果为`true`会导致一个问题，`View`在还没有初始化完成时加载图片，这时它的真实宽高为0，会取它`LayoutParams`的宽高，而图片缓存的 key 与这个宽高有关，所以当`View`初始化完成再次需要加载该图片时，`getWidth()`和`getHeight()`返回的宽高都已经变化，缓存 key 不一样，从而导致缓存命中失败会再次从网络下载一次图片。可通过`ImageLoaderConfiguration.Builder.denyCacheImageMultipleSizesInMemory()`设置不允许内存缓存缓存一张图片的多个尺寸。  
见：https://github.com/nostra13/Android-Universal-Image-Loader/issues/376  