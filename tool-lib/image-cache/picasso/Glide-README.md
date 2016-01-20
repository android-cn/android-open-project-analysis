Glide.java

RequestManagerRetriever.java
RequestManager 管理器，根据 Fragment 或者 Activity 得到 RequestManager

RequestManagerFragment
继承自 Fragment

RequestManager
请求管理器

GlideBuilder.java
GlideBuilder 创建者，用于创建 Glide 实例。
    sourceExecutor 获取数据线程池，最大并发为 cpu 核数
    diskCacheExecutor disk 缓存获取线程池，最大并发为 1
    bitmapPool 内存缓存，11 以上为 LRU cache
    memoryCache resource 的内存缓存
    engine 引擎

BitmapPreFiller.java

ResourceEncoder.java
决定缓存原数据、转换后的数据还是不缓存数据到本地。

RequestTracker.java
Request 执行

DecodeJob.java
获取数据

EngineJob.java
管理 DecodeJob

DataFetcher.java
获取数据

MemorySizeCalculator.java
内存缓存大小计算器，和屏幕分辨率、长宽及自配置有关。

GlideExecutor.java
Glide 线程池，自定义线程名，可配置核心池大小

ResourceCallback.java
资源加载完成或失败回调接口

Registry.java
注册 module、decode、encode。

GlideModule.java
全局 Glide 配置，可用于修改默认的 Builder 或者注册新的处理配置。

ManifestParser.java
解析 Manifest 得到所有的 GlideModule

AttributeStrategy
LRU BitMap Cache，以 Bitmap 的 width、height、config 为联合 key。

GroupedLinkedMap.java
LRU LinkedMap