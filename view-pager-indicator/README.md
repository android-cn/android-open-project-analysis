
ViewPagerindicator 源码解析
----------------
> 本文为 [Android 开源项目实现原理解析](https://github.com/android-cn/android-open-project-analysis) 中 ViewPagerindicator 部分，项目地址：[viewpagerindicator](http://viewpagerindicator.com)，分析的版本：[8cd549](https://github.com/JakeWharton/Android-ViewPagerIndicator/commit/8cd549f23f3d20ff920e19a2345c54983f65e26b "Commit id is 8cd549f23f3d20ff920e19a2345c54983f65e26b")，分析者：[lightSky](https://github.com/lightSky)，校对者：[drakeet](https://github.com/drakeet)，校对状态：未完成   


### 1. 功能介绍

### 1.1 ViewPagerIndicator  
ViewPagerIndicator用于各种基于AndroidSupportLibrary中ViewPager的界面导航。主要特点：使用简单、效果好、样式全。

### 2. 总体设计
该项目总体设计非常简单，一个pageIndicator接口类，具体样式的导航类实现该接口，然后根据具体样式去实现相应的逻辑。
IcsLinearLayout：LinearLayout的扩展，支持了4.0以上的divider特性。
CirclePageIndicator、LinePageIndicator、UnderlinePageIndicator、TitlePagerIndicator继承自View。    
TabPageIndicator、IconPageIndicator 继承自HorizontalScrollView。  
### 3. 详细设计    
####3.1类关系图
![viewpagerindicator img](image/class_relation.png)  
####3.2 自定义控件相关知识  
####3.2.1 自定义控件步骤  
以下直接引用了其中的一些篇章前言，具体内容大家直接点击链接进入正文

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
    
####3.2.2 View绘制机制  
参考文献：http://developer.android.com/guide/topics/ui/how-android-draws.html

#####3.2.2.1  基础概念 
当Activity接收到焦点的时候，它会被请求绘制布局。Android framework将会处理绘制的流程，但Activity必须提供View层级的根节点。绘制是从根节点开始的，需要measure和draw布局树。绘制会遍历和渲染每一个与无效区域相交的view。相反，每一个ViewGroup负责绘制它所有的childrenview，而View会负责绘制自身。树的遍历是有序的，parent view要先于child View被绘制，

绘制布局有两步：measure和layout  
measure过程的实现在measure(int,int)方法中，而且从上到下的有序绘制view。在递归的过程中，每一个视图（View）将尺寸规格向下传递给View，在measure过程的最后，每个视图存储了它的尺寸。
layout过程从layout(int, int, int, int)方法开始，也是自上而下遍历。在这个过程中，每个parent view根据measure过程计算出来
的尺寸确定所有的child view具体位置。  

注意：Android框架不会绘制无效区域之外的部分,但会考虑绘制视图的背景。你可以使用invalidate()去强制对一个view进行重绘。  

当一个View的measure()执行完的时候，它自己以及所有的孩子节点的getMeasuredWidth()和getMeasuredHeight()方法的值就必须被设置了。一个视图的测量宽度和测量高度值必须在父视图约束范围之内，这可以保证在measure的最后,所有的父母都接受所有孩子的测量。
一个父视图，可以在其child view上多次的调用measure()方法。比如，父视图可以先根据未指明的dimension调用measure方法去测量每一个
child view的大小，如果所有child的未约束尺寸太大或者太小的时候，则会使用一个确切的大小，然后在每一个childview上再次调用measure方法去测量每一个view的大小。（也就是说，如果children对于获取到的大小不满意的时候，父视图会介入并设置测量规则进行第二次measure）

measure过程使用了两个类来传递尺寸：  
一个是ViewGroup.LayoutParams类（View自身的布局参数）  
一个是MeasureSpecs类（父视图对子视图的测量要求）

ViewGroup.LayoutParams  
被子视图用于告诉他们的父视图他们应该怎样被测量和放置（就是子视图自身的布局参数）。一个基本的LayoutParams只用来描述视图的高度和宽度。对于，每一方面的尺寸，你可以指定下列方式之一：  
1、具体数值   
2、MATCH_PARENT 表示子视图希望和父视图一样大(不含padding)   
3、WRAP_CONTENT 表示视图为正好能包裹其内容大小(包含padding)    

对于ViewGroup的子类，也有相应的ViewGroup.LayoutParams的子类，例如RelativeLayout有相应的ViewGroup.LayoutParams的子类,拥有设置子视图水平和垂直的能力。

MeasureSpecs  
用于从上到下传递父视图对子视图测量需求,其有三种模式:      

**UNSPECIFIED**  
父视图可以为子视图设置它所期望的大小。比如一个LinearLayout可以在它的子view上调用measure()方法去测量一个高设置为UNSPECIFIED模式，宽为240pixels的view大小。    

**EXACTLY**  
父视图决定子视图的确切大小，子视图必须使用该大小，并确保它所有的子视图可以适应在该尺寸的范围内；相对应属性的是MATCH_PARENT   

**AT_MOST**  
父视图为子视图指定一个最大值。子视图必须确保它自己的所有子视图在该尺寸范围内，相应的属性为WRAP_CONTENT   
 
#####3.2.2.2 measure核心方法  
measure(int widthMeasureSpec, int heightMeasureSpec)方法  
该方法定义在View.java类中，final修饰符修饰，因此不能被重载，但measure调用链会回调View/ViewGroup对象的onMeasure()方法，因此我们只需要复写onMeasure()方法去计算自己的控件尺寸即可。  
该方法的两个参数分别是父视图提供的测量规格MeasureSpec。当父视图调用子视图的measure函数对子视图进行测量时，会传入这两个参数。通过这两个参数以及子视图本身的LayoutParams来共同决定子视图的测量规格MeasureSpec。其实整个measure过程就是从上到下遍历，不断的根据父视图的MeasureSpec和子视图自身的LayotuParams获取子视图自己的MeasureSpec，最终调用子视图的measure(int widthMeasureSpec, int heightMeasureSpec)方法确定最终的mMeasuredWidth和mMeasuredHeight。ViewGroup的measureChildWithMargins函数中体现了这个过程。  
具体过程分析可以参考：http://blog.csdn.net/wangjinyu501/article/details/9008271

setMeasuredDimension()方法  
View在测量阶段的最终大小的设定是由setMeasuredDimension()方法决定的,该方法最终会对每个View的mMeasuredWidth和mMeasuredHeight进行赋值，一旦这两个变量被赋值，则意味着该View的测量工作结束，setMeasuredDimension()也是必须要调用的方法，否则会报异常。在setMeasuredDimension()方法内部，你可以根据需求，去计算View的尺寸。  

#####3.2.2.3 layout相关核心概念及方法  
子视图的具体位置都是相对与父视图的位置。与onMeasure过程类似，ViewGroup在onLayout函数中通过调用其children的layout函数来设置子视图相对与父视图中的位置，具体位置由函数layout的参数决定，当我们继承ViewGroup时必须重载onLayout函数（ViewGroup中onLayout是abstract修饰），然而onMeasure并不要求必须重载，因为相对与layout来说，measure过程并不是必须的。  

实现onLayout通常做法就是进行一个for循环调用每一个子视图的layout(l, t, r, b)函数，传入不同的参数l, t, r, b来确定每个子视图在父视图中的显示位置。onLayout过程会通过调用getMeasuredWidth()和getMeasuredHeight()方法获取到measure过程得到的mMeasuredWidth和mMeasuredHeight,这两个参数为layout过程提供了一个很重要的参考值（不是必须的）。    

之所以说measure过程不是必须的，是因为layout过程中的4个参数l, t, r,b完全可以由视图设计者任意指定，如果在自定义的onLayout中手动指定了layout的参数，而不用measure过程的值，也是可以的，当然一般没人会这么做，这样也违背了Android框架的绘制机制，通常的做法是根据需求在measure过程决定尺寸，layout步骤决定位置，除非你只是指定View的位置，而不考虑View的尺寸。  

#####3.2.2.4 绘制流程相关核心概念及方法    
draw过程在measure()和layout()之后进行，最终会调用到mView的draw()函数，这里的mView对于Actiity来说就是PhoneWindow.DecorView。  

首先来看下与draw过程相关的函数：  

ViewRootImpl.draw()：仅在ViewRootImpl.performTraversals()的内部调用

DecorView.draw()：ViewRootImpl.draw()会调用到该函数，DecorView.draw()继承自Framelayout由于DecorView、FrameLayout以及FrameLayout的父类ViewGroup都未复写draw(),而ViewGroup的父类是View，因此DecorView.draw()调用的就是View.draw()。

View.onDraw()：绘制View本身，自定义View往往会重载该函数来绘制View本身的内容

View.dispatchDraw()： View中的dispatchDraw是空实现，ViewGroup复写了该函数，内部循环调用View.drawChild()来发起对子视图的绘制，你不应该重载ViewGroup的dispatchDraw，因为该函数的默认实现代表了View的绘制流程

ViewGroup.drawChild()：该函数只在ViewGroup中实现，因为只有ViewGroup才需要绘制child，drawChild内部又会调用View.draw()来完成子视图的绘制（也有可能直接调用dispatchDraw）  

View.draw(Canvas)方法：  
```
 /**
     * Manually render this view (and all of its children) to the given Canvas.
     * The view must have already done a full layout before this function is
     * called.  When implementing a view, implement
     * {@link #onDraw(android.graphics.Canvas)} instead of overriding this method.
     * If you do need to override this method, call the superclass version.
     *
     * @param canvas The Canvas to which the View is rendered.  
     *
     * 根据给定的Canvas自动渲染View（包括其所有子View）。在调用该方法之前必须要完成layout。当你自定义view的时候，
     * 应该去是实现onDraw(Canvas)方法，而不是draw(canvas)方法。如果你确实需要复写该方法，请记得先调用父类的方法。
     */
    public void draw(Canvas canvas) {
    
        / * Draw traversal performs several drawing steps which must be executed
         * in the appropriate order:
         *
         *      1. Draw the background if need
         *      2. If necessary, save the canvas' layers to prepare for fading
         *      3. Draw view's content
         *      4. Draw children (dispatchDraw)
         *      5. If necessary, draw the fading edges and restore layers
         *      6. Draw decorations (scrollbars for instance)
         */

 	// Step 1, draw the background, if needed
        if (!dirtyOpaque) {
            drawBackground(canvas);
        }
        
        // Step 2, save the canvas' layers
        
        // Step 3, draw the content
        if (!dirtyOpaque) 
        	onDraw(canvas);

        // Step 4, draw the children
        dispatchDraw(canvas);

        // Step 5, draw the fade effect and restore layers
        
        // Step 6, draw decorations (scrollbars)
        onDrawScrollBars(canvas);
    }

```

源码中已经清楚的注释了整个绘制过程：  
View的背景绘制-----> View本身内容的绘制----->子视图的绘制（如果包含子视图）----->渐变框的绘制---->滚动条的绘制。  
onDraw()和dispatchDraw()分别为View本身内容和子视图绘制的函数。  
View和ViewGroup的onDraw()都是空实现，因为具体View如何绘制由设计者来决定的，默认不绘制任何东西。
ViewGroup复写了dispatchDraw()来对其子视图进行绘制，通常应用程序不应该对dispatchDraw()进行复写，其默认实现体现了View系统的绘制流程。

dispatchDraw()的核心代码就是通过for循环调用drawChild()对ViewGroup的每个子视图进行动画以及绘制:
```
dispatchDraw(Canvas canvas){

...

 if ((flags & FLAG_RUN_ANIMATION) != 0 && canAnimate()) {//处理ChildView的动画
 	final boolean buildCache = !isHardwareAccelerated();
            for (int i = 0; i < childrenCount; i++) {
                final View child = children[i];
                if ((child.mViewFlags & VISIBILITY_MASK) == VISIBLE) {
                    final LayoutParams params = child.getLayoutParams();
                    attachLayoutAnimationParameters(child, params, i, childrenCount);
                    bindLayoutAnimation(child);
                    if (cache) {
                        child.setDrawingCacheEnabled(true);
                        if (buildCache) {
                            child.buildDrawingCache(true);
                        }
                    }
                }
            }
            
 	final LayoutAnimationController controller = mLayoutAnimationController;
            if (controller.willOverlap()) {
                mGroupFlags |= FLAG_OPTIMIZE_INVALIDATE;
            }

	controller.start();//启动View的动画
}

 //绘制ChildView
 for (int i = 0; i < childrenCount; i++) {
            int childIndex = customOrder ? getChildDrawingOrder(childrenCount, i) : i;
            final View child = (preorderedList == null)
                    ? children[childIndex] : preorderedList.get(childIndex);
            if ((child.mViewFlags & VISIBILITY_MASK) == VISIBLE || child.getAnimation() != null) {
                more |= drawChild(canvas, child, drawingTime);
            }
        }

...

}

```
drawChild()方法：  
核心过程：为子视图分配合适的cavas剪切区，剪切区的大小正是由layout过程决定的，而剪切区的位置取决于滚动值以及子视图当前的动画。设置完剪切区后就会调用子视图的draw()函数进行具体的绘制，如果子视图的包含SKIP_DRAW标识，那么仅调用dispatchDraw()，即跳过子视图本身的绘制，但要绘制视图可能包含的字视图。完成了dispatchDraw()过程后，View系统会调用onDrawScrollBars()来绘制滚动条。

####3.2.3 Android的用户输入  	
要注意的有以下几点（更详细的介绍可以参考原文）  

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

```java
public class CirclePageIndicator extends View implements PageIndicator {
    private static final int INVALID_POINTER = -1;

    /**当前界面的索引*/
    private int mCurrentPage;

    /**当前界面的索引，和mCurrentPage值一样*/
    private int mSnapPage;

    /**ViewPager的水平偏移量 */
    private float mPageOffset;

    /**ViewPager的滑动状态 */
    private int mScrollState;

    /**Indicator的模式：水平、竖直*/
    private int mOrientation;

    /**每一次onTouch事件产生时水平位置的最后偏移量 */
    private float mLastMotionX = -1;

    /**当前处于活动中pointer的ID*/
    private int mActivePointerId = INVALID_POINTER;

    /** 用户是否主观的滑动屏幕的标识*/
    private boolean mIsDragging;

    /**
     * circle有2种绘制模式:
     * mSnap = true：circle之间不绘制，只绘制最终的实心点
     * mSnap = false：viewPager滑动过程中，相邻circle之间根据mPageOffset实时绘制circle
     */
    private boolean mSnap;

    /**
     * “Touch slop”是指在用户触摸事件可被识别为移动手势前,移动过的那一段像素距离。
     * Touchslop通常用来预防用户在做一些其他操作时意外地滑动，例如触摸屏幕上的元素时产生的滑动。
     */
    private int mTouchSlop;

    public CirclePageIndicator(Context context) {
        this(context, null);
    }

    public CirclePageIndicator(Context context, AttributeSet attrs) {
        this(context, attrs, R.attr.vpiCirclePageIndicatorStyle);
    }

    public CirclePageIndicator(Context context, AttributeSet attrs, int defStyle) {
        super(context, attrs, defStyle);
        if (isInEditMode()) return;

        //Load defaults from resources
        final Resources res = getResources();
        final int defaultPageColor = res.getColor(R.color.default_circle_indicator_page_color);
        final int defaultFillColor = res.getColor(R.color.default_circle_indicator_fill_color);

        ......

        //Retrieve styles attributes
        TypedArray a = context.obtainStyledAttributes(attrs, R.styleable.CirclePageIndicator, defStyle, 0);

        ......

        a.recycle();//这里记得及时释放资源

        final ViewConfiguration configuration = ViewConfiguration.get(context);
        mTouchSlop = ViewConfigurationCompat.getScaledPagingTouchSlop(configuration);
    }

    public void setOrientation(int orientation) {
        switch (orientation) {
            case HORIZONTAL:
            case VERTICAL:
                mOrientation = orientation;
                requestLayout();//设置完方向之后，就开始请求布局,会执行 measure , layout步骤
                break;

            default:
                throw new IllegalArgumentException("Orientation must be either HORIZONTAL or VERTICAL.");
        }
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);

        if (mViewPager == null) {
            return;
        }
        final int count = mViewPager.getAdapter().getCount();
        if (count == 0) {
            return;
        }

        if (mCurrentPage >= count) {
            setCurrentItem(count - 1);
            return;
        }

        /**
         * CirclePageIndicator 分为水平和竖直放置两种模式
         */

        //TODO 这里给出图，更好。。。。。。。。。
        int longSize;	/** 当前方向的Indicator宽度*/
        int longPaddingBefore;/** 当前方向的Indicator起始位置 */
        int longPaddingAfter;	/** 当前方向的Indicator结束位置 */
        int shortPaddingBefore;	/** 如果Indicator是水平方向，则取：padding Top ，竖直方向则取：padding Left */
        if (mOrientation == HORIZONTAL) {//水平方向，则由上、右、左三个方向来确定绘制范围
            longSize = getWidth();
            longPaddingBefore = getPaddingLeft();
            longPaddingAfter = getPaddingRight();
            shortPaddingBefore = getPaddingTop();
        } else {//垂直方向，则由左、上、下、来确定绘制的范围。
            longSize = getHeight();
            longPaddingBefore = getPaddingTop();
            longPaddingAfter = getPaddingBottom();
            shortPaddingBefore = getPaddingLeft();
        }


        final float threeRadius = mRadius * 3;//两相邻circle的间距
        final float shortOffset = shortPaddingBefore + mRadius;//当前方向的垂直方向的圆心坐标位置
        float longOffset = longPaddingBefore + mRadius;//当前方向的圆心位置
        if (mCentered) {
            longOffset += ((longSize - longPaddingBefore - longPaddingAfter) / 2.0f) - ((count * threeRadius) / 2.0f);
        }

        float dX;
        float dY;

        float pageFillRadius = mRadius;
        if (mPaintStroke.getStrokeWidth() > 0) {
            pageFillRadius -= mPaintStroke.getStrokeWidth() / 2.0f;
        }

        //循环的 draw circle
        for (int iLoop = 0; iLoop < count; iLoop++) {
            float drawLong = longOffset + (iLoop * threeRadius);//计算当前方向的每个circle偏移量
            if (mOrientation == HORIZONTAL) {
                dX = drawLong;
                dY = shortOffset;
            } else {
                dX = shortOffset;
                dY = drawLong;
            }

            //只绘制透明度 > 0的circle
            if (mPaintPageFill.getAlpha() > 0) {
                canvas.drawCircle(dX, dY, pageFillRadius, mPaintPageFill);
            }

            // Only paint stroke if a stroke width was non-zero
            //有pageFillRadius时才绘制
            if (pageFillRadius != mRadius) {
                canvas.drawCircle(dX, dY, mRadius, mPaintStroke);
            }
        }

        //Draw the filled circle according to the current scroll
        //根据滑动的位置画出实心的点
        float cx = (mSnap ? mSnapPage : mCurrentPage) * threeRadius;//计算实心点的目标位置
        if (!mSnap) {//不是跳跃模式，则根据当前界面的偏移量平滑地绘制
            cx += mPageOffset * threeRadius;
        }
        if (mOrientation == HORIZONTAL) {//计算实心圆的坐标
            dX = longOffset + cx;
            dY = shortOffset;
        } else {
            dX = shortOffset;
            dY = longOffset + cx;
        }
        canvas.drawCircle(dX, dY, mRadius, mPaintFill);
    }

    /**
     * 模板代码
     */
    public boolean onTouchEvent(MotionEvent ev) {
        if (super.onTouchEvent(ev)) {
            return true;
        }
        if ((mViewPager == null) || (mViewPager.getAdapter().getCount() == 0)) {//无效的ViewPager，啥也不做
            return false;
        }

        final int action = ev.getAction() & MotionEventCompat.ACTION_MASK;
        switch (action) {
            case MotionEvent.ACTION_DOWN:
                mActivePointerId = MotionEventCompat.getPointerId(ev, 0);//记录第一触摸点的ID
                mLastMotionX = ev.getX();//获取当前水平移动距离
                break;

            case MotionEvent.ACTION_MOVE: {
                final int activePointerIndex = MotionEventCompat.findPointerIndex(ev, mActivePointerId);//获取第一点的索引
                final float x = MotionEventCompat.getX(ev, activePointerIndex);//根据第一点的索引获取其X坐标
                final float deltaX = x - mLastMotionX;//计算X方向的偏移

                if (!mIsDragging) {
                    if (Math.abs(deltaX) > mTouchSlop) {//如果用户是主观的滑动屏幕，则设置标识为 mIsDragging = true
                        mIsDragging = true;
                    }
                }

                if (mIsDragging) {//如果用户拖拽了屏幕，处理ViewPager移动相应的偏移量
                    mLastMotionX = x;//重新赋值当前的X坐标，以便下次重新计算偏移量
                    if (mViewPager.isFakeDragging() || mViewPager.beginFakeDrag()) {
                        mViewPager.fakeDragBy(deltaX);
                    }
                }

                break;
            }

            case MotionEvent.ACTION_CANCEL:
            case MotionEvent.ACTION_UP:
                if (!mIsDragging) {
                    final int count = mViewPager.getAdapter().getCount();
                    final int width = getWidth();
                    final float halfWidth = width / 2f;
                    final float sixthWidth = width / 6f;

                    if ((mCurrentPage > 0) && (ev.getX() < halfWidth - sixthWidth)) {// 向后滑动，回退，以1/3屏幕宽度 为分界线。。。。。。。。。因为布局文件设置为fill_parent????????
                        if (action != MotionEvent.ACTION_CANCEL) {
                            mViewPager.setCurrentItem(mCurrentPage - 1);
                        }
                        return true;
                    } else if ((mCurrentPage < count - 1) && (ev.getX() > halfWidth + sixthWidth)) {//向前滑动
                        if (action != MotionEvent.ACTION_CANCEL) {
                            mViewPager.setCurrentItem(mCurrentPage + 1);
                        }
                        return true;
                    }
                }

                mIsDragging = false;//设置ViewPager滑动标识为false.
                mActivePointerId = INVALID_POINTER;//设置第一个触摸点的ID为invalid
                if (mViewPager.isFakeDragging()) mViewPager.endFakeDrag();//结束ViewPager的临时滑动
                break;

            case MotionEventCompat.ACTION_POINTER_DOWN: {//除最初点外的第一个外出现在屏幕上的点
                final int index = MotionEventCompat.getActionIndex(ev);
                mLastMotionX = MotionEventCompat.getX(ev, index);
                mActivePointerId = MotionEventCompat.getPointerId(ev, index);
                break;
            }

            case MotionEventCompat.ACTION_POINTER_UP://当非第一点离开屏幕时
                final int pointerIndex = MotionEventCompat.getActionIndex(ev);//获取抬起手指的索引
                final int pointerId = MotionEventCompat.getPointerId(ev, pointerIndex);//获取抬起手指的ID
                if (pointerId == mActivePointerId) {//如果之前跟踪的mActivePointerId是当前抬起的手指ID，那么就重新为mActivePointerId 赋值另一个活动中的pointerId
                    final int newPointerIndex = pointerIndex == 0 ? 1 : 0;
                    mActivePointerId = MotionEventCompat.getPointerId(ev, newPointerIndex);
                }
                mLastMotionX = MotionEventCompat.getX(ev, MotionEventCompat.findPointerIndex(ev, mActivePointerId));//获取仍活动在屏幕上pointer的X坐标值
                break;
        }

        return true;
    }


    /*
     * (non-Javadoc)
     *
     * @see android.view.View#onMeasure(int, int)
     * View在测量阶段的最终大小的设定是由setMeasuredDimension()方法决定的,也是必须要调用的方法，否则会报异常，
     * 这里就直接调用了setMeasuredDimension()方法设置值了。
     */
    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {

        if (mOrientation == HORIZONTAL) {
            setMeasuredDimension(measureLong(widthMeasureSpec), measureShort(heightMeasureSpec));
        } else {
            setMeasuredDimension(measureShort(widthMeasureSpec), measureLong(heightMeasureSpec));
        }
    }

    /**
     * 决定View的宽度
     *
     * Determines the width of this view
     *
     * @param measureSpec
     *            A measureSpec packed into an int
     * @return The width of the view, honoring constraints from measureSpec
     *
     * 测量的步骤分为获取MeasureSpec模式，获取系统建议的值、自己计算height和width（需要考虑自身的padding）
     */
    private int measureLong(int measureSpec) {
        int result;
        int specMode = MeasureSpec.getMode(measureSpec);//获取MeasureSpec模式
        int specSize = MeasureSpec.getSize(measureSpec);//获取系统建议的值

        if ((specMode == MeasureSpec.EXACTLY) || (mViewPager == null)) {//用户指定了具体的值
            //We were told how big to be
            result = specSize;
        } else {
            //UNSPECIFIED 或者 AT_MOST 模式，则根据ViewPager的page数量计算宽度
            final int count = mViewPager.getAdapter().getCount();//
            result = (int)(getPaddingLeft() + getPaddingRight()
                    + (count * 2 * mRadius) + (count - 1) * mRadius + 1);
            //Respect AT_MOST value if that was what is called for by measureSpec
            //AT_MOST 特殊处理，从系统建议值和自己计算值中取一个较小值
            if (specMode == MeasureSpec.AT_MOST) {
                result = Math.min(result, specSize);
            }
        }
        return result;
    }

    /**
     * 决定View的高度
     *
     * @param measureSpec
     *            A measureSpec packed into an int
     * @return The height of the view, honoring constraints from measureSpec
     */
    private int measureShort(int measureSpec) {
        int result;
        int specMode = MeasureSpec.getMode(measureSpec);
        int specSize = MeasureSpec.getSize(measureSpec);

        if (specMode == MeasureSpec.EXACTLY) {
            //We were told how big to be
            result = specSize;
        } else {
            //Measure the height
            result = (int)(2 * mRadius + getPaddingTop() + getPaddingBottom() + 1);
            //Respect AT_MOST value if that was what is called for by measureSpec
            if (specMode == MeasureSpec.AT_MOST) {//获取一个合适的值
                result = Math.min(result, specSize);
            }
        }
        return result;
    }
}

```

####3.1.4 参考文献  
View的绘制：  
http://blog.csdn.net/wangjinyu501/article/details/9008271  
http://blog.csdn.net/qinjuning/article/details/7110211
  	
Touch事件传递：  
http://blog.csdn.net/xiaanming/article/details/21696315
http://blog.csdn.net/wangjinyu501/article/details/22584465
