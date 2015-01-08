##Android-Universal-Image-Loader源码分析
> 本文为 Android [开源项目实现原理解析](https://github.com/android-cn/android-open-project-analysis)中
universal-image-loader-demo-huxian99 部分  
项目地址：[Android-Universal-Image-Loader](https://github.com/nostra13/Android-Universal-Image-Loader),分析的版本：[eb794c3](https://github.com/nostra13/Android-Universal-Image-Loader/commit/eb794c306c1707a6cce80764b01f52109d5b9056),Demo 地址：[Universal-Image-Loader Demo](https://github.com/android-cn/android-open-project-demo/tree/master/universal-image-loader-demo-huxian99)  
分析者：[huxian99](https://github.com/huxian99)，校对者：[grumoon](https://github.com/grumoon)，校对状态：未完成

###1. 功能介绍
#####简单的说就做了一件事, 将图片显示在相应的控件上.
看似简单的一句话, 但实际项目里会碰到各种各样问题，比如多图片的缓存策略，磁盘缓存，网络请求的线程池策略等等. 好在Android-Universal-Image-Loader这个库都帮我们做好了，只需要简单配置就可以使用了.
####1.1 自定义配置
可以根据自己的项目需求，自己配置使用Universal-Image-Loader
主要通过ImageLoderConfiguration配置下载器，解码器，内存缓存，磁盘缓存及DisplayImageOptions等
####1.2 简单上手
#####1.2.1 添加依赖
```Gradle
	compile 'com.nostra13.universalimageloader:universal-image-loader:1.9.3'
```
#####1.2.2添加权限
```xml
	<uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
```
#####1.2.3配置参数  
配置ImageLoaderConfiguration并初始化ImageLoader， 建议在Application里初始化, 但是一定要在使用之前配置并初始化.
```java
public class YourApplication extends Application {

	@Override
	public void onCreate() {
	    super.onCreate();
	    ImageLoaderConfiguration configuration = new ImageLoaderConfiguration.Builder(this)
	        .//添加你的配置需求
	        .build();
	    ImageLoader.getInstance().init(configuration);
    }
}
```
当然不要忘记在AndroidManifest的application节点下添加android:name="YourApplication"
接下来就是使用了
ImageLoader.getInstance().displayImage();
ImageLoader.getInstance().loadImage();

###2. 总体设计
整个库分为 **Downloader**, **Decoder**, **Cache**, **Processor**, **Displayer**这几个模块  
其中 **Cache** 又分为 **DiskCache** 和 **MemoryCache** 两部分

**Downloader**模块主要负责从图片的各个来源获取输入流, 具体来源参考4.2.10  
**Cache**中的DiskCache主要负责将图片缓存在本地磁盘上, 具体参考4.2.12，  MemoryCache主要负责将图片缓存在内存里， 具体参考4.2.17  
**Decoder**模块主要负责将图片decode成bitmap对象, 具体参考4.2.15  
**Processor**模块提供了接口供开发者自己去实现，具体参考4.2.21  
**Displayer**模块将bitmap对象显示在相应的控件上, 具体参考4.2.19  

###3. 流程图
![](https://github.com/android-cn/android-open-project-analysis/blob/master/universal-image-loader/image/universal-image-loader-flow.png)  

###4. 详细设计
####4.1 类关系图
![](https://github.com/android-cn/android-open-project-analysis/blob/master/universal-image-loader/image/relation-class.png)  
####4.2 核心类功能介绍
#####4.2.1 ImageLoaderConfiguration.java
封装了ImageLoader的配置信息，可以根据自己项目的需要定义，需要在Application里面做初始化操作
主要成员变量及含义：
```java
/** 资源 */
final Resources resources;
/** 内存缓存最大图片宽 */
final int maxImageWidthForMemoryCache;
/** 内存缓存最大图片高 */
final int maxImageHeightForMemoryCache;
/** 磁盘缓存最大图片宽 */
final int maxImageWidthForDiskCache;
/** 磁盘缓存最大图片高 */
final int maxImageHeightForDiskCache;
/** 磁盘缓存的处理器 */
final BitmapProcessor processorForDiskCache;
/** 若未指定，默认ThreadPoolExecutor的实例 */
final Executor taskExecutor;
/** 若未指定，默认ThreadPoolExecutor的实例 */
final Executor taskExecutorForCachedImages;
/** 用户自定义执行者 */
final boolean customExecutor;
/** 用户自定义缓存图片的执行者 */
final boolean customExecutorForCachedImages;
/** 线程池大小 */
final int threadPoolSize;
/** 线程优先级 */
final int threadPriority;
/** 队列进程enum类型 有FIFO, LIFO可供选择 */
final QueueProcessingType tasksProcessingType;
/** 内存缓存 */
final MemoryCache memoryCache;
/** 磁盘缓存 一般放在SD卡 */
final DiskCache diskCache;
/** 下载器 */
final ImageDownloader downloader;
/** 图片解码器 类似于我们常用的BitmapFactory.decode... 将图片资源解码成bitmap对象 */
final ImageDecoder decoder;
/** DisplayImageOptions对象 */
final DisplayImageOptions defaultDisplayImageOptions;
/** 处理网络拒绝访问的下载器 */
final ImageDownloader networkDeniedDownloader;
/** 处理慢网络情况的下载器 */
final ImageDownloader slowNetworkDownloader;
```
#####4.2.2 ImageLoaderConfiguration.Builder.java静态内部类
用于构造ImageLoaderConfiguration，其属性也是ImageLoaderConfiguration的一些设置参数.
```java
//设置内存缓存的选项
memoryCacheExtraOptions(int maxImageWidthForMemoryCache, int maxImageHeightForMemoryCache)

//设置磁盘缓存的选项
diskCacheExtraOptions(int maxImageWidthForDiskCache, int maxImageHeightForDiskCache, BitmapProcessor processorForDiskCache)

//设置自定义加载和显示任务的线程池
taskExecutor(Executor executor)

//设置自定义显示任务的线程池
taskExecutorForCachedImages(Executor executorForCachedImages)

//设置图片显示任务的线程池大小
threadPoolSize(int threadPoolSize)

//设置线程的优先级
threadPriority(int threadPriority)

//设置拒绝缓存一张图片的多个尺寸
denyCacheImageMultipleSizesInMemory()

//设置加载和显示图片任务队列的类型
tasksProcessingOrder(QueueProcessingType tasksProcessingType)

//设置最大内存缓存字节数
memoryCacheSize(int memoryCacheSize)

//设置最大内存缓存所占app可用内存的百分比
memoryCacheSizePercentage(int availableMemoryPercent)

//设置内存缓存算法
memoryCache(MemoryCache memoryCache)

//设置最大磁盘缓存字节数
diskCacheSize(int maxCacheSize)

//设置在磁盘缓存文件夹下最多文件数
diskCacheFileCount(int maxFileCount)

//设置磁盘缓存文件生成的命名规则
diskCacheFileNameGenerator(FileNameGenerator fileNameGenerator)

//设置磁盘缓存
diskCache(DiskCache diskCache)

//设置下载图片的工具
imageDownloader(ImageDownloader imageDownloader)

//设置decode出bitmap的工具
imageDecoder(ImageDecoder imageDecoder)

//设置默认的DisplayImageOptions
defaultDisplayImageOptions(DisplayImageOptions defaultDisplayImageOptions)

//log信息
writeDebugLogs()

public ImageLoaderConfiguration build() {
	initEmptyFieldsWithDefaultValues();
	return new ImageLoaderConfiguration(this);
}
```
build()方法用于创建ImageLoaderConfiguration，其中initEmptyFieldsWithDefaultValues()方法用于初始化配置，若用户没有配置相关项，UIL会通过DefaultConfigurationFactory定义一个默认当配置.  
e.g.用户未指定磁盘缓存的命名规则，则在DefaultConfigurationFactory.createFileNameGenerator()方法中可以看到它默认帮我们选择了HashCode的命名规则
#####4.2.3 DisplayImageOptions.java
图片显示的一些选项，比如是否在磁盘缓存，memory缓存，加载失败时显示的图片等  
主要成员变量含义：
```java
/** 加载时显示resource的图片 */
private final int imageResOnLoading;
/** 空uri时显示resource的的图片 */
private final int imageResForEmptyUri;
/** 加载失败时显示resource的的图片 */
private final int imageResOnFail;
/** 加载时显示drawabled对象 */
private final Drawable imageOnLoading;
/** 空uri时显示drawabled对象 */
private final Drawable imageForEmptyUri;
/** 加载失败时显示drawabled对象 */
private final Drawable imageOnFail;
/** 在加载前是否重置view */
private final boolean resetViewBeforeLoading;
/** 是否缓存在内存中 */
private final boolean cacheInMemory;
/** 是否缓存在磁盘中 */
private final boolean cacheOnDisk;
/** 图片的缩放类型 默认是IN_SAMPLE_POWER_OF_2 */
private final ImageScaleType imageScaleType;
/** BitmapFactory.Options */
private final Options decodingOptions;
/** 设置在开始加载前的延迟时间 */
private final int delayBeforeLoading;
/** 是否考虑图片的EXIF信息 */
private final boolean considerExifParams;
/** 下载时可以额外传入的对象，方便用户自己扩展 */
private final Object extraForDownloader;
/** 缓存在内存之前的处理程序 */
private final BitmapProcessor preProcessor;
/** 缓存在内存之后的处理程序 */
private final BitmapProcessor postProcessor;
/** 图片的显示方式 */
private final BitmapDisplayer displayer;
/** handler对象 */
private final Handler handler;
/** 是否同步加载 */
private final boolean isSyncLoading;
```
#####4.2.4 DisplayImageOptions.Builder.java静态内部类  
用于构造DisplayImageOptions，其属性也是DisplayImageOptions的一些设置参数.  
```java
//设置图片加载过程中显示的图片
showImageOnLoading(int imageRes)

//设置空URI显示的图片
showImageForEmptyUri(int imageRes)

//设置发生错误显示的图片
showImageOnFail(int imageRes)

//设置图片加载前是否重置
resetViewBeforeLoading(boolean resetViewBeforeLoading)

//设置是否将图片缓存在内存里
cacheInMemory(boolean cacheInMemory)

//设置是否将图片缓存在磁盘上
cacheOnDisk(boolean cacheOnDisk)

//设置图片缩放的类型
imageScaleType(ImageScaleType imageScaleType)

//设置Bitmap.Config
bitmapConfig(Bitmap.Config bitmapConfig)

//设置图片解码的选项
decodingOptions(Options decodingOptions)

//设置加载任务前的延迟
delayBeforeLoading(int delayInMillis)

//设置下载时额外的对象
extraForDownloader(Object extra)

//设置是否考虑EXIF信息
considerExifParams(boolean considerExifParams)

//设置内存缓存bitmap对象前的处理
preProcessor(BitmapProcessor preProcessor)

//设置内存缓存bitmap对象后的处理
postProcessor(BitmapProcessor postProcessor)

//设置图片的显示方式
displayer(BitmapDisplayer displayer)

//true是直接调用LoadAndDisplayImageTask对象的run方法，false是放到线程池里面执行，默认false
syncLoading(boolean isSyncLoading)

//设置显示图片和触发ImageLoadingListener事件的自定义Handler对象
handler(Handler handler)
```
#####4.2.5 ImageLoader.java
采取了单例模式，用于图片的加载和显示，在使用之前必须先进行初始化.  
我们在使用时调用的displayImage和loadImage方法最终都会走到下面这个方法中
```java
public void displayImage(String uri, ImageAware imageAware, DisplayImageOptions options, ImageLoadingListener listener, ImageLoadingProgressListener progressListener)
```
因此只分析这一个并简化如下
```java
...
//从内存缓存中获取了bitmap对象
Bitmap bmp = configuration.memoryCache.get(memoryCacheKey);
if (bmp != null && !bmp.isRecycled()) {
	// 缓存中已获取bitmap，执行ProcessAndDisplayImageTask
} else {
	// 缓存中未获取bitmap, 执行LoadAndDisplayImageTask
}
```
#####4.2.6 ProcessAndDisplayImageTask.java
```java
BitmapProcessor processor = imageLoadingInfo.options.getPostProcessor();
Bitmap processedBitmap = processor.process(bitmap);
DisplayBitmapTask displayBitmapTask = new DisplayBitmapTask(processedBitmap, imageLoadingInfo, engine,
				LoadedFrom.MEMORY_CACHE);	LoadAndDisplayImageTask.runTask(displayBitmapTask, imageLoadingInfo.options.isSyncLoading(), handler, engine);
```
可以看到主要是做了一些postProcess，然后执行DisplayBitmapTask将bitmap对象显示到相应到控件上.
#####4.2.7 LoadAndDisplayImageTask.java(重点)
这是一个实现了Runnable接口的线程类. Android-Universal-Image-Loader**主要的操作**都是在这个类完成的. 从网络获取输入流，做磁盘的缓存，decode出bitmap对象，内存缓存等，其中display是在run方法中执行了DisplayBitmapTask这个线程，直接看run方法
```java
bmp = configuration.memoryCache.get(memoryCacheKey);
if (bmp == null || bmp.isRecycled()) {
	bmp = tryLoadBitmap();
	...
	...
	...
	if (bmp != null && options.isCacheInMemory()) {
		L.d(LOG_CACHE_IMAGE_IN_MEMORY, memoryCacheKey);
		configuration.memoryCache.put(memoryCacheKey, bmp);
	}
}
```
从上面代码段中可以看到先是从内存缓存中去读取bitmap对象，若bitmap对象不存在，则调用tryLoadBitmap()方法获取bitmap对象，若在DisplayImageOptions.Builder中设置了cacheInMemory(true), 就将bitmap对象缓存到内存中.  
接下来看tryLoadBitmap()
```java
File imageFile = configuration.diskCache.get(uri);
if (imageFile != null && imageFile.exists()) {
	...
	bitmap = decodeImage(Scheme.FILE.wrap(imageFile.getAbsolutePath()));
}
if (bitmap == null || bitmap.getWidth() <= 0 || bitmap.getHeight() <= 0) {
	...
	String imageUriForDecoding = uri;
	if (options.isCacheOnDisk() && tryCacheImageOnDisk()) {
		imageFile = configuration.diskCache.get(uri);
		if (imageFile != null) {
			imageUriForDecoding = Scheme.FILE.wrap(imageFile.getAbsolutePath());
		}
	}
	checkTaskNotActual();
	bitmap = decodeImage(imageUriForDecoding);
	...
}
```
很简单的流程，根据uri看看磁盘中是不是已经缓存了这个文件，如果有了，调用decodeImage方法，将图片文件decode成bitmap对象，若文件不存在，调用tryCacheImageOnDisk()方法去下载并缓存图片到本地磁盘，再通过decodeImage方法将图片文件decode成bitmap对象
下面看tryCacheImageOnDisk()方法里具体都有什么东西
```java
	loaded = downloadImage();
```
主要就是这一句话,如果你在ImageLoaderConfiguration中还配置了maxImageWidthForDiskCache或者maxImageHeightForDiskCache，还会调用一个叫resizeAndSaveImage()方法，主要是调整一下尺寸，并保存这个图片文件, 下面主要看一下downloadImage()方法
```java
private boolean downloadImage() throws IOException {
	InputStream is = getDownloader().getStream(uri, options.getExtraForDownloader());
	if (is == null) {
		L.e(ERROR_NO_IMAGE_STREAM, memoryCacheKey);
		return false;
	} else {
		try {
			return configuration.diskCache.save(uri, is, this);
		} finally {
			IoUtils.closeSilently(is);
		}
	}
}
```
主要将下载任务交给downloader去获取输入流，然后将其保存在diskCache中
其中downloader如果没有配置的话，默认是BaseImageDownloader.
下面会分析LoadAndDisplayImageTask.java这个类,这个类信息量比较大.
现在做一下总结:  
1. 判断图片的内存缓存是否存在，若存在直接执行步骤8  
2. 判断图片的磁盘缓存是否存在，若存在直接执行步骤5  
3. 从网络上下载图片  
4. 将图片缓存在磁盘上  
5. 将图片decode成bitmap对象  
6. 根据DisplayImageOptions配置对图片进行预处理(Pre-process Bitmap)  
7. 将bitmap对象缓存到内存中  
8. 根据DisplayImageOptions配置对图片进行后处理(Post-process Bitmap)  
9. 执行DisplayBitmapTask将图片显示在相应的控件上  
#####4.2.8 ImageLoaderEngine.java
执行线程LoadAndDisplayImageTask的引擎，三个线程池涉及的线程调优策略.  
```java
private Executor taskExecutor;
private Executor taskExecutorForCachedImages;
private Executor taskDistributor;
```
默认情况下:  
taskDistributor使用Executors.newCachedThreadPool()工厂方法创建.  
taskExecutor和taskExecutorForCachedImages线程池配置一致.  
具体内容如下:   

parameters|taskDistributor|taskExecutorForCachedImages/taskExecutor
---|---|---
corePoolSize|0|3
maximumPoolSize|Integer.MAX_VALUE|3
keepAliveTime|60|0
unit|SECONDS|MILLISECONDS
workQueue|SynchronousQueue|LIFOLinkedBlockingDeque/LinkedBlockingQueue
priority|5|3
  
taskDistributor用于从磁盘获取图片操作，根据在磁盘缓存中是否找到，使用不同的Executor执行任务. 在缓存中则使用taskExecutorForCachedImages去执行，若不在则使用taskExecutor去执行.  
[合理的配置线程池请参考这里](http://www.infoq.com/cn/articles/java-threadPool) 

#####4.2.9 DisplayBitmapTask.java
实现Runnable接口的线程类  
```java
displayer.display(bitmap, imageAware, loadedFrom);
engine.cancelDisplayTaskFor(imageAware);
listener.onLoadingComplete(imageUri, imageAware.getWrappedView(), bitmap);
``` 
第一句主要用于将bitmap对象显示在imageAware上.
第二句调用了ImageLoaderEngine中的cancelDisplayTaskFor方法，移除正在加载的任务.
第三句执行onLoadingComplete()方法，将bitmap对象回调给用户.

下面主要分析下一张图片从download到display的每个阶段具体实现类, 每个类的分析选择DefaultConfigurationFactory中默认的配置
#####4.2.10 ImageDownloader.java
定义了图片下载的接口，通过定义的枚举Scheme, 可以看出Android-Universal-Image-Loader支持图片的来源
```java
HTTP("http"), HTTPS("https"), FILE("file"), CONTENT("content"), ASSETS("assets"), DRAWABLE("drawable"), UNKNOWN("");
```
#####4.2.11 BaseImageDownloader.java
ImageDownloader的具体实现类，主要通过调用getStream()方法根据不同来源获取输入流
```java
@Override
public InputStream getStream(String imageUri, Object extra) throws IOException {
	switch (Scheme.ofUri(imageUri)) {
		case HTTP:
		case HTTPS:
			return getStreamFromNetwork(imageUri, extra);
		case FILE:
			return getStreamFromFile(imageUri, extra);
		case CONTENT:
			return getStreamFromContent(imageUri, extra);
		case ASSETS:
			return getStreamFromAssets(imageUri, extra);
		case DRAWABLE:
			return getStreamFromDrawable(imageUri, extra);
		case UNKNOWN:
		default:
			return getStreamFromOtherSource(imageUri, extra);
	}
}
```  
#####4.2.12 DiskCache.java
定义了磁盘缓存的接口, 主要方法
File get(String imageUri)根据原始图片的uri去获取缓存图片的文件  
boolean save() 保存图片到磁盘中
boolean remove(String imageUri) 根据图片uri移出图片  
#####4.2.13 UnlimitedDiskCache.java
三个构造器主要继承了BaseDiskCache.java  
#####4.2.14 BaseDiskCache.java
抽象类实现了DiskCache接口  
主要看两个save()方法
```java
/** 通过输入流imageStream写入到相应的文件中 */
boolean save(String imageUri, InputStream imageStream, IoUtils.CopyListener listener)
/** 通过调用bitmap.compress()方法将bitmap对象写入到相应的文件中 */
boolean save(String imageUri, Bitmap bitmap)
```
在网络图片download下来并缓存到磁盘中用的是第一种save下面主要分析下：  
```java
public boolean save(String imageUri, InputStream imageStream, IoUtils.CopyListener listener) throws IOException {
	File imageFile = getFile(imageUri);
	File tmpFile = new File(imageFile.getAbsolutePath() + TEMP_IMAGE_POSTFIX);
	boolean loaded = false;
	try {
		OutputStream os = new BufferedOutputStream(new FileOutputStream(tmpFile), bufferSize);
		try {
			loaded = IoUtils.copyStream(imageStream, os, listener, bufferSize);
		} finally {
			IoUtils.closeSilently(os);
		}
	} finally {
		if (loaded && !tmpFile.renameTo(imageFile)) {
			loaded = false;
		}
		if (!loaded) {
			tmpFile.delete();
		}
	}
	return loaded;
}
```
它先是生成一个后缀名.tmp的临时文件，通过downloader得到的输入流imageStream拷贝到OutputStream中, finally中将临时文件tmpFile重命名回imageFile，并将tmpFile删除掉, 如果这些实现都没出什么问题，就reutrn一个true, 告诉别人，我save成功了

#####4.2.15 ImageDecoder.java
定义了decode的接口，定义了Bitmap decode()方法
#####4.2.16 BaseImageDecoder.java
decode过程的具体实现类, 主要看decode方法，传入一个主要的参数就是ImageDecodingInfo, 根据decode的信息去解析图片.
```java
Options decodingOptions = prepareDecodingOptions(imageInfo.imageSize, decodingInfo);
decodedBitmap = BitmapFactory.decodeStream(imageStream, null, decodingOptions);
```
源码中主要是通过decodingOptions去decode一个bitmap对象，那么主要看一下相对应的prepareDecodingOptions()
```java
protected Options prepareDecodingOptions(ImageSize imageSize, ImageDecodingInfo decodingInfo) {
	ImageScaleType scaleType = decodingInfo.getImageScaleType();
	int scale;
	if (scaleType == ImageScaleType.NONE) {
		scale = 1;
	} else if (scaleType == ImageScaleType.NONE_SAFE) {
		scale = ImageSizeUtils.computeMinImageSampleSize(imageSize);
	} else {
		ImageSize targetSize = decodingInfo.getTargetSize();
		boolean powerOf2 = scaleType == ImageScaleType.IN_SAMPLE_POWER_OF_2;
		scale = ImageSizeUtils.computeImageSampleSize(imageSize, targetSize, decodingInfo.getViewScaleType(), powerOf2);
	}
	if (scale > 1 && loggingEnabled) {
		L.d(LOG_SUBSAMPLE_IMAGE, imageSize, imageSize.scaleDown(scale), scale, decodingInfo.getImageKey());
	}

	Options decodingOptions = decodingInfo.getDecodingOptions();
	decodingOptions.inSampleSize = scale;
	return decodingOptions;
}
```
其中主要看这里
```java
scale = ImageSizeUtils.computeImageSampleSize(imageSize, targetSize, decodingInfo.getViewScaleType(), powerOf2);
...
decodingOptions.inSampleSize = scale;
```
是根据这个scale设置options的inSampleSize，也就是缩放的比例
通过查看ImageSizeUtils.computeImageSampleSize可以得到下面
```java
switch (viewScaleType) {
	case FIT_INSIDE:
		...
		break;
	case CROP:
		...
		break;
}
```
可以看出它是根据viewScaleType做了两种处理, 那么什么是FIT_INSIDE，什么又是CROP呢，翻看ViewScaleType源代码看到
```java
public static ViewScaleType fromImageView(ImageView imageView) {
	switch (imageView.getScaleType()) {
		case FIT_CENTER:
		case FIT_XY:
		case FIT_START:
		case FIT_END:
		case CENTER_INSIDE:
			return FIT_INSIDE;
		case MATRIX:
		case CENTER:
		case CENTER_CROP:
		default:
			return CROP;
	}
}
```
很好理解了，就是我们设置ImageView的ScaleType...  
那么FIT_INSIDE和CROP里面是什么呢，就是比对srcSize和targetSize大小，当srcSize比targetSize大，scale就扩大，具体可以自己看一下，然后将scale返回赋值给inSampleSize就可以了, 如果还不了解为什么这么做的朋友可以去看一下[官方training-有效加载大图](http://developer.android.com/training/displaying-bitmaps/load-bitmap.html)
这样，图片的压缩处理就做好了，然后将bitmap返回.  

#####4.2.17 MemoryCache.java
定义了内存缓存的接口类，默认算法的选择是LruMemoryCache，并且将android系统分配给每个应用内存的1/8作为阈值  
#####4.2.18 LruMemoryCache.java
LRU: Least Recently Used近期最少使用算法.
一般实现LRU算法都是使用LinkedHashMap，UIL也不例外，看到它定义了
```java
private final LinkedHashMap<String, Bitmap> map;
private final int maxSize;
private int size;  //缓存中bytes的大小
```
但是它没有重写removeEldestEntry，而是自己写了个trimToSize()方法控制内存缓存的大小
get(), remove()方法很好理解，因此只看put(),源代码如下:
```java
@Override
public final boolean put(String key, Bitmap value) {
	if (key == null || value == null) {
		throw new NullPointerException("key == null || value == null");
	}

	synchronized (this) {
		size += sizeOf(key, value);
		Bitmap previous = map.put(key, value);
		if (previous != null) {
			size -= sizeOf(key, previous);
		}
	}

	trimToSize(maxSize);
	return true;
}
```
其中trimToSize主要目的是移出链表表头的Entry对象，因为LinkedHashMap是采用了链表结构，每当有一个Entry对象被加入进来或者被使用，这个Entry对象就会被放置在链尾，当size超过当前的maxSize时，就会将链表表头的Entry对象从缓存中remove掉, 重新计算缓存的size  
#####4.2.19 Displayer.java
在控件(比如ImageView)中显示bitmap对象, 适用于一些bitmap对象的改变或者任何动画效果
假如你使用了FadeInBitmapDisplayer，在显示过程中就会有一个淡入的动画效果，也可以自己实现一些动画，只要实现Displayer接口就可以了. 默认实现的是SimpleBitmapDisplayer  
#####4.2.20 SimpleBitmapDisplayer.java
进去一下，发现它只定义了一个方法
```java
@Override
public void display(Bitmap bitmap, ImageAware imageAware, LoadedFrom loadedFrom) {
	imageAware.setImageBitmap(bitmap);
}
```
做的仅仅是把bitmap对象display到imageAware控件上
#####4.2.21 BitmapProcessor.java  
到这里整个流程都走完了，那么细心的你一定会发现流程图中不是还有预处理(Pre-process Bitmap)和后处理(Post-process Bitmap)吗? 没错，那么现在分析它，看源代码
```java
public interface BitmapProcessor {
	Bitmap process(Bitmap bitmap);
}
```
只是定义了一个方法，让开发者自己去实现它.  
比如你想要为你的图片添加一个水印，那么可以在Pre-process这个阶段自己去实现BitmapProcessor接口，这样还可以把加了水印的图片缓存到内存中
但是不要忘记要在DisplayImageOptions中配置一下   
#####4.2.22 PauseOnScrollListener.java  
最后分析一下这个类, 它实现了OnScrollListener接口， 如果在ListView或GridView中有加载图片的工作最好还是使用它. 简单的一行代码:  
```java
gridView.setOnScrollListener(new PauseOnScrollListener(ImageLoader.getInstance(), false, true));
```
看下主要的成员变量及方法
```java
/** 滑动过程是否暂停 */
private final boolean pauseOnScroll;
/** Fling过程是否暂停 */
private final boolean pauseOnFling;
```
个人选择是pauseOnScroll＝false，pauseOnFling＝true.
在用户滑动屏幕的时候还是不要暂停, 因为用户滑动屏幕说明他还是关注图片内容的, 如果在fling状态下, 用户可能迫不及待想看后面的内容，最好就不要处理图片来"打扰"用户了, 当然你可以根据自己的喜好设置他们.  

###5. 杂谈
####聊聊LRU
Universal-Image-Loader在内存缓存时默认使用了LRU算法.  
LRU: Least Recently Used近期最少使用算法, 选用了基于链表结构的LinkedHashMap.  
假设情景：内存缓存设置的阈值只够存储两个bitmap对象，当put第三个bitmap对象时，将近期最少使用的bitmap对象移除.  
图1: 初始化LinkedHashMap, 并按使用顺序来排序, accessOrder = true;  
图2: 向缓存池中放入bitmap1和bitmap2两个对象.  
图3: 继续放入第三个bitmap3，根据假设情景，将会超过设定缓存池阈值.  
图4: 释放对bitmap1对象的引用.  
图5: bitmap1对象被GC回收.  
![](https://github.com/android-cn/android-open-project-analysis/blob/master/universal-image-loader/image/lru_header.png)  
  
![](https://github.com/android-cn/android-open-project-analysis/blob/master/universal-image-loader/image/lru_put.png)  
  
![](https://github.com/android-cn/android-open-project-analysis/blob/master/universal-image-loader/image/lru_put_exceed_maxsize2.png) 
  
![](https://github.com/android-cn/android-open-project-analysis/blob/master/universal-image-loader/image/lru_put_trim.png)  
  
![](https://github.com/android-cn/android-open-project-analysis/blob/master/universal-image-loader/image/lru_trim_result.png)  
