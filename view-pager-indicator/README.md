open-project-analysis
=====================

### 1. 简介

**功能介绍：**  
引用[viewpagerindicator](http://viewpagerindicator.com)项目主页的简短介绍：ViewPager适用于Android Support Library中的ViewPager和其开发的ActionBarSherlock，项目基于Patrik Åkerfeldt's ViewFlow。  
应用于各种界面的导航。  
	

### 2. 主要特点：
	* 使用简单
	* 效果好 

### 3. 详细设计
	

**归类**  
	CirclePageIndicator、LinePageIndicator、UnderlinePageIndicator 这三种是极其相似的。可以归为一类。
	TabPageIndicator、IconPageIndicator 继承自HorizontalScrollView，极其相似，可以归为一类。
	TitlePagerIndicator：实现最复杂,单独介绍。
		
**结构**  
	PageIndicator，接口类，定义了不同类型的Indicator的公用的方法。
	IcsLinearLayout ：LinearLayout的扩展，支持了4.0以上的divider特性。在TabPagerIndicator中使用到。

**实现**  
	这里先介绍两个模块：自定义控件的创建和Android的用户输入。  
介绍这两个模块的原因是:所有的可交互的自定义控件一定都会有这两个模块，另外，介绍完这两个模块后，我相信，大家把这两个模块的基本知识点都看明白后，一定都能看懂ViewPagerIndicator的实现了。本人只做简单的介绍，就不再详细的赘述了。  

大家可能会担心这些知识点到哪里才能找到最权威和最系统的讲解，其实官方文档就有，到如今我也没有很耐心的把官方文档看个遍。已已经有好心人把它翻译了：一个GitHub协作项目android-training-course-in-chinese，该项目就是对Google的官方文档进行翻译，现已有PDF版本了，大家可以去下载。（强烈建议大家直接去下载。然后慢慢咀嚼，一定会Android的整体知识架构有清晰的认识。我觉得特别有帮助，最好的学习资料莫过官方文档）
		
Google官方中文文档翻译项目地址：[android-training-course-in-chinese](https://github.com/kesenhoo/android-training-course-in-chinese)

		
下面就简单的介绍一下整个流程:
		（以下的分析中就直接引用了其中的一些篇章前言，具体内容大家直接点击链接就可以看到，这里就不再赘述，建议大家做下笔记，内容还是比较多的）

1. [自定义控件创建步骤](http://hukai.me/android-training-course-in-chinese/ui/custom-view/index.html) ：  
	* 继承一个View。
	* 定义自定义的属性（外观与行为）。
	* 应用自定义的属性。
	* 添加属性和事件。

2. [自定义View的绘制](http://hukai.me/android-training-course-in-chinese/ui/custom-view/custom-draw.html)  
自定义view的最重要的一个部分是自定义它的外观。根据你的程序的需求，自定义绘制动作可能简单也可能很复杂。
重绘一个自定义的view的最重要的步骤是重写onDraw()方法。onDraw()的参数是一个Canvas对象。Canvas类定义了绘制文本，线条，图像与许多其他图形的方法。你可以在onDraw方法里面使用那些方法来创建你的UI。

3. [使View可交互](http://hukai.me/android-training-course-in-chinese/ui/custom-view/make-interactive.html)  
绘制UI仅仅是创建自定义View的一部分。你还需要使得你的View能够以模拟现实世界的方式来进行反馈。Objects应该总是与现实情景能够保持一致。用户应该可以感受到UI上的微小变化，并对这些变化反馈到现实世界中。例如，当用户做fling(迅速滑动)的动作，应该在滑动开始与结束的时候给用户一定的反馈。像许多其他UI框架一样，Android提供一个输入事件模型。在Android中最常用的输入事件是touch，它会触发onTouchEvent(android.view.MotionEvent))的回调。重写这个方法来处理touch事件：

4. [Android的用户输入](http://hukai.me/android-training-course-in-chinese/best-user-input.html)  
这里着重要看一下拖拽与缩放这一部分。因为在ViewPagerIndicator的几种实现：Circle，Title，UnderLine的onTouchEvent里的处理逻辑是一样的，而且和官方文档中的代码逻辑也是一样的，看了讲解之后，相信大家就会有所了解了：http://hukai.me/android-training-course-in-chinese/input/gestures/scale.html


**TabPageIndicator、IconPageIndicator**

继承自HorizontalScrollView,使用了IcsLinearLayout
继承自HorizontalScrollView，拥有其左右滑动的效果，以及其它以实现的操作，比如对child的measure

**IconPageIndicator**  
	notifyDataSetChanged：
		像IcsLinearLayout中添加child,同时请求requestLayout(),该方法会触发measure和layout这两个步骤，measure是测绘每个看控件的大小，layout是为了给每个View确定好位置，有了大小和位置后，接下来就是draw了。

	onAttachedToWindow：
		开辟一块绘画层，即将开始draw。应确保在onDraw之前调用，但在measure之前和之后调用都可。
	onDetachedFromWindow
		当View 从window脱离时被调用，之时就不再拥有绘画层。
		在这两个方法里主要针对Runnable 做操作，从消息队列中加入和移除该message。  
	removeCallbacks（）；可以再UI线程之外触发，但只有在View 依附到window时。
	
**TabPageIndicator**  
	onMeasure：MeasureSpec 的三种方式及处理
	setFillViewport：测量子View，以便其填充可见区域。这里除EXACTLY模式外，都填充。


**CirclePageIndicator、LinePageIndicator、UnderlinePageIndicator**

**mTouchSlop**  
	“Touch slop”是指在用户触摸事件可被识别为移动手势前,移动过的那一段像素距离。Touchslop通常用来预防用户在做一些其他操作时意外地滑动，例如触摸屏幕上的元素时。

[onTouchEvent](http://hukai.me/android-training-course-in-chinese/input/gestures/scale.html)		
	对于pointer的处理是模板方法，在拖拽与缩放中有详细的讲解：
	
要注意的有以下几点（结合原文以及代码，立马就清晰了）：
		
**1. 保持对最初点的追踪**  
拖拽操作时，即使有额外的手指放置到屏幕上了，app也必须保持对最初的点（手指）的追踪。比如，想象在拖拽图片时，用户放置了第二根手指在屏幕上，并且抬起了第一根手指。如果你的app只是单独地追踪每个点，它会把第二个点当做默认的点，并且把图片移到该点的位置。
	
**2. 区分原始点及之后的任意触摸点**   
为了防止这种情况发生，你的app需要区分初始点以及之后任意的触摸点。要做到这一点，它需要追踪处理多触摸手势中提到过的ACTION_POINTER_DOWN、 ACTION_POINTER_UP事件。每当第二根手指按下或拿起时，ACTION_POINTER_DOWN、ACTION_POINTER_UP事件就会传递给onTouchEvent())回调函数。
	
**3. 确保操作中的点的ID(the active pointer ID)不会引用已经不在触摸屏上的触摸点**  
当ACTION_POINTER_UP事件发生时，示例程序会移除对该点的索引值的引用，确保操作中的点的ID(the active pointer ID)不会引用已经不在触摸屏上的触摸点。这种情况下，app会选择另一个触摸点来作为操作中(active)的点，并保存它当前的x、y值。由于在ACTION_MOVE事件时，这个保存的位置会被用来计算屏幕上的对象将要移动的距离，所以app会始终根据正确的触摸点来计算移动的距离。

ViewPagerIndicator中的onTouchEvent中的代码也就是官方文档的模板代码，就是为了确保以上几点，拿到可用，确信的点然后处理ViewPager相应的偏移和滑动

**核心函数**  
onDraw  onTouchEvent

这里就算对ViewPagerIndicator的实现原理介绍完毕了。可能大家说我图懒省事，其实并不然。第一，关于自定义控件的知识点并不是一下就能说清楚的。其内容之多，这里根本无法做详细的介绍。如果你把这些基础看明白了，一定会知道ViewPagerIndicator是如何实现的，其实该项目并不复杂。另外对于自定义控件的这些知识是非常重要的，真心想学习的同学，一定会认真研究这些基本的知识点，并做下笔记，因为在以后的学习或者研读其它优秀开源项目的时候，一定会遇到这些知识。
	
**授人以鱼不如授人以渔。真正学到的，才是自己的**

### 4. 使用方法

	在你的布局文件中，需要导航的位置添加类似如下代码：
```
	<com.viewpagerindicator.TitlePageIndicator
	    android:id="@+id/titles"
	    android:layout_height="wrap_content"
	    android:layout_width="fill_parent" />
```
	在你的Activity中的onCreate方法中将该Indicator绑定到ViewPager
```	
	 //Set the pager with an adapter
	 ViewPager pager = (ViewPager)findViewById(R.id.pager);
	 pager.setAdapter(new TestAdapter(getSupportFragmentManager()));
```
	绑定Indicator到Adapter
```
	 TitlePageIndicator titleIndicator = (TitlePageIndicator)findViewById(R.id.titles);
	 titleIndicator.setViewPager(pager);
```
	如果你实现了OnPageChangeListener，你应该将它设置到Indicator当中去，而不是直接设置到ViewPager上：

	`titleIndicator.setOnPageChangeListener(mPageChangeListener);`


