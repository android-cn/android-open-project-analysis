Retrofit 源码解析
----------------
> 本文为 [Android 开源项目实现原理解析](https://github.com/android-cn/android-open-project-analysis) 中 Retrofit 部分  
> 项目地址：[Retrofit](https://github.com/square/retrofit)，分析的版本：，Demo 地址：    
> 分析者：[zonda](https://github.com/zonda)，校对者：，完成状态：未完成   

## 1、什么是Retrofit？
## 2、常用示例
## 3、动态代理
## 4、使用场景
## 5、结语
***
## 1、什么是Retrofit？
按照 [官方文档](http://square.github.io/retrofit/)， Retrofit 是用于 Android 或 Java [REST](http://zh.wikipedia.org/wiki/REST) 客户端的一个类型安全的框架。换句话说，Retrofit 是一个通过 Java 动态代理，针对 REST API 实现的网络请求框架，将 REST API 转化为 Java 的接口。
***  
## 2、常用示例
Retrofit 的使用较为简单，以 RetrofitDemo为例，Retrofit 首先，需要实现一个接口 OpenWeatherApiService，其中每个方法对应一个 API，通过设置返回值类型和是否有 Callback 回调，告知 Retrofit 是否需要异步请求；然后，创建一个 RestAdapter 用于动态生成对应接口 OpenWeatherApiService 的一个实例；最后，使用该实例请求服务器数据。


根据接口设置的返回值类型和参数，Retrofit 的接口方法可分为 3 类，以 OpenWeatherApiService 中的方法为例：


 - “同步”请求，无 Callback 回调，有返回值


```java
    @GET("/history/city")
    public WeatherHistoryResult getHistoryWeather(@Query("q") String queryInfo);
```
 - “异步”请求，有 Callback 回调，无返回值


```java
    @GET("/weather")
    public void getTodayWeather(@Query("q") String queryInfo, @Query("lang") String language,
            Callback<WeatherTodayResult> callback);
```
 - “RxJava”请求，无 Callback 回调，有返回值，且返回值为 Observable 类型，***此时，工程必须包含 [rxjava-core.jar](http://search.maven.org/#search%7Cga%7C1%7Cg%3A%22com.netflix.rxjava%22%20AND%20a%3A%22rxjava-core%22)***
```java
    @GET("/weather")
    public Observable<WeatherTodayResult> getTodayWeather(@QueryMap Map<String, String> params);
```
## 3、动态代理


动态代理即通过实现 InvocationHandler 接口创建自己的调用处理器，同时为Proxy指定 ClassLoader 对象和一组接口来创建动态代理类，Retrofit 的核心便是 Java 的动态代理。实现动态代理有两种方式，代码如下：


```java
(T)Proxy.newProxyInstance(T.class.getClassLoader(),new Class[]{T.class}),new InvocationHanlder()); 
```


```java 
Class proxyClass = Proxy.getProxyClass(T.class.getClassLoader(), new Class[]{T.class}); 
InvocationHandler handler = new InvocationHanlder();
(T)proxyClass.getConstructor(new Class[]{InvactionHandler.class}).newInstance(new Object[]{handler});
```
## 4、使用场景


Retrofit 通过动态代理很好的实现了网络请求和UI的解耦，但是作为一个库他并未提供网络数据存储的部分，对于线程控制也未做过多的优化，因此如有以上需求的项目可能就需要自行扩展相应的功能。此外，对于 [RxJava](https://github.com/Netflix/RxJava/wiki) 是 Netflix 编写的一个库，主要用于改进下载序列的回调时的嵌套带来的麻烦。因此，个人觉得若不是有一个相互关联的任务集合，则无使用 RxJava 的必要。


## 5、结语

以上仅仅是关于 Retrofit 的一个基本介绍，关于 [RxJava](https://github.com/Netflix/RxJava/wiki)、[OkHttp](http://square.github.io/okhttp/) 等相关内容并不在本文范围之内，因此未做过多介绍，如有遗漏或错误请多多指正。

