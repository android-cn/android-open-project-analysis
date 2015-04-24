Fresco 源码解析
====================================
> 本文为 [Android 开源项目源码解析](https://github.com/android-cn/android-open-project-analysis) 中 Fresco 部分  
 项目地址：[Fresco](https://github.com/facebook/fresco)，分析的版本：[e46dab3](https://github.com/facebook/fresco/commit/e46dab3c2beac3f16163e8593f8a74840606aaef)，Demo 地址：[Fresco Demo](https://github.com/aosp-exchange-group/android-open-project-demo/tree/master/fresco-demo)
 分析者：[blackiedm](https://github.com/blackiedm)，分析状态：未完成，校对者：[Trinea](https://github.com/trinea)，校对状态：未开始


###1. 功能介绍
####1.1 Fresco
Fresco 是一个强大的图片加载和显示组件。支持从网络、本地文件系统、本地资源下载图片。

它有三级缓存(两级内存、一级磁盘缓存)。并且支持Android2.3(API level 9) 及其以上系统。

####1.2 特性
- 出色的内存管理。当图片不显示即离屏时，占用的内存将会被释放（在Android5.0以下的系统）。
- 支持渐进式图片格式(Progressive JPEG)。
- 支持Gif和WebP图片格式。（而且还支持其动画）
- 多样式的呈现方式。
    - 取代居中，自定义聚焦点。
    - 支持圆角或圆形图片展示。
    - 获取图片失败时，可以点击占位图重新加载。
    - 可以自定义背景，覆盖层，进度条。
    - 支持自定义聚焦即手按下时的覆盖层。
- 多样式加载
    - 为图片指定不同远程路径，并且可以使用已经缓存在本地的图片。
    - 支持先显示低清图片，当加载完成后在过渡为高清图片
    - 图片加载完成后回调通知
    - 如有EXIF缩略图，在大图加载完成之前，可先显示缩略图（只支持本地图片）
    - 支持缩放或旋转功能
    - 支持处理加载完成的图片
    - 支持WebP格式图片

####1.3 基本使用
#####1.3.1 Manifest 配置 和 Gradle依赖
在Manifest里面添加权限：

```xml
<uses-permission android:name="android.permission.INTERNET"/>
```

添加依赖：(x.x.x为版本号)

```java
dependencies{
// your app's other dependencies
compile 'com.facebook.fresco:fresco:x.x.x'
}
```
#####1.3.2 初始化
在Application初始化时，调用：

```java
Fresco.initialize(context);
```
#####1.3.3 基本使用
一般情况下，使用SimpleDraweeView基本可完成你所需要的功能。

1. XML自定义使用：

    在XML布局中，加入命名空间：

        ```xml
        <RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
            xmlns:fresco="http://schemas.android.com/apk/res-auto"
        ```

    使用SimpleDraweeView:

        ```xml
        <com.facebook.drawee.view.SimpleDraweeView
            android:id="@+id/img"
            android:layout_centerInParent="true"
            android:layout_width="200dp"
            android:layout_height="200dp"
            android:layout_margin="10dp"
            fresco:fadeDuration="1000"
            fresco:placeholderImage="@color/placeholder"
            fresco:failureImage="@color/error"
            fresco:retryImage="@color/retrying"
            fresco:backgroundImage="@color/blue"
            />
        ```

    设置图片路径：(支持的URIs: http://, https://, file://, content://, asset://, res://)

        ```java
        //注意:这里是指绝对路径
        simpleDraweeView.setImageURI(uri);
        ```

2. 代码自定义使用：

    ```java
    GenericDraweeHierarchyBuilder builder = new GenericDraweeHierarchyBuilder(getResources());
    GenericDraweeHierarchy hierarchy = builder.setFadeDuration(1000)
            //设置实际图片的scaleType
            .setActualImageScaleType(ScalingUtils.ScaleType.FOCUS_CROP)
            .setActualImageFocusPoint(new PointF(1f, 1f))
            //设置占位图drawable和scaleType
            .setPlaceholderImage(getResources().getDrawable(R.color.placeholder), ScalingUtils.ScaleType.CENTER_CROP)
            //设置error drawable和scaleType
            .setFailureImage(getResources().getDrawable(R.color.error), ScalingUtils.ScaleType.CENTER_CROP)
            //设置重试drawable， 记得在controller下设置setTapToRetryEnabled(true)
            .setRetryImage(getResources().getDrawable(R.color.retrying))
            .build();
    simpleDraweeView.setHierarchy(hierarchy);
    DraweeController controller = Fresco.newDraweeControllerBuilder()
            //tap-to-retry load image
            .setTapToRetryEnabled(true)
            //在构建新的控制器时需要setOldController，这可以防止重新分配内存
            .setOldController(simpleDraweeView.getController())
            //注意:uri是指绝对路径
            .setUri(uri)
            .build();
    simpleDraweeView.setController(controller);
    ```

 具体使用可参考[Fresco demo](https://github.com/aosp-exchange-group/android-open-project-demo/tree/master/fresco-demo)


###2. 详细设计
###2.1 类详细介绍
类及其主要函数功能介绍、核心功能流程图，流程图可使用 [Google Drawing](https://docs.google.com/drawings)、[Visio](http://products.office.com/en-us/visio/flowchart-software)、[StarUML](http://staruml.io/)。  
###2.2 类关系图
类关系图，类的继承、组合关系图，可是用 [StarUML](http://staruml.io/) 工具。  

**完成时间**  
- 根据项目大小而定，目前简单根据项目 Java 文件数判断，完成时间大致为：`文件数 * 7 / 10`天，特殊项目具体对待  

###3. 流程图
主要功能流程图  
- 如 Retrofit、Volley 的请求处理流程，Android-Universal-Image-Loader 的图片处理流程图  
- 可使用 [Google Drawing](https://docs.google.com/drawings)、[Visio](http://products.office.com/en-us/visio/flowchart-software)、[StarUML](http://staruml.io/) 等工具完成，其他工具推荐？？  
- 非所有项目必须，不需要的请先在群里反馈  

**完成时间**  
- `两天内`完成  

###4. 总体设计
整个库分为哪些模块及模块之间的调用关系。  
- 如大多数图片缓存会分为 Loader 和 Processer 等模块。  
- 可使用 [Google Drawing](https://docs.google.com/drawings)、[Visio](http://products.office.com/en-us/visio/flowchart-software)、[StarUML](http://staruml.io/) 等工具完成，其他工具推荐？？  
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
并自行校验优化一遍，确认无误后将文章开头的  
`分析状态：未完成`  
变为：  
`分析状态：已完成`  

本期校对会由专门的`Buddy`完成，可能会对分析文档进行一些修改，请大家理解。  

**完成时间**  
- `两天内`完成  

**到此便大功告成，恭喜大家^_^**  
