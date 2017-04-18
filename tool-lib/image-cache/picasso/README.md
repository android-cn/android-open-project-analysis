Picasso 源码分析
====================================
> 本文为 [Android 开源项目源码解析](http://a.codekk.com) 中 Picasso 部分  
> 项目地址：[Picasso](https://github.com/square/picasso)，分析的版本：[10bac5b](https://github.com/square/picasso/commit/10bac5ba59c7379eb4be4a2e7af074edc4bd9200)，Demo 地址：[Picasso Demo](https://github.com/aosp-exchange-group/android-open-project-demo/tree/master/picasso-demo)  
> 分析者：[Trinea](https://github.com/Trinea)，分析状态：进行中。校对者：，校对状态：未开始

### 1. 功能介绍
Picasso 是 Square 开源的图片缓存库，主要特点有：  
* 包含内存缓存和磁盘缓存两级缓存。  
* 在 Adapter 中自动处理 ImageView 的缓存并且取消之前的图片下载任务。  
* 方便进行图片转换处理。  

Picasso 优点：
1. 多种图片类型处理
2. 自带统计监控功能
3. 支持优先级处理
4. 支持延迟到图片尺寸计算完成加载
支持飞行模式、并发线程数根据网络类型而变 1.2.3.4

### 4. 详细设计
4.1 Picasso.java
Picasso 通过 `with(context)` 得到全局单例，默认单例内存缓存为 LRU，磁盘缓存不超过 50M，三个并发同时进行网络请求。

cache 内存缓存
stats 统计信息
targetToAction 保存 Action 的 Map，key 为 target。
targetToDeferredRequestCreator 保存延迟请求的 Map，key 为 view。

Picasso 可由 Picasso.Builder 构造，包含

    private Downloader downloader 下载器，默认 api 9 以上用 okhttp 下载，9 以下用 urlconnection 下载
    private ExecutorService service 用于获取图片的线程池
    private Cache cache 内存缓存
    private Listener listener 回调接口
    private RequestTransformer transformer 图片请求拦截器，用于处理图片请求，图片转换接口
    private List<RequestHandler> requestHandlers 不同类型图片请求处理器
    private Bitmap.Config defaultBitmapConfig 图片配置
    private boolean indicatorsEnabled 是否显示图片来源指示器
    private boolean loggingEnabled 是否允许日志

RequestHandler 目前可处理包括本地资源、联系人、其他 content、asset 资源、本地文件、网络图片。

Cache.java
内存缓存接口

LruCache.java
最近使用的内存缓存

Downloader.java
网络图片加载器的接口，仅包含`load`和`shutdown`两个接口。

Dispatcher.java
负责分发和处理 Action，包括提交、暂停、继续、取消、网络状态变化、重试等等。

RequestCreator.java
Requet 构建器

BitmapHunter.java
获取图片的任务，实现了 Runnable，包括当前缓存实例、分发器、能处理它的 RequestHandler。

Downloader.Response
Downloader 的`load`接口返回类型，可包含表示图片的`Bitmap`或是`InputStream`

RequestHandler.java
除了某些类型的图片请求，比如 file、resource。

Downloader.Result
RequestHandler 的`load`接口返回类型，可包含表示图片的`Bitmap`或是`InputStream`

RequestWeakReference.java
包含`Action`的 WeakReference

Request.java
图片请求信息，包括唯一 id、uri(resourceid)、起始时间、网络策略、优先级、裁剪信息、目标尺寸、缓存的 key。

Action.java
包含一次图片请求需要的信息，如当前缓存实例、请求、目标 View、key、网络策略、内存策略、失败图片、是否已取消等。

NetworkPolicy.jva
网络请求策略，分为不本地存储(每次走网络)、不自己本地存储(使用 okhttp)、离线模式三种。

MemoryPolicy.java
内存缓存策略，包括不走缓存、最新不写缓存。

Stats.java
图片缓存使用统计信息类，

StatsSnapshot.java
图片缓存使用情况某一时刻具体信息，可由 Stats 生成。