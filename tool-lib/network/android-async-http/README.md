
android-async-http 源码原理解析
====================================

> 本文为 [Android 开源项目源码解析](http://a.codekk.com) 中 android-async-http 部分  
 项目地址：[android-async-http](https://github.com/loopj/android-async-http)，分析的版本：[1.4.8] ，Demo 地址：[sample](https://github.com/loopj/android-async-http/tree/1.4.8/sample/src/main/java/com/loopj/android/http/sample)    
 分析者：[samuelhehe](https://github.com/samuelhehe)，分析状态：未完成，校对者：[Trinea](https://github.com/trinea)，校对状态：未开始   

# 1. 功能介绍  

## 功能介绍：

android-async-http  是一个基于回调的并基于Apache's httpClient libraries 为Android 开发的一个框架。 所有的请求在非UI线程中处理，而回调是在Handler里面处理。你也可以在Service或者后台线程使用， lib 会自动识别 并在context里处理。  具体使用方法可见 [http://loopj.com/android-async-http/](http://loopj.com/android-async-http/)

## 概念

Http请求 (AsyncHttpRequest)：又可以成为Http请求的单线程对象，通过将该单线程模型submit至 线程池中统一管理。

Http响应处理者(HttpResponseHandler)： 可以针对性的生成不同数据类型的相应处理者，包含Json， String， binary， file ...的处理操作。

重试处理者(RetryHandler)：可以针对不同的异常情况根据自身预设定的white&black exception List 进行智能筛选，选择需要重试的请求。   

拦截器(Interceptor): 在该框架中的应用类似于Spring中的 aop(Aspect Oriented Programming), 多次设定于Httpclient属性中。

## 特点： 

* 异步发送HTTP请求，在回调函数中处理响应
* HTTP请求过程不在UI线程进行
* 使用线程池来管理并发数
* 支持GET/POST请求参数单独设置
* 无需其他库上传序列化JSON数据
* 处理重定向
* 体积小，只有90K
* 针对不同的网络连接对重试次数进行智能优化
* 支持gzip
* 二进制通信协议使用BinaryHttpResponseHandler处理
* 内置Json解析，使用JsonHttpResponseHandler对响应进行处理
* 使用FileAsyncHttpResponseHandler直接将响应保存到文件中
* 动态保存Cookie，将Cookie保存到应用的SharedPreferences中
* 使用BaseJsonHttpResponseHandler可以搭配Jackson JSON，Gson或者其他的Json反序列化库
* 支持SAX解析，使用SaxAsyncHttpResponseHandler
* 支持多语言多种编码方式，不只是UTF-8

## 优点：

个人觉得优点就是 

1. 该框架是基于回调的，各种回调类型的处理都有
2. 直接放到主线程使用匿名调用即可
3. 可以扩展自己的回调处理
4. 支持处理重定向，文档的续传
5. 可以保存Cookie， 可以添加请求凭证保存，单次输入，多次使用
6. 根据常用网络请求异常进行黑白名单查询，针对性重试机构
7. 整个请求处理是使用gzip的，节省流量，速度快 用起来方便。

# 2. 总体设计

## 总体设计图

![art2](https://cloud.githubusercontent.com/assets/5669999/9300689/490ddd0e-44fb-11e5-8381-0a438010371f.png)
图 2-1  总体设计图

以上是android-async-http 的总体设计图。 

* 整个请求子线程 AsyncHttpRequest处于的 参数封装是在主类AsyncHttpClient 中进行的。 处理完成之后将该请求线程 提交至 threadPool 中由线程池调度完成。  

* 请求的处理是在子线程AsyncHttpRequest中处理的，其中包含了ResponseHandlerInterface 的响应回调处理大接口， 也处理了由于各种原因造成的访问失败的智能重试。

* 请求的响应实现ResponseHandlerInterface接口的AsyncHttpResponseHandler类，类似于BaseAdapter模式，基本上使用Handler消息处理了数据处理的各种情况。 其中AsyncHttpResponseHandler的各种子类负责了对应各种数据类型的解析与处理。


# 3. 流程图

### 3.1. 总体流程图
![art_flow](https://cloud.githubusercontent.com/assets/5669999/9241562/e5bda17c-41a7-11e5-837b-2030d8866e99.png)

图 3-1 总体流程图

### 3.2. 请求线程流程图
![thread](https://cloud.githubusercontent.com/assets/5669999/9300719/a3af1232-44fb-11e5-87cb-3022c476d863.png)

图 3-2 请求线程流程图

### 3.3. makeRequestWithRetries函数模块调用

![methodreq](https://cloud.githubusercontent.com/assets/5669999/9300718/a36420d8-44fb-11e5-96bc-de0274203d09.png)

图 3-3 请求函数模块调用

### 3.4. makeRequest函数模块调用

![method2](https://cloud.githubusercontent.com/assets/5669999/9300720/a3e84430-44fb-11e5-9d32-bc5b89e0ddb4.png)

图 3-4 请求函数模块调用 


### 3.5 整体回调顺序
![callbackseq](https://cloud.githubusercontent.com/assets/5669999/9326673/118d0980-45ce-11e5-869f-985ac5ad739f.png)

图 3-5 请求函数整体回调顺序


# 4. 详细设计

## 4.1 总类关系图 
![art](https://cloud.githubusercontent.com/assets/5669999/9300898/08c27fbe-44fd-11e5-8ed0-8b76a6d0bf02.png)
图 4-1

## 4.2 核心类功能介绍

### 4.2.1 类详细介绍(未完成)

1. ResponseHandlerInterface 整个框架中网络访问返回数据的处理接口，集合了包括消息发送，数据处理回调在内的共16个函数。

2. AsyncHttpClient.class 整个框架的调用集合类与各个参数的组装调用类。

3. AsyncHttpRequest.class 网络访问子线程类

4. AsyncHttpResponseHandler.class 网络响应处理主类 

**4.1  AsyncHttpResponseHandler 类图** 

![responsehandler](https://cloud.githubusercontent.com/assets/5669999/9345981/813c2592-4649-11e5-980b-3f84a14a393e.png)

图 4.1-1 AsyncHttpResponseHandler 类图

**4.2 成员变量**

```
   
    private Handler handler;  /// 内部类 ResponderHandler 的赋值对象， 负责发送，处理消息。
   
    private boolean useSynchronousMode; 
   
    private boolean usePoolThread;

    private URI requestURI = null;
    
    private Header[] requestHeaders = null;  /// 请求头信息
    
    private Looper looper = null; /// 默认如果looper为null，则赋值为Looper.myLooper() 也就是获取的主线程中的Looper ;

```

**4.3 重要函数**

```

// Methods which emulate android's Handler and Message methods
   
    protected void handleMessage(Message message) {
       
        Object[] response;

        try {
            switch (message.what) {
                case SUCCESS_MESSAGE:
                    response = (Object[]) message.obj;
                    if (response != null && response.length >= 3) {
                        onSuccess((Integer) response[0], (Header[]) response[1], (byte[]) response[2]);
                    } else {
                        Log.e(LOG_TAG, "SUCCESS_MESSAGE didn't got enough params");
                    }
                    break;
                case FAILURE_MESSAGE:
                    response = (Object[]) message.obj;
                    if (response != null && response.length >= 4) {
                        onFailure((Integer) response[0], (Header[]) response[1], (byte[]) response[2], (Throwable) response[3]);
                    } else {
                        Log.e(LOG_TAG, "FAILURE_MESSAGE didn't got enough params");
                    }
                    break;
                case START_MESSAGE:
                    onStart();
                    break;
                case FINISH_MESSAGE:
                    onFinish();
                    break;
                case PROGRESS_MESSAGE:
                    response = (Object[]) message.obj;
                    if (response != null && response.length >= 2) {
                        try {
                            onProgress((Long) response[0], (Long) response[1]);
                        } catch (Throwable t) {
                            Log.e(LOG_TAG, "custom onProgress contains an error", t);
                        }
                    } else {
                        Log.e(LOG_TAG, "PROGRESS_MESSAGE didn't got enough params");
                    }
                    break;
                case RETRY_MESSAGE:
                    response = (Object[]) message.obj;
                    if (response != null && response.length == 1) {
                        onRetry((Integer) response[0]);
                    } else {
                        Log.e(LOG_TAG, "RETRY_MESSAGE didn't get enough params");
                    }
                    break;
                case CANCEL_MESSAGE:
                    onCancel();
                    break;
            }
        } catch(Throwable error) {
            onUserException(error);
        }
    }

```

分析：

* 接收处理 请求访问以及返回过程中的所有状态， 包含  SUCCESS_MESSAGE, FAILURE_MESSAGE, START_MESSAGE, FINISH_MESSAGE, PROGRESS_MESSAGE,RETRY_MESSAGE, CANCEL_MESSAGE 在内的各种情况，以及回调。 

```

    byte[] getResponseData(HttpEntity entity) throws IOException {
        byte[] responseBody = null;
        if (entity != null) {
            InputStream instream = entity.getContent();
            if (instream != null) {
                long contentLength = entity.getContentLength();
                if (contentLength > Integer.MAX_VALUE) {
                    throw new IllegalArgumentException("HTTP entity too large to be buffered in memory");
                }
                int buffersize = (contentLength <= 0) ? BUFFER_SIZE : (int) contentLength;
                try {
                    ByteArrayBuffer buffer = new ByteArrayBuffer(buffersize);
                    try {
                        byte[] tmp = new byte[BUFFER_SIZE];
                        long count = 0;
                        int l;
                        // do not send messages if request has been cancelled
                        while ((l = instream.read(tmp)) != -1 && !Thread.currentThread().isInterrupted()) {
                            count += l;
                            buffer.append(tmp, 0, l);
                            sendProgressMessage(count, (contentLength <= 0 ? 1 : contentLength));
                        }
                    } finally {
                        AsyncHttpClient.silentCloseInputStream(instream);
                        AsyncHttpClient.endEntityViaReflection(entity);
                    }
                    responseBody = buffer.toByteArray();
                } catch (OutOfMemoryError e) {
                    System.gc();
                    throw new IOException("File too large to fit into available memory");
                }
            }
        }
        return responseBody;
    }

```

分析：

*  sendMessage中包含了 所有 的发送消息通用函数， 调用handler的消息发送函数，send Or post 至ResponderHandler 内部， ResponderHandler内部调用AsyncHttpResponseHandler 内部 handleMessage 函数，来处理各种情况。
 
```

   protected void sendMessage(Message msg) ;
   
   protected void postRunnable(Runnable runnable);
   
```

* 接收返回HttpEntity 实体内部 content ， 并按照BufferSize来读取数据， append至 ByteArrayBuffer 内部， 同时发布进度状态sendProgressMessage ，回调onProgress 函数。 最后返回 ByteArrayBuffer 的byte[] . 

NOTE： 其中ByteArrayBuffer 属于自增长型的缓冲数据类型， code中 的默认大小为4096也就是 1KB大小 ,  而ByteArrayBuffer 过大也会导致 OutOfMemoryError ， 超过 VM heapsize 的分配内存之后就会报内存溢出异常。  

```
 		if (contentLength > Integer.MAX_VALUE) {
                    throw new IllegalArgumentException("HTTP entity too large to be buffered in memory");
        }
                
```

后边 Int 的数据类型范围：  -2147483648~2147483647 最大值 2147483647 = 2^10 也就是 1G ， 上边code写的， 也就是最大不能超1GB ， 这样的话但是后边的强转之后直接将int设置到ByteArrayBuffer中， 显然作者是将这个问题交给了系统自动去检测了。 


```
	Runtime rt = Runtime.getRuntime();
	long maxMemory = rt.maxMemory();
	Log.v("onCreate", "maxMemory:" + Long.toString(maxMemory));

```

个人认为，由于每一种机型的型号不同，系统分配的 HeapSize也不同，  可以根据这个来进行匹配，检测。
 链接见： 
[http://stackoverflow.com/questions/2630158/detect-application-heap-size-in-android/9428660#9428660](http://stackoverflow.com/questions/2630158/detect-application-heap-size-in-android/9428660#9428660)

[http://stackoverflow.com/questions/9818407/out-of-memory-error-in-android-due-to-heap-size-increasing](http://stackoverflow.com/questions/9818407/out-of-memory-error-in-android-due-to-heap-size-increasing)


5. RequestParams  请求参数的组装，实现Serializable 接口。 



6. RetryHandler.class 重试处理类

**6.1 成员变量**

黑白名单列表：

```
	 HashSet<Class<?>> exceptionWhitelist = new HashSet<Class<?>>();
	 
     HashSet<Class<?>> exceptionBlacklist = new HashSet<Class<?>>();

```

默认黑白名单处理策略：

```
 static {
 
        // Retry if the server dropped connection on us
        exceptionWhitelist.add(NoHttpResponseException.class);
        
        // retry-this, since it may happens as part of a Wi-Fi to 3G failover
        exceptionWhitelist.add(UnknownHostException.class);
        
        // retry-this, since it may happens as part of a Wi-Fi to 3G failover
        exceptionWhitelist.add(SocketException.class);
        
        // never retry timeouts
        exceptionBlacklist.add(InterruptedIOException.class);
        
        // never retry SSL handshake failures
        exceptionBlacklist.add(SSLException.class);
    }

```

**6.2 重要函数**

```

public boolean retryRequest(IOException exception, int executionCount, HttpContext context){

 		boolean retry = true;
 		
        Boolean b = (Boolean) context.getAttribute(ExecutionContext.HTTP_REQ_SENT);
        
        boolean sent = (b != null && b);
        
        if (executionCount > maxRetries) {
        
            // Do not retry if over max retry count
            retry = false;
            
        } else if (isInList(exceptionWhitelist, exception)) {
        
            // immediately retry if error is whitelisted
            retry = true;
            
        } else if (isInList(exceptionBlacklist, exception)) {
        
            // immediately cancel retry if the error is blacklisted
            retry = false;
            
        } else if (!sent) {
        
            // for most other errors, retry only if request hasn't been fully sent yet
            retry = true;
        }
        if (retry) {
        
            // resend all idempotent requests
            HttpUriRequest currentReq = (HttpUriRequest) context.getAttribute(ExecutionContext.HTTP_REQUEST);
            
            if (currentReq == null) {
                return false;
            }
        }
        
        if (retry) {
            SystemClock.sleep(retrySleepTimeMS);
            
        } else {
        
            exception.printStackTrace();
        }
        
        return retry;
    }

```
             
分析：
整体异常处理策略就是，根据参数IOException 的类型，在对应的黑白名单列表中进行轮询操作，如果发现匹配项目，则进行相应的操作， 例如： 如果异常出现在白名单列表中，也就是在网络访问中常发生的，不可避免的，由用户网络环境造成的异常， 则返回可以重试标识。反之，则结束重试。

**6.3 其他函数**

添加黑白名单的方法，轮询方法等。

```

addClassToWhitelist(Class<?> cls);

addClassToBlacklist(Class<?> cls);

boolean isInList(HashSet<Class<?>> list, Throwable error);

```

7. RequestHandle 请求句柄类，例如 请求任务取消，是否完成，垃圾回收操作，  也挺重要的。 

**7.1成员变量**

这个类貌似只有这么一个成员变量 就是 AsyncHttpRequest 的弱引用， 看来作者对于内存溢出考虑也挺周全。 

```

private final WeakReference<AsyncHttpRequest> request; 

```

**7.2重要函数**

```

 public boolean cancel(final boolean mayInterruptIfRunning) {
        final AsyncHttpRequest _request = request.get();
        if (_request != null) {
            if (Looper.myLooper() == Looper.getMainLooper()) {
                new Thread(new Runnable() {
                    @Override
                    public void run() {
                        _request.cancel(mayInterruptIfRunning);
                    }
                }).start();
            } else {
                _request.cancel(mayInterruptIfRunning);
            }
        }
        return false;
    }

```

分析：

取消请求，直接将AsyncHttpRequest 内部的iscancel 置为false,   调用HttpUriRequest的abort 函数， 中断请求，发送任务取消Notification。 

NOTE : 这里我发现 mayInterruptIfRunning参数木有用到，我找了半天没有找到。 

**7.3 其他函数**

```
 	 public boolean isFinished() ; // 判断是否请求结束。 
 
     public boolean isCancelled() ; // 判断任务是否已经取消过了。
     
     public boolean shouldBeGarbageCollected() // 这货好像是这个版本才出现的， 作用是 告诉 gc要准备收回该对象占用内存

```

8. SyncHttpClient.class  异步网络请求类



9. PersistentCookieStore 本地的cookie的存储类  

**9.1 成员变量**

```
    private final ConcurrentHashMap<String, Cookie> cookies; /// 运行时维护的一个cookie的Map

    private final SharedPreferences cookiePrefs; /// cookie 保存在了sharedPreference中

```

**9.2 成员函数**

```
    public void addCookie(Cookie cookie) ;/// 添加cookie
    
    public void clear() ; /// 清除cookie
        
	public boolean clearExpired(Date date);  // 清除过期的cookie 
	
	public void deleteCookie(Cookie cookie); // 删除指定名称的cookie 从sharedPreference 

	protected Cookie decodeCookie(String cookieString); // 将cookie以16进制编码的方式通过 SerializableCookie readObject(ObjectInputStream in)函数添加至Cookie中，返回Cookie
	
	protected String encodeCookie(SerializableCookie cookie); // 将Cookie对象序列化至String中

```


13. DataAsyncHttpResponseHandler 数据的异步下载处理



14. FileAsyncHttpResponseHandler 文件的异步下载处理

**14.1成员变量**

```
    protected final File mFile;  /// 临时文档的操作对象。
    protected final boolean append; /// 文档操作对象是否在上一次写的基础上追加数据。 

```

**14.2重要函数**

```

	@Override
    protected byte[] getResponseData(HttpEntity entity) throws IOException {
        if (entity != null) {
            InputStream instream = entity.getContent();
            long contentLength = entity.getContentLength();
            FileOutputStream buffer = new FileOutputStream(getTargetFile(), this.append);
            if (instream != null) {
                try {
                    byte[] tmp = new byte[BUFFER_SIZE];
                    int l, count = 0;
                    // do not send messages if request has been cancelled
                    while ((l = instream.read(tmp)) != -1 && !Thread.currentThread().isInterrupted()) {
                        count += l;
                        buffer.write(tmp, 0, l);
                        sendProgressMessage(count, contentLength);
                    }
                } finally {
                    AsyncHttpClient.silentCloseInputStream(instream);
                    buffer.flush();
                    AsyncHttpClient.silentCloseOutputStream(buffer);
                }
            }
        }
        return null;
    }

```

分析：
该方法是处理接收的HttpEntiry 的Content 内容，使用流写至一个临时的file， 同时使用父类AsyncHttpResponseHandler的Handler 发送进度， 最后关闭流。

**14.3 其他函数**

```

public boolean deleteTargetFile() ;// 删除目标文件；

protected File getTemporaryFile(Context context);// 获取临时创建的文件；

```

以及其他父类 AsyncHttpResponseHandler中含有的状态回调函数。

15. BinaryHttpResponseHandler  二进制数据的下载处理

16. TextHttpResponseHandler 文本数据的加载处理

**16.1 成员变量**

```
    private String responseCharset = DEFAULT_CHARSET;/// 相应数据编码，默认UTF-8
    
```

**16.2 成员函数**

```

  public static String getResponseString(byte[] stringBytes, String charset) {
        try {
            String toReturn = (stringBytes == null) ? null : new String(stringBytes, charset);
            if (toReturn != null && toReturn.startsWith(UTF8_BOM)) {
                return toReturn.substring(1);
            }
            return toReturn;
        } catch (UnsupportedEncodingException e) {
            Log.e(LOG_TAG, "Encoding response into string failed", e);
            return null;
        }
    }
    
```

分析：
该方法是处理接收byte[] , 通过使用父类AsyncHttpResponseHandler的Handler 中onSuccess方法回传的byte[] ,添加指定的charset(默认编码UTF-8),返回从第一个字符开始截取的string对象。

NOTE：中间有个常量 UTF8_BOM，内容是 发现BOM的作用是识别UTF-8编码的作用，具体请参考链接：[utf-8与utf-8(无BOM)的区别](http://afericazebra.blog.163.com/blog/static/30050408201211199298711/)

17. JsonHttpResponseHandler Json数据的加载解析处理

18. BaseJsonHttpResponseHandler 基础Json数据的解析，继承自TextHttpResponseHandler

**18.1 成员变量**
无

**18.2 重要函数**

```
     public final void onSuccess(final int statusCode, final Header[] headers, final String responseString) {
        if (statusCode != HttpStatus.SC_NO_CONTENT) {
            Runnable parser = new Runnable() {
                @Override
                public void run() {
                    try {
                        final JSON_TYPE jsonResponse = parseResponse(responseString, false);
                        postRunnable(new Runnable() {
                            @Override
                            public void run() {
                                onSuccess(statusCode, headers, responseString, jsonResponse);
                            }
                        });
                    } catch (final Throwable t) {
                        Log.d(LOG_TAG, "parseResponse thrown an problem", t);
                        postRunnable(new Runnable() {
                            @Override
                            public void run() {
                                onFailure(statusCode, headers, t, responseString, null);
                            }
                        });
                    }
                }
            };
            if (!getUseSynchronousMode() && !getUsePoolThread()) {
                new Thread(parser).start();
            } else {
                // In synchronous mode everything should be run on one thread
                parser.run();
            }
        } else {
            onSuccess(statusCode, headers, null, null);
        }
    }

```

分析：

使用TextHttpResponseHandler 中onSuccess的Text返回方法， 在返回String内容不为null的情况下， 直接解析responseString, 并调用抽象函数parseResponse ，解析的过程在子线程中运行，这取决于用户的是操作方式，是异步还是同步.

**18.3 其他函数**

```

protected abstract JSON_TYPE parseResponse(String rawJsonData, boolean isFailure); // 需要子类来实现具体的解析方式.

```

19. JsonStreamerEntity Json流的处理，该类适合Json base 64 编码的上传操作， 节省内存，可以适合大文件。


20. SaxAsyncHttpResponseHandler xml 的解析处理，继承自AsyncHttpResponseHandler 使用的sax方法解析。


21. PreemptiveAuthorizationHttpRequestInterceptor  预验证拦截器类 

22. MyRedirectHandler 重定向处理器类， 源码中标识该类引用自stackoverflow <a href="https://stackoverflow.com/questions/3420767/httpclient-redirecting-to-url-with-spaces-throwing-exception">https://stackoverflow.com/questions/3420767/httpclient-redirecting-to-url-with-spaces-throwing-exception</a>

23. Base64 二进制数据的 Base 64 的编码与解码

24. Base64DataException Base 64 编码的操作异常类

25. Base64OutputStream Base 64 输出流操作类

26. HttpDelete HttpDelete 操作类，继承自HttpEntityEnclosingRequestBase类

27. HttpGet HttpGet 操作类，继承自HttpEntityEnclosingRequestBase类

28. HttpPatch HttpPatch 操作类，继承自HttpEntityEnclosingRequestBase类

29. JsonValueInterface 接口，可以中App来封装完整的封装JSON的值

30. SerializableCookie 配角， 在PersistentCookieStore类中序列化Cookie的值时使用

31. SimpleMultipartEntity 简单的多部分实体，主要用于发送一个或多个文档

32. Utils 工具类 


# 5. 杂谈总结

第一次大胆分析大神的源码，整个分析过程一共使用了三天8月6号就完成了,只是一直忙于工作没有上传,十分抱歉，文章中包括与设计图的制作，中间还有一些部分不太了解，有很多不对的地方，请大家批评指正 。框架整个设计框架巧妙的使用了基于Apache HttpClient框架的回调。充分使用了Httpclient 中的各种 handler ， interceptor 将整个数据响应过程通过接口使其可扩展化，过程化，结构化。 而框架整体看起来又浑然一体，的确很牛气。
