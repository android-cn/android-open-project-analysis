Fresco 源码解析
====================================
> 本文为 [Android 开源项目源码解析](http://a.codekk.com) 中 Fresco 部分  
 项目地址：[Fresco](https://github.com/facebook/fresco)，分析的版本：[e46dab3](https://github.com/facebook/fresco/commit/e46dab3c2beac3f16163e8593f8a74840606aaef)，Demo 地址：[Fresco Demo](https://github.com/aosp-exchange-group/android-open-project-demo/tree/master/fresco-demo)
 分析者：[blackiedm](https://github.com/blackiedm)，分析状态：未完成，校对者：[Trinea](https://github.com/trinea)，校对状态：未开始


### 1. 功能介绍
#### 1.1 Fresco
Fresco 是一个强大的图片加载和显示组件。支持从网络、本地文件系统、本地资源下载图片。

它有三级缓存(两级内存、一级磁盘缓存)。并且支持Android2.3(API level 9) 及其以上系统。

#### 1.2 特性
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

#### 1.3 基本使用
##### 1.3.1 Manifest 配置 和 Gradle依赖
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
##### 1.3.2 初始化
在Application初始化时，调用：

```java
Fresco.initialize(context);
```
##### 1.3.3 基本使用
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

    设置图片路径：

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


### 2. 详细设计
#### 2.1 类关系图
#### 2.2 类详细介绍

##### 2.2.1 SimpleDraweeView
官方推荐使用的显示图片类，输入一个URI,即可显示图片。可通过xml自定义图片的展示样式，也可以通过代码`.setHierarchy(hierarchy)`和`.setController(controller)`自定义。

##### (1) initialize(Supplier<? extends SimpleDraweeControllerBuilder> draweeControllerBuilderSupplier)
初始化`mSimpleDraweeControllerBuilder`的供应器`sDraweeControllerBuilderSupplier`。需要在使用`SimpleDraweeView`前调用，官方把该初始化放在`Fresco.initializeDrawee(Context context)`。
##### (2) setImageURI(Uri uri, Object callerContext)
设置URI。内部通过调用``mSimpleDraweeControllerBuilder`构建一个控制器同时为控制器设置URI，然后调用`setController(controller)`为绘图层设置控制器,函数参数:
**uri:** 图片的 uri。支持的URIs: http://, https://, file://, content://, asset://, res://。具体介绍可见[Supported URIs](http://frescolib.org/docs/supported-uris.html#_)。
**callerContext:** 调用上下文。可以为`null`。
##### (3) setImageURI(Uri uri)
内部调用`setImageURI(Uri uri, Object callerContext)`，`callerContext`设为空
##### (4) shutDown()
关闭供应器。把`sDraweeControllerBuilderSupplier`设为空

##### 2.2.2 GenericDraweeHierarchy
视图层，用于组织和维护最终绘制和呈现的`Drawable`对象。可自定义占位图、加载失败占位图、重新加载占位图、进度条、背景、叠加图、按压状态下的叠加图等多个呈现效果。
##### (1) getActualImageBounds(RectF outBounds)
获取实际图片的后缩放边界。注：返回的边界不裁剪。参数：
**outBounds:** 用于填充边界
##### (2) getRoundingParams()
获取圆形配置参数`RoundingParams`。
##### (3) getTopLevelDrawable()
返回视图层最顶部的`drawable`。
##### (4) reset()
恢复初始状态。通过`controller`调用。
##### (5) setActualImageColorFilter(ColorFilter colorfilter)
为实际图片添加颜色过滤器。参数：
**colorfilter:** 颜色过滤器。
##### (6) setActualImageFocusPoint(PointF focusPoint)
设置实际图片聚焦点。参数：
**focusPoint:** 聚焦点。同时需要把实际图片的缩放类型设置为`ScaleType.FOCUS_CROP`。范围：左上角为(0,0)，右下角为(1,1)。
##### (7) setActualImageScaleType(ScalingUtils.ScaleType scaleType)
设置实际图片的缩放类型。参数：
**scaleType:** 缩放类型。具体可见`2.2.4 ScalingUtils.ScaleType`
##### (8) setControllerOverlay(Drawable drawable)
设置控制器覆盖层。由`controller`调用。参数：
**drawable:** 用作展示控制器覆盖层。
##### (9) setFailure(Throwable throwable)
当实际图片完全获取失败，则显示对应的占位符。由`controller`调用。参数：
**throwable:** 失败异常
##### (10) setImage(Drawable drawable, boolean immediate, int progress)
设置实际图片。由`controller`调用。参数：
**drawable:** 实际图片`drawable`。
**immediate:** 如果为`true`，则立即显示图片(没有渐变效果)。
**progress:** 进度条数值，范围［0，100］。当设置了`mProgressBarImage`，该值才有效。
##### (11) setPlaceholderImage((int resourceId)
设置占位图`drawable`。缩放类型不变。
##### (12) setPlaceholderImage(Drawable drawable)
参考(11)。
##### (13) setProgress(int progress, boolean immediate)
更新进度条。由`controller`调用。参数：
**progress:** 进度条数值，范围［0，100］。当设置了`mProgressBarImage`，该值才有效。如果需要隐藏进度条，可把值设为100。
**immediate:** 如果为`true`，则立即显示图片(没有渐变效果)。
##### (14) setRetry(Throwable throwable)
渐显重新加载占位图`mRetryImage`，如果`mRetryImage`没设置则显示占位图`mPlaceholderImage`。由`controller`调用。
##### (15) setRoundingParams(RoundingParams roundingParams)
设置圆形参数。

##### 2.2.3 GenericDraweeHierarchyBuilder
Builder模式，用于构建`GenericDraweeHierarchy`。

**主要属性：** 
##### (1). static final int DEFAULT_FADE_DURATION
渐变效果时间默认时间为300。
##### (2). static final ScalingUtils.ScaleType DEFAULT_SCALE_TYPE
默认缩放类型为`ScaleType.CENTER_INSIDE`。
##### (3). static final ScalingUtils.ScaleType DEFAULT_ACTUAL_IMAGE_SCALE_TYPE
默认实际占位图缩放类型为`ScaleType.CENTER_CROP`。
##### (4). int mFadeDuration
渐变效果时间。默认为`DEFAULT_FADE_DURATION`。
##### (5). Drawable mPlaceholderImage
占位图`drawable`。默认为`null`。
##### (6). ScaleType mPlaceholderImageScaleType
占位图缩放类型。默认为`null`。
##### (7). Drawable mRetryImage
重新加载占位图。默认为`null`。
##### (8). ScaleType mRetryImageScaleType
重新加载占位图缩放类型。默认为`null`。
##### (9). Drawable mFailureImage
失败占位图。默认为`null`。
##### (10). ScaleType mFailureImageScaleType
失败占位图缩放类型。默认为`null`。
##### (11). Drawable mProgressBarImage
进度条占位图。默认为`null`。
##### (12). ScaleType mProgressBarImageScaleType
进度条占位图缩放类型。默认为`null`。
##### (13). ScaleType mActualImageScaleType
实际图片缩放类型。默认为`DEFAULT_ACTUAL_IMAGE_SCALE_TYPE`。
##### (14). Matrix mActualImageMatrix
实际图片变换矩阵。默认为`null`。
##### (15). PointF mActualImageFocusPoint
实际图片缩放聚焦点。当缩放类型为`FOCUS_CROP`才有效。默认为`null`。
##### (16). ColorFilter mActualImageColorFilter
实际图片颜色过滤器。默认为`null`。
##### (17). List<Drawable> mBackgrounds
背景图片队列。默认为`null`。
##### (18). List<Drawable> mOverlays
覆盖层队列。默认为`null`。
##### (19). Drawable mPressedStateOverlay
按压状态下的覆盖层。默认为`null`。
##### (20). RoundingParams mRoundingParams
圆角配置参数。默认为`null`。

**主要函数：**
##### (1). GenericDraweeHierarchy build()
构建`GenericDraweeHierarchy`。代码如下：
```java
  public GenericDraweeHierarchy build() {
    validate();
    return new GenericDraweeHierarchy(this);
  }
```
##### (2). reset()
Builder模式，重置`builder`状态。内部调用`init()`。
##### (3). setActualImageColorFilter(ColorFilter colorFilter)
Builder模式，设置实际图片颜色过滤器。
##### (4). setActualImageFocusPoint(PointF focusPoint)
Builder模式，设置实际图片聚焦点。
##### (5). setActualImageMatrix(Matrix actualImageMatrix)
Builder模式，设置实际图片的变换矩阵，并移除缩放类型。
##### (6). setActualImageScaleType(ScalingUtils.ScaleType actualImageScaleType)
Builder模式，设置实际图片的缩放类型，并移除变换矩阵。
##### (7). setBackground(Drawable background)
Builder模式，设置单一背景。
##### (8). setBackground(List<Drawable> backgrounds)
Builder模式，设置背景。在层级结构和覆盖层被绘制前，按队列顺序绘制背景，第一个背景绘制在最底层。
##### (9). setFadeDuration(int fadeDuration)
Builder模式，设置渐变动画的时间。
##### (10). setFailureImage(Drawable failureDrawable)
Builder模式，设置失败占位图，其默认缩放类型为`DEFAULT_SCALE_TYPE`。
##### (11). setFailureImage(Drawable failureDrawable, ScalingUtils.ScaleType failureImageScaleType)
Builder模式，设置失败占位图和其缩放类型。
##### (12). setOverlay(Drawable overlay)
Builder模式，设置单一覆盖层。
##### (13). setOverlay(List<Drawable> overlays)
Builder模式，设置覆盖层。在层级结构和背景层被绘制后，按队列顺序绘制覆盖层，最后一个覆盖层绘制在最顶层。
##### (14). setPlaceholderImage(Drawable placeholderDrawable, ScalingUtils.ScaleType placeholderImageScaleType)
Builder模式，设置占位图和其缩放类型。默认使用透明的`ColorDrawable`。
##### (15). setPlaceholderImage(Drawable placeholderDrawable)
Builder模式，设置占位图，其默认缩放类型为`DEFAULT_SCALE_TYPE`。占位图默认使用透明的`ColorDrawable`。
##### (16). setPressedStateOverlay(Drawable drawable)
Builder模式，设置按压状态下的覆盖层。
##### (17). setProgressBarImage(Drawable progressBarImage, ScalingUtils.ScaleType progressBarImageScaleType)
Builder模式，设置进度条和其缩放类型。
##### (18). setProgressBarImage(Drawable progressBarImage)
Builder模式，设置进度条，其默认缩放类型为`DEFAULT_SCALE_TYPE`。
##### (19). setRetryImage(Drawable retryDrawable, ScalingUtils.ScaleType retryImageScaleType)
Builder模式，设置重新加载占位图和其缩放类型。
##### (20). setRetryImage(Drawable retryDrawable)
Builder模式，设置重新加载占位图，其默认缩放类型为`DEFAULT_SCALE_TYPE`。
##### (21). setRoundingParams(RoundingParams roundingParams)
Builder模式，设置圆角参数。

##### 2.2.4 ScalingUtils.ScaleType
图片缩放类型:

- CENTER:边界内居中,不缩放。
- CENTER_CROP:边界内居中，缩放子类使得两个尺寸(宽和高)大于或等于父类对应尺寸，并且至少有一个尺寸完全适合。
- CENTER_INSIDE:边界内居中，保留宽高比，缩放子类使其完全适应父类，如果子类比父类小，不需要按比例缩放。
- FIT_CENTER:边界内居中，保留宽高比，缩放子类使其完全适应父类，并且至少有一个尺寸完全适合。
- FIT_END:子类对其分类右下角，保留宽高比，缩放子类使其完全适应父类，并且至少有一个尺寸完全适合。
- FIT_START:子类对其分类左上角，保留宽高比，缩放子类使其完全适应父类，并且至少有一个尺寸完全适合。
- FIT_XY:分别缩放宽和高，使其完全适应父类。这将会改变宽高比。
- FOCUS_CROP:缩放子类使得两个尺寸(宽和高)大于或等于父类对应尺寸，并且至少有一个尺寸完全适合。
在父类边界内，尽可能不留空白区域，以子类的聚焦点居中。如果聚焦点设置为（0.5F，0.5F），则相当于CENTER_CROP。

##### 2.2.5 DraweeController
视图层控制器接口。把视图`view`的事件转发到控制器，控制器基于这些事件控制层级绘制。
主要接口:
##### (1).  Animatable getAnimatable()
返回`Animatable`,让客户端能够控制该动画。
##### (2). DraweeHierarchy getHierarchy()
返回`hierarchy`。
##### (3). onAttach()
当包含层级结构`hierarchy`的视图被暂时或永久附加到`window`窗口时的回调接口。
##### (4). onDetach()
当包含层级结构`hierarchy`的视图暂时或永久与`window`窗口分离时的回调接口。
##### (5). onTouchEvent(MotionEvent event) 
当包含层级结构`hierarchy`的视图接收到一个触摸事件时的回调接口。参数：
**event:** 触摸事件。
##### (6). setHierarchy(DraweeHierarchy hierarchy)
设置新的层级结构`hierarchy`。

##### 2.2.6 SimpleDraweeControllerBuilder
简单的视图控制生成器接口。
##### (1). DraweeController build()
生成视图控制器。
##### (2). setCallerContext(Object callerContext)
Builder模式，设置回调上下文。
##### (3). setOldController(DraweeController oldController)
Builder模式，设置新的视图控制器时，需要重用旧的视图控制器的内存，这样可节省不必要的内存分配。
##### (4). setUri(Uri uri)
Builder模式，设置图片`uri`。
 
##### 2.2.7 DraweeHierarchy
`drawee hierarchy` 接口。视图结构层的基类，用于组织和维护最终绘制和呈现的Drawable对象。
##### (1). Drawable getTopLevelDrawable()
返回是涂层顶部`drawable`。

##### 2.2.6 DraweeHolder
视图控制器`DraweeController`和视图结构器`DraweeHierarchy`的持有者，用于处理`DraweeView`的回调事件。
##### (1). static DraweeHolder<DH> create(DH hierarchy, Context context)
创建一个新的`DraweeHolder`实例。当`activity`触发`onStop`和`onStart`方法时，将会通知该实例回调控制器`onDetach`或`onAttach`方法。
参数：
**hierarchy:** 视图结构层`DraweeHierarchy`。在`DraweeHolder`类里，只有一个`DraweeHierarchy`单例。
**context:** 官方建议转换为`ListenableActivity`作为上下文`contetxt`。该方法将会调用`registerWithContext(contetxt)`，不过已经被弃用。
##### (2). DraweeController getController()
返回控制器`controller`，如果没设置则返回`null`。
##### (3). DH getHierarchy()
返回`hierarchy`，，如果没设置则抛出空指针异常。
##### (4). Drawable getTopLevelDrawable()
返回顶层视图，如果没设置则返回`null`。
##### (5). boolean hasHierarchy()
判断是否设置视图结构层`hierarchy`。
##### (6). void onAttach()
通知控制器进入展示图片状态。在`DraweeView`中，`onFinishTemporaryDetach()`和`onAttachedToWindow()`必须调用此方法。
##### (7). void onDetach()
释放使用的图片资源。在`DraweeView`中，`onStartTemporaryDetach()`和`onDetachedFromWindow()`必须调用此方法。
##### (8). void onDraw()
顶部视图`draw(canvas)`的回调接口。
##### (9). boolean onTouchEvent(MotionEvent event)
`view`的触摸事件，内部调用`controller`的`onTouchEvent`方法。
##### (10). void onVisibilityChange(boolean isVisible)
顶部视图可见性改变的回调接口。
##### (11). void registerWithContext (Context context)
已弃用。
##### (12). void setController (DraweeController draweeController)
设置新的控制器。
##### (13) void setHierarchy (DH hierarchy)
设置`drawee hierarchy`。
