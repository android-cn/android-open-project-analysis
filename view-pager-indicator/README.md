
ViewPagerindicator 源码解析
----------------
> 本文为 [Android 开源项目实现原理解析](https://github.com/android-cn/android-open-project-analysis) 中 ViewPagerindicator 部分  
> 项目地址：[ViewPagerIndicator](https://github.com/JakeWharton/Android-ViewPagerIndicator/)，分析的版本：[8cd549f](https://github.com/JakeWharton/Android-ViewPagerIndicator/commit/8cd549f23f3d20ff920e19a2345c54983f65e26b "Commit id is 8cd549f23f3d20ff920e19a2345c54983f65e26b")，Demo 地址：[ViewPagerIndicator Demo](https://github.com/android-cn/android-open-project-demo/tree/master/viewpager-indicator-demo)  
> 分析者：[lightSky](https://github.com/lightSky)，校对者：[aaronplay](https://github.com/AaronPlay)，校对状态：完成   


### 1. 功能介绍

### 1.1 ViewPagerIndicator  
ViewPagerIndicator用于各种基于AndroidSupportLibrary中ViewPager的界面导航。主要特点：使用简单、效果好、样式全。

### 2. 总体设计
该项目总体设计非常简单，一个pageIndicator接口类，具体样式的导航类实现该接口，然后根据具体样式去实现相应的逻辑。
IcsLinearLayout：LinearLayout的扩展，支持了4.0以上的divider特性。
CirclePageIndicator、LinePageIndicator、UnderlinePageIndicator、TitlePagerIndicator继承自View。TabPageIndicator、IconPageIndicator 继承自HorizontalScrollView。

CirclePageIndicator、LinePageIndicator、UnderlinePageIndicator继承自View的原因是它们样式相对简单继承自View，定制一套测量和绘制逻辑更简单，而且免去了Measure部分繁琐的步骤，效率更高。  
TitlePagerIndicator相对复杂，Android系统提供的控件中没有类似的，而且实现底部line精准的控制也复杂，所以只能继承自View，实现绘制逻辑，达到理想的效果。      

TabPageIndicator、IconPageIndicator继承自HorizontalScrollView是由于它们的ChildView会复杂些，继承自LinearLayout，一个个add上去更简单，而且当tab比较多的时候，也不用自己处理水平滑动的功能。  
### 3. 详细设计    
####3.1类关系图
![viewpagerindicator img](image/class_relation.png)  
####3.2 自定义控件相关知识  
由于ViewPagerIndicator项目全部都是自定义View，因此对于其原理的分析，就是对自定义View的分析，自定义View涉及到的核心部分有：View的绘制机制和Touch事件传递机制。对于View的绘制机制，这里做了详细的阐述，而对于Touch事件，由于该项目只是Indicator，因此没有涉及到复杂的Touch传递机制，该项目中与Touch机制相关只有onTouch(Event)方法，因此只对该方法涉及到的相关知识进行介绍。
####3.2.1 自定义控件步骤  
以下直接引用了其中的一些篇章前言，具体内容大家直接点击链接进入正文

1. 自定义控件创建步骤  
	* 继承View或View的子类，同时为了允许layouteditor创建并编辑你的view的实例，你需要提供至少一个构造函数，它应该包含一个Contenx与一个AttributeSet对象作为参数。
	* 定义自定义的属性（外观与行为），在values/下建立styleable的xml文件，具体可参考ViewPagerIndicator库中的vpi_attrs.xml文件 。
	* 应用自定义的属性，在构造函数中，做一些初始化，具体可看考。
	* 添加属性和事件。

2. [自定义View的绘制](http://hukai.me/android-training-course-in-chinese/ui/custom-view/custom-draw.html)  
自定义view的最重要的一个部分是自定义它的外观。根据你的程序的需求，自定义绘制动作可能简单也可能很复杂。
重绘一个自定义的view的最重要的步骤是重写onDraw()方法。onDraw()的参数是一个Canvas对象。Canvas类定义了绘制文本，线条，图像与许多其他图形的方法。你可以在onDraw方法里面使用那些方法来创建你的UI。

3. [使View可交互](http://hukai.me/android-training-course-in-chinese/ui/custom-view/make-interactive.html)  
绘制UI仅仅是创建自定义View的一部分。你还需要使得你的View能够以模拟现实世界的方式来进行反馈。Objects应该总是与现实情景能够保持一致。用户应该可以感受到UI上的微小变化，并对这些变化反馈到现实世界中。例如，当用户做fling(迅速滑动)的动作，应该在滑动开始与结束的时候给用户一定的反馈。像许多其他UI框架一样，Android提供一个输入事件模型。在Android中最常用的输入事件是touch，它会触发onTouchEvent(android.view.MotionEvent))的回调。重写这个方法来处理touch事件：

4. [Android的用户输入](http://hukai.me/android-training-course-in-chinese/best-user-input.html)  
这里着重要看一下拖拽与缩放这一部分。因为在ViewPagerIndicator的几种实现：Circle，Title，UnderLine的onTouchEvent里的处理逻辑是一样的，而且和官方文档中的代码逻辑也是一样的，看了讲解之后，相信大家就会有所了解了：http://hukai.me/android-training-course-in-chinese/input/gestures/scale.html
    
####3.2.2 View绘制机制  
请直接参考[公共技术点viewdrawflow](https://github.com/android-cn/android-open-project-analysis/blob/master/tech/viewdrawflow.md)部分  
####3.2.3 Android的用户输入  	
要注意的有以下几点（更详细的介绍可以参考[拖拽与缩放](http://hukai.me/android-training-course-in-chinese/input/gestures/scale.html)部分）


##### 3.2.3.1 保持对最初点的追踪  
拖拽操作时，即使有额外的手指放置到屏幕上了，app也必须保持对最初的点（手指）的追踪。比如，想象在拖拽图片时，用户放置了第二根手指在屏幕上，并且抬起了第一根手指。如果你的app只是单独地追踪每个点，它会把第二个点当做默认的点，并且把图片移到该点的位置。
	
##### 3.2.3.2 区分原始点及之后的任意触摸点   
为了防止这种情况发生，你的app需要区分初始点以及之后任意的触摸点。要做到这一点，它需要追踪处理多触摸手势中提到过的ACTION_POINTER_DOWN、 ACTION_POINTER_UP事件。每当第二根手指按下或拿起时，ACTION_POINTER_DOWN、ACTION_POINTER_UP事件就会传递给onTouchEvent())回调函数。
	
##### 3.2.3.3 确保操作中的点的ID(the active pointer ID)不会引用已经不在触摸屏上的触摸点  

当ACTION_POINTER_UP事件发生时，示例程序会移除对该点的索引值的引用，确保操作中的点的ID(the active pointer ID)不会引用已经不在触摸屏上的触摸点。这种情况下，app会选择另一个触摸点来作为操作中(active)的点，并保存它当前的x、y值。由于在ACTION_MOVE事件时，这个保存的位置会被用来计算屏幕上的对象将要移动的距离，所以app会始终根据正确的触摸点来计算移动的距离。

mTouchSlop  
指在用户触摸事件可被识别为移动手势前,移动过的那一段像素距离。Touchslop通常用来预防用户在做一些其他操作时意外地滑动，例如触摸屏幕上的元素时。

[onTouchEvent](http://hukai.me/android-training-course-in-chinese/input/gestures/scale.html)		
对于pointer的处理是模板方法，在拖拽与缩放中有详细的讲解。ViewPagerIndicator中的onTouchEvent中的代码也就是官方文档的模板代码，就是为了确保以上几点，获取到可用、确信的点，然后处理ViewPager相应的偏移和滑动。
  
####3.2.4 CirclePageIndicator 源码分析  

CirclePageIndicator.java  
继承自 View 实现了 PageIndicator,整个绘制过程中用到的方法调用规则为：  
![circle_indicator_method_flow img](image/circle_indicator_method_flow.png)  
**(1) 主要成员变量含义**  
1.`mCurrentPage` 当前界面的索引  
2.`mSnapPage` Sanp模式下，当前界面的索引  
3.`mPageOffset` ViewPager的水平偏移量  
4.`mScrollState` ViewPager的滑动状态  
5.`mOrientation` Indicator的模式：水平、竖直  
6.`mLastMotionX` 每一次onTouch事件产生时水平位置的最后偏移量  
7.`mActivePointerId` 当前处于活动中pointer的ID默认值为 -1  
8.`mIsDragging` 用户是否主观的滑动屏幕的标识  
9.`mSnap`   
circle有2种绘制模式:  
     mSnap = true：ViewPager滑动过程中，circle之间不绘制，只绘制最终的实心点  
     mSnap = false：ViewPager滑动过程中，相邻circle之间根据mPageOffset实时绘制circle    
10.`mTouchSlop`   
指在用户触摸事件可被识别为移动手势前,移动过的那一段像素距离。    
Touchslop通常用来预防用户在做一些其他操作时意外地滑动，例如触摸屏幕上的元素时产生的滑动。

**(2) 核心方法**    
1.**onDraw(Canvas canvas) **   
`threeRadius`两相邻circle的间距  
`shortOffset`当前方向的垂直方向的圆心坐标位置  
`longOffset` 当前方向的圆心位置
```java
	//循环的 draw circle
        for (int iLoop = 0; iLoop < count; iLoop++) {
            float drawLong = longOffset + (iLoop * threeRadius);//计算当前方向的每个circle偏移量
            canvas.drawCircle(dX, dY, pageFillRadius, mPaintPageFill);//绘制空心的circle
            canvas.drawCircle(dX, dY, mRadius, mPaintStroke);//绘制stroke
            ...
            计算实心的circle的坐标
            ...
            canvas.drawCircle(dX, dY, mRadius, mPaintFill);//绘制当前page的circle
        }
```
2.**onTouchEvent(MotionEvent ev)**  
这一部分为模板代码，在其它几种Indicator的实现中，对于Touch的事件的处理是相同的  
`MotionEvent.ACTION_DOWN`:记录第一触摸点的ID,获取当前水平移动距离  
`MotionEvent.ACTION_MOVE`: 获取第一点的索引并计算其偏移，处理用户是否是主观的滑动屏幕  
`MotionEvent.ACTION_CANCEL`:如果用户不是主动滑动，则以ViewPager的宽带的1/3为临界点进行previous和next  
page的处理，最后还原mIsDragging，mActivePointerId、viewpager的fakeDragging状态  
`MotionEvent.ACTION_UP`:同上    
`MotionEventCompat.ACTION_POINTER_DOWN`:当除最初点外的第一个外出现在屏幕上时，触发该事件，这时记录新的mLastMotionX，mActivePointerId  
`MotionEventCompat.ACTION_POINTER_UP`:当非第一点离开屏幕时，获取抬起手指的ID，如果之前跟踪的mActivePointerId是当前抬起的手指ID，那么就重新为mActivePointerId 赋值另一个活动中的pointerId，最后再次获取仍活动在屏幕上pointer的X坐标值  

3.**onMeasure(int widthMeasureSpec, int heightMeasureSpec)**  
View在测量阶段的最终大小的设定是由setMeasuredDimension()方法决定的,也是必须要调用的方法，否则会报异常，这里就直接调用了setMeasuredDimension()方法设置值了。根据CircleIndicator的方向，计算相应的width、height  
```java
        if (mOrientation == HORIZONTAL) {
            setMeasuredDimension(measureLong(widthMeasureSpec), measureShort(heightMeasureSpec));
        } else {
            setMeasuredDimension(measureShort(widthMeasureSpec), measureLong(heightMeasureSpec));
        }
    }
```    
4.**measureLong**    
与之对应的有measureShort，只是处理的方向不同  
如果该View的测量要求为EXACTLY，则能直接确定子View的大小,该大小就是MeasureSpec.getSize(measureSpec)的值  
如果该View的测量要求为UNSPECIFIED或AT_MOST模式，则根据实际需求计算宽度  
```java
        if ((specMode == MeasureSpec.EXACTLY) || (mViewPager == null)){
            //We were told how big to be
            result = specSize;
        } else {
            final int count = mViewPager.getAdapter().getCount();
            result = (int)(getPaddingLeft() + getPaddingRight()
                    + (count * 2 * mRadius) + (count - 1) * mRadius + 1);
            //如果父视图的测量要求为AT_MOST，即限定了一个最大值，则再从系统建议值和自己计算值中取一个较小值
            if (specMode == MeasureSpec.AT_MOST) {
                result = Math.min(result, specSize);
            }
        }
        return result;
```
##4 杂谈##
大多数的App中的导航都类似，ViewPagerIndicator能够满足你开发的基本需求，如果不能满足，你可以在源码的基础上进行一些简单的改造。其中有一点是很多朋友提出的就是LineIndicator没有实现TextView颜色状态的联动。这个有已经实现的开源库:[PagerSlidingTabStrip](https://github.com/jpardogo/PagerSlidingTabStrip)，你可以作为参考。  
对于什么时候需要自定义控件以及如何更好的进行自定义控件的定制，你可以参考这篇文章[深入解析Android的自定义布局](http://greenrobot.me/devpost/android-custom-layout) 相信会有一些启发。  
整片文章看下来，确实比较多，也是花了一部分时间写的，其实之前是自己整理了一些相关知识，这次一下全部跟大家分享了。整篇文章都在讲View的绘制机制，三个过程也都很详细的通过源码分析介绍了。如果你对View的绘制机制还不清楚，而且希望将来往更高级的方向发展，这一步一定会经历的，那么请你耐心看完，你可以分多次研读，过程中出现问题或者原文分析不到位的地方，欢迎PR。  
当你掌握了这些基本的知识，你可以去研究GitHub上的一部分开源项目了（因为Touch事件这里介绍的不多，而很多项目和Touch事件相关）。

参考文献  
[how-android-draws](http://developer.android.com/guide/topics/ui/how-android-draws.html)  
[Google Android官方培训课程中文版](https://github.com/kesenhoo/android-training-course-in-chinese)  

View的绘制：  
http://blog.csdn.net/wangjinyu501/article/details/9008271  
http://blog.csdn.net/qinjuning/article/details/7110211  
http://blog.csdn.net/qinjuning/article/details/8074262
  	
Touch事件传递：  
http://blog.csdn.net/xiaanming/article/details/21696315  
http://blog.csdn.net/wangjinyu501/article/details/22584465    

相关资源：  
[best-practices-for-android-user-interface](http://www.rapidvaluesolutions.com/tech_blog/best-practices-for-android-user-interface/)    
[深入解析Android的自定义布局](http://greenrobot.me/devpost/android-custom-layout/)  
[慕课网自定义FlowLayout课程](http://www.imooc.com/learn/237)

