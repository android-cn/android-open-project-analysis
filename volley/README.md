Volley 实现原理解析
====================================
> 本文为 [Android 开源项目实现原理解析](https://github.com/android-cn/android-open-project-analysis) 中 Volley 部分  
> 项目地址：[Volley](https://android.googlesource.com/platform/frameworks/volley/)，分析的版本：[35ce778](https://android.googlesource.com/platform/frameworks/volley/+/35ce77836d8e1e951b8e4b2ec43e07fb7336dab6)，Demo 地址：[Volley Demo](https://github.com/android-cn/android-open-project-demo/tree/master/volley-demo)    
> 分析者：[grumoon](https://github.com/grumoon)，校对者：[promeG](https://github.com/promeG)，校对状态：未完成   


###1. 功能介绍  
Volley是Google推出的Android异步网络调用框架和图片加载框架。在Google I/O 2013大会上发布。
> 名字由来：a burst or emission of many things or a large amount at once  
> 发布演讲时候的配图  
> ![Volley](image/volley.png)

从名字由来和配图中无数急促的火箭可以看出Volley的特点：特别适合**数据量小，通信频繁**的网络操作。（个人认为Android应用中绝大多数的网络操作都属于这种类型）。

**Volley的主要特点**
> 1.默认Android2.3及以上基于HttpURLConnection，2.3以下使用基于HttpClient   
> 2.符合Http **缓存语义** 的缓存机制   
> 3.请求队列的优先级排序   
> 4.提供多样的取消机制   
> 5.提供简便的图片加载工具    

###2. 详细设计
###2.1 核心类功能介绍
####2.1.1 Volley.java 
这个和Volley框架同名的类，其实是个工具类。作用是帮助构建一个RequestQueue对象。有两个重载的静态方法。
```java
public static RequestQueue newRequestQueue(Context context)

public static RequestQueue newRequestQueue(Context context, HttpStack stack)
```
第一个方法的实现调用第二个方法，传HttpStack参数为null。
第二个方法中，如果HttpStatck为null，则默认情况下如果系统版本大于等于9，采用基于UrlConnection的HrulStack，如果小于9，采用基于HttpClient的HttpClientStack。
```java
if (stack == null) {
    if (Build.VERSION.SDK_INT >= 9) {
        stack = new HurlStack();
    } else {
        stack = new HttpClientStack(AndroidHttpClient.newInstance(userAgent));
    }
}
```

得到了HttpStack,然后通过它构造一个代表网络（Network）的实现BasicNetwork。
接着构造一个代表缓存（Cache）的基于Disk的实现DiskBasedCache。
最后将网络（Network）对象和缓存（Cache）对象传入构建一个RequestQueue，启动这个RequestQueue，并返回。
```java
Network network = new BasicNetwork(stack);
RequestQueue queue = new RequestQueue(new DiskBasedCache(cacheDir), network);
queue.start();
return queue;
```
> 我们平时大多采用`Volly.newRequestQueue(context)`的默认实现，构建RequestQueue。
> 通过源码可以看出，我们可以抛开Volley工具类构建自定义的RequestQueue，采用自定义的HttpStatck，采用自定义的Network实现，采用自定义的Cache实现等来构建RequestQueue。
**优秀框架的高可拓展性的魅力来源于此啊**

####2.1.2 Request.java
代表一个网络请求的抽象类。我们通过构建Request类的具体实现（StringRequest,JsonRequest,ImageRequest等），并且将其加入到RequestQueue中来完成网络请求操作。

Volley支持8个http请求方法**GET,POST,PUT,DELETE,HEAD,OPTIONS,TRACE,PATCH**
Request类中包含了，请求url，请求方法，请求Header，请求Body，请求的优先级等信息。

**因为是抽象类，子类必须重写的两个方法。**
```java
abstract protected Response<T> parseNetworkResponse(NetworkResponse response);
```
子类重写此方法，将网络返回的原生字节内容，转换成合适的类型。此方法会在工作线程中被调用。


```java
abstract protected void deliverResponse(T response);
```
子类重写此方法，将解析成合适类型的内容传递给它们的监听回调。

**以下两个方法也经常会被重写**
```java
protected Map<String, String> getParams()
```
重写这个方法，可构建用于POST或者PUT的键值对参数。
```java
public byte[] getBody()
```
重写此方法，可以构建POST或者PUT的最终Body字节内容。


####2.1.3 RequestQueue.java
Volley核心类，将请求Request请求加入到一个运行的RequestQueue中，来完成请求操作。
####(1)主要成员变量
RequestQueue中维护了两个基于优先级的Request队列，缓存请求队列和网络请求队列.
```java
private final PriorityBlockingQueue<Request<?>> mCacheQueue = new PriorityBlockingQueue<Request<?>>();
private final PriorityBlockingQueue<Request<?>> mNetworkQueue = new PriorityBlockingQueue<Request<?>>();
```
维护了一个当前正在处理的请求的集合
```java
private final Set<Request<?>> mCurrentRequests = new HashSet<Request<?>>();
```
维护了一个等待请求的集合
如果一个请求正在被处理并且可以被缓存，后续的相同url的请求，将进入等待队列。
```java
private final Map<String, Queue<Request<?>>> mWaitingRequests = new HashMap<String, Queue<Request<?>>>();
```

####(2)启动队列
new出RequestQueue以后，必须start，然后才能加入请求。

```java
/**
 * Starts the dispatchers in this queue.
 */
public void start() {
    stop();  // Make sure any currently running dispatchers are stopped.
    // Create the cache dispatcher and start it.
    mCacheDispatcher = new CacheDispatcher(mCacheQueue, mNetworkQueue, mCache, mDelivery);
    mCacheDispatcher.start();

    // Create network dispatchers (and corresponding threads) up to the pool size.
    for (int i = 0; i < mDispatchers.length; i++) {
        NetworkDispatcher networkDispatcher = new NetworkDispatcher(mNetworkQueue, mNetwork,
                mCache, mDelivery);
        mDispatchers[i] = networkDispatcher;
        networkDispatcher.start();
    }
}
```
start操作，开启一个**缓存调度线程**和默认的4个**网络调度线程**。
缓存调度线程不断的从缓存请求队列中取出Request去处理，网络调度线程不断的从网络请求队列中取出Request去处理。
####(3)加入请求
####(4)请求完成
####(5)请求取消
####2.1.4 CacheDispatcher.java
缓存调度线程类，不断的从缓存请求队列中取出Request处理。
####(1)成员变量
####(2)处理流程
####2.1.5 NetworkDispatcher.java
网络调度线程类，不断的从网络请求队列中取出Request处理。
####(1)成员变量
####(2)处理流程

####2.1.6 Cache.java
缓存接口
####2.1.7 DiskBasedCache.java
Volley中基于Disk的缓存实现类
####2.1.8 NoCache.java
不做任何操作的缓存实现类

####2.1.9 Network.java
代表网络的接口
唯一的方法，用于执行特定请求
```java
public NetworkResponse performRequest(Request<?> request) throws VolleyError;
```
####2.1.10 NetworkResponse.java
Network中方法performRequest的返回值。
封装了网络请求响应的StatusCode，Headers和Body等。
成员变量：
`int statusCode` Http状态码
`byte[] data` Body数据
`Map<String, String> headers` 响应Headers
`boolean notModified` 表示是否为304响应
`long networkTimeMs` 请求耗时
####2.1.11 BasicNetwork.java
Volley中默认的网络接口实现类


####2.1.12 HttpStack.java
代表Http请求栈的接口
####2.1.13 HttpClientStack.java
基于HttpClient的http栈的实现类
####2.1.14 HurlStack.java
基于urlconnection的http栈的实现类

####2.1.15 Response.java
封装了经过解析后的数据，请求调度线程向主线程传输用途

####2.1.16 PoolingByteArrayOutputStream.java
集成自ByteArrayOutputStream
####2.1.17 ByteArrayPool.java
ByteArray 池
####2.1.18 HttpHeaderParser.java
Http header的解析工具类
####2.1.19 RetryPolicy.java
重试策略接口
####2.1.20 DefaultRetryPolicy.java
默认的重试策略实现类
####2.1.21 ResponseDelivery.java
请求结果的传输接口
####2.1.22 ExecutorDelivery.java
请求结果传输实现类
####2.1.23 StringRequest.java
继承Request类,代表了一个返回值为String的请求。
将网络返回结果，解析为String。
####2.1.24 JsonRequest.java
抽象类，继承自Request，代表了JSON请求。
####2.1.25 JsonObjectRequest.java
继承自JsonRequest，将网络返回值解析为JsonObject。
####2.1.26 JsonArrayRequest.java
继承自JsonRequest，将网络返回值解析为JsonArray。
####2.1.27 ImageRequest.java
继承Request类,代表了一个返回值为Image的请求。
将网络返回结果，Bitmap。
####2.1.28 ImageLoader.java
封装了了ImageRequst的方便使用的工具类
####2.1.29 NetworkImageView.java
可以记在网络图片的ImageView
####2.1.30 ClearCacheRequest.java
用于人为清空Http缓存的请求，添加到RequestQueue后能很快执行，因为优先级很高，为`Priority.IMMEDIATE`
####2.1.31 RequestFuture.java
代表了一个Volley Request的Future
####2.1.32 Authenticator.java
Http认证交互接口，用于基本认证或者摘要认证
####2.1.33 AndroidAuthenticator.java
默认的Android认证交互实现类
####2.1.34 VolleyLog.java
Volley的Log工具类
####2.1.35 VolleyError.java
Volley中所有错误异常的父类，继承自Exception，可通过此类设置和获取NetworkResponse或者请求的耗时。
####2.1.36 AuthFailureError.java
继承自VelleyError，代表认证失败错误。
####2.1.37 NetworkError.java
继承自VolleyError，代表网络错误。
####2.1.38 ParseError.java
继承自VolleyError，代表内容解析错误。
####2.1.49 ServerError.java
继承自VolleyError，代表服务端错误。
####2.1.40 TimeoutError.java
继承自VolleyError，代表请求超时错误。
####2.1.41 NoConnectionError.java
继承自NetworkError，代表无法建立连接错误。
###2.2 类关系图

###3. 流程图
主要功能流程图  
- 如 Retrofit、Volley 的请求处理流程，Android-Universal-Image-Loader 的图片处理流程图  
- 可使用 StartUML、Visio 或 Google Drawing 等工具完成，其他工具推荐？？  
- 非所有项目必须，不需要的请先在群里反馈  

**完成时间**  
- `两天内`完成  

###4. 总体设计
整个库分为哪些模块及模块之间的调用关系。  
- 如大多数图片缓存会分为 Loader 和 Processer 等模块。  
- 可使用 StartUML、Visio 或 Google Drawing 等工具完成，其他工具推荐？？  
- 非所有项目必须，不需要的请先在群里反馈。  

**完成时间**  
- `两天内`完成  

###5. 杂谈
该项目存在的问题、可优化点及类似功能项目对比等，非所有项目必须。  

**完成时间**  
- `两天内`完成  

###6. 修改完善  
在完成了上面 5 个部分后，移动模块顺序，将  
`2. 详细设计` -> `2.1 核心类功能介绍` -> `2.2 类关系图` -> `3. 流程图` -> `4. 总体设计`  
顺序变为  
`2. 总体设计` -> `3. 流程图` -> `4. 详细设计` -> `4.1 类关系图` -> `4.2 核心类功能介绍`  
并自行校验优化一遍，确认无误后，让`校对 Buddy`进行校对，`校对 Buddy`校队完成后将  
`校对状态：未完成`  
变为：  
`校对状态：已完成`  

**完成时间**  
- `两天内`完成  

**到此便大功告成，恭喜大家^_^**  

并自行校验优化一遍，确认无误后，让`校对 Buddy`进行校对，`校对 Buddy`校队完成后将  
`校对状态：未完成`  
变为：  
`校对状态：已完成`  

**完成时间**  
- `两天内`完成  

**到此便大功告成，恭喜大家^_^**  
