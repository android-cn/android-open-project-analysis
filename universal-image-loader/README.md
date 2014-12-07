##Android-Universal-Image-Loader源码分析
> 本文为 Android [开源项目实现原理解析](https://github.com/android-cn/android-open-project-analysis)中
universal-image-loader-demo-huxian99 部分  
项目地址：[Android-Universal-Image-Loader](https://github.com/nostra13/Android-Universal-Image-Loader)  
分析的版本：1.9.3，Demo 地址：[Android-Universal-Image-Loader Demo](https://github.com/android-cn/android-open-project-demo/tree/master/universal-image-loader-demo-huxian99)  
分析者：[huxian99](https://github.com/huxian99)，校对者：校对状态：未完成

##1. 功能介绍
#####简单的说就做了一件事, 将**图片**显示在**相应的控件**上.
看似简单的一句话, 但实际项目里会碰到各种各样问题，比如多图片的缓存策略，磁盘缓存，网络请求的线程池策略等等. 好在Universal-Image-Loader这个库都帮我们做好了，只需要简单配置就可以使用了.
####1.1 高度自定义配置
可以根据自己的项目需求，自己配置使用Universal-Image-Loader
主要通过ImageLoderConfiguration配置下载器，解码器，内存缓存，磁盘缓存及DisplayImageOptions等
####1.2 简单上手
1.2.1**添加依赖**
```Gradle
	compile 'com.nostra13.universalimageloader:universal-image-loader:1.9.3'
```
1.2.2**添加权限**
```xml
	<uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
```
1.2.3**配置ImageLoaderConfiguration并初始化ImageLoader**， 建议在Application里初始化, 但是一定要在使用之前配置并初始化.
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

##2. 详细设计