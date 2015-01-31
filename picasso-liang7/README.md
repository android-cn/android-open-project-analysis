Picasso 源码解析
----------------
> 本文为 [Android开源项目源码解析](https://github.com/android-cn/android-open-project-analysis) 中 Picasso 部分    
> 项目地址：[Picasso](https://github.com/square/picasso)，分析的版本：，Demo 地址：[picasso-demo](https://github.com/android-cn/android-open-project-demo/tree/master/picasso-demo)  
> 分析者：[愛早起](https://github.com/liang7)，校对者：完成状态：未完成  

###1. 功能介绍
#####Android系统下载和缓存图片并加载的项目。
- 自动处理ImageView的回收和下载任务管理。
- 处理复杂的图像转换以减少内存使用。
- 提供内存和磁盘的高速缓存。

####1.1 一句话就能使用

```java
Picasso.with(context).load("http://i.imgur.com/DvpvklR.png").into(imageView);
```

####1.2 项目配置
#####1.2.1 添加依赖

```Gradle
compile 'com.squareup.picasso:picasso:2.4.0'
```

or Maven:

```Maven
<dependency>
  <groupId>com.squareup.picasso</groupId>
  <artifactId>picasso</artifactId>
  <version>2.4.0</version>
</dependency>
```

#####1.2.2添加权限

```xml
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
```

