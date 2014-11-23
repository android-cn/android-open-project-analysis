
### 1. 简介

**功能介绍：**  
引用[viewpagerindicator](http://viewpagerindicator.com)项目主页的简短介绍：ViewPager适用于Android Support Library中的ViewPager和其开发的ActionBarSherlock，项目基于Patrik Åkerfeldt's ViewFlow。  
应用于各种界面的导航。  
	

### 2. 主要特点：
	* 使用简单
	* 效果好 
	* 样式多

### 3. 详细设计
	

**归类**  
	CirclePageIndicator、LinePageIndicator、UnderlinePageIndicator 这三种是极其相似的。可以归为一类。
	TabPageIndicator、IconPageIndicator 继承自HorizontalScrollView，极其相似，可以归为一类。
	TitlePagerIndicator：实现最复杂,单独介绍。
		
**结构**  
	PageIndicator，接口类，定义了不同类型的Indicator的公用的方法。
	IcsLinearLayout ：LinearLayout的扩展，支持了4.0以上的divider特性。在TabPagerIndicator中使用到。

**实现**    

**TabPageIndicator、IconPageIndicator**

继承自HorizontalScrollView,使用了IcsLinearLayout
继承自HorizontalScrollView，拥有其左右滑动的效果，以及其它以实现的操作，比如对child的measure

**CirclePageIndicator、LinePageIndicator、UnderlinePageIndicator**    
继承自View，关键字段和方法：  

**mTouchSlop**  
	“Touch slop”是指在用户触摸事件可被识别为移动手势前,移动过的那一段像素距离。Touchslop通常用来预防用户在做一些其他操作时意外地滑动，例如触摸屏幕上的元素时。

[onTouchEvent](http://hukai.me/android-training-course-in-chinese/input/gestures/scale.html)		
	对于pointer的处理是模板方法，在拖拽与缩放中有详细的讲解：

**相关理论知识**  

自定义控件的创建和Android的用户输入  
介绍这两个模块的原因:所有的可交互的自定义控件一定都会有这两个模块，另外，介绍完这两个模块后，大家从整体上就可以看明白ViewPagerIndicator的实现了。  

大家可能会担心这些知识点到哪里才能找到最权威和最系统的讲解，其实官方文档就有。已经有好心人把它翻译了：一个GitHub协作项目android-training-course-in-chinese，该项目就是对Google的官方文档进行翻译，现已有PDF版本了，大家可以去下载。（强烈建议大家直接去下载，一定会Android的整体知识架构有清晰的认识。我觉得特别有帮助，最好的学习资料肯定是官方文档，本人现在学习的主要资料）
		
Google官方中文文档翻译项目地址：[android-training-course-in-chinese](https://github.com/kesenhoo/android-training-course-in-chinese)
		
下面就简单的介绍自定义控件的整个流程:
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
    
	
用户的输入部分要注意的有以下几点（具体可以参考原文）：
		
**1. 保持对最初点的追踪**  
拖拽操作时，即使有额外的手指放置到屏幕上了，app也必须保持对最初的点（手指）的追踪。比如，想象在拖拽图片时，用户放置了第二根手指在屏幕上，并且抬起了第一根手指。如果你的app只是单独地追踪每个点，它会把第二个点当做默认的点，并且把图片移到该点的位置。
	
**2. 区分原始点及之后的任意触摸点**   
为了防止这种情况发生，你的app需要区分初始点以及之后任意的触摸点。要做到这一点，它需要追踪处理多触摸手势中提到过的ACTION_POINTER_DOWN、 ACTION_POINTER_UP事件。每当第二根手指按下或拿起时，ACTION_POINTER_DOWN、ACTION_POINTER_UP事件就会传递给onTouchEvent())回调函数。
	
**3. 确保操作中的点的ID(the active pointer ID)不会引用已经不在触摸屏上的触摸点**  
当ACTION_POINTER_UP事件发生时，示例程序会移除对该点的索引值的引用，确保操作中的点的ID(the active pointer ID)不会引用已经不在触摸屏上的触摸点。这种情况下，app会选择另一个触摸点来作为操作中(active)的点，并保存它当前的x、y值。由于在ACTION_MOVE事件时，这个保存的位置会被用来计算屏幕上的对象将要移动的距离，所以app会始终根据正确的触摸点来计算移动的距离。

ViewPagerIndicator中的onTouchEvent中的代码也就是官方文档的模板代码，就是为了确保以上几点，拿到可用，确信的点然后处理ViewPager相应的偏移和滑动

**View绘制机制**  
我们扩展一下View的绘制机制，因为在viewpagerindicator的源码中会有相关的涉及  
参考文献：http://developer.android.com/guide/topics/ui/how-android-draws.html

当Activity接收到焦点的时候，它会被请求绘制布局。Android framework将会处理绘制的流程，但Activity必须提供View层级的根节点。绘制是从根节点开始的，需要measure和draw布局树。绘制会遍历和渲染每一个与无效区域相交的view。相反，每一个ViewGroup负责绘制它所有的childrenview，而View会负责绘制自身。树的遍历是有序的，parent view要先于child View被绘制，

绘制布局有两步：measure和layout  
measure过程的实现在measure(int,int)方法中，而且从上到下的有序绘制view。在递归的过程中，每一个视图（View）将尺寸规格向下传递给View，在measure过程的最后，每个视图存储了它的尺寸。
layout过程从layout(int, int, int, int)方法开始，也是自上而下遍历。在这个过程中，每个parent view根据measure过程计算出来
的尺寸为所有的child view指定位置。  

注意：该框架不会绘制无效区域之外的部分,同时会注意绘制视图的背景。你可以使用 invalidate()去强制一个view重绘。  

当一个View的measure()执行完的时候，它自己以及所有的孩子节点的getMeasuredWidth()和getMeasuredHeight()方法的值就必须被设置了。一个视图的测量宽度和测量高度值必须在父视图约束范围之内，这可以保证在measure的最后,所有的父母都接受所有孩子的测量。
一个父视图，可以在其child view上多次的调用measure()方法。比如，父视图可以先根据未指明的dimension调用measure方法去测量每一个
child view的大小，如果所有child的未约束尺寸太大或者太小的时候，则会使用一个确切的大小，然后在每一个childview上再次调用measure方法去测量每一个view的大小。（也就是说，如果children对于获取到的大小不满意的时候，父视图会介入并设置测量规则进行第二次measure）

measure过程使用了两个类来传递尺寸：  
一个是ViewGroup.LayoutParams类（View自身的布局参数）  
一个是MeasureSpecs类（父视图对子视图的测量要求）

ViewGroup.LayoutParams类被子视图用于告诉他们的父视图他们应该怎样被测量和放置（就是子视图自身的布局参数）。一个是基本的LayoutParams，只是用来描述视图的高度和宽度。它的尺寸可以有三种表示方法：  
1、具体数值   
2、MATCH_PARENT 表示子视图希望和父视图一样大(不含padding)   
3、WRAP_CONTENT 表示视图为正好能包裹其内容大小(包含padding)    

还有一些ViewGroup.LayoutParams的子类，例如RelativeLayout有相应的LayoutParams的子类,拥有设置子视图水平和垂直的能力。

MeasureSpecs用于从上到下传递父视图对子视图测量需求。
MeasureSpec有三种模式:      

UNSPECIFIED  
父视图可以为子视图设置它所期望的大小。比如一个LinearLayout可以在它的子view上调用measure()方法去测量一个高设置为UNSPECIFIED模式，宽为240pixels的view大小。    

EXACTLY  
父视图决定子视图的确切大小，子视图必须使用该大小，并确保它所有的子视图可以适应在该尺寸的范围内；相对应属性的是MATCH_PARENT   

AT_MOST  
父视图为子视图指定一个最大值。子视图必须确保它自己的所有子视图在该尺寸范围内，相应的属性为WRAP_CONTENT

  
**源码分析 CirclePageIndicator **  

```
public class CirclePageIndicator extends View implements PageIndicator {
    private static final int INVALID_POINTER = -1;

    /**
     * 当前界面的索引
     */
    private int mCurrentPage;

    /**
     * 当前界面的索引，和mCurrentPage值一样
     */
    private int mSnapPage;
    /**
     * ViewPager的水平偏移量
     */
    private float mPageOffset;
    /**
     * ViewPager的滑动状态
     */
    private int mScrollState;
  
    /**
     * Indicator的模式：水平、竖直
     */
    private int mOrientation;
    private boolean mCentered;
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
    /**
     * 每一次onTouch事件产生时水平位置的最后偏移量
     */
    private float mLastMotionX = -1;

    /**
     * 当前处于活动中pointer的ID
     */
    private int mActivePointerId = INVALID_POINTER;

    /**
     * 用户是否主观的滑动屏幕的标识
     */
     private boolean mIsDragging;

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

    public void setOrientation(int orientation) {//设置完方向之后，就开始请求布局
        switch (orientation) {
            case HORIZONTAL:
            case VERTICAL:
                mOrientation = orientation;
                requestLayout();//会执行 measure , layout，draw步骤
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
     *
     * MeasureSpc类封装了父View传递给子View的布局(layout)要求。每个MeasureSpc实例代表宽度或者高度(只能是其一)要求，它有三种模式：
     *  ①、UNSPECIFIED(未指定)，父元素不对子元素施加任何束缚，子元素可以得到任意想要的大小；
     *  ②、EXACTLY(完全)，父元素决定自元素的确切大小，子元素将被限定在给定的边界里而忽略它本身大小；相对应的是 FILL_PARENT
     *  ③、AT_MOST(至多)，子元素至多达到指定大小的值。相对应的是 WRAP_CONTENT
     *
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

  
  几篇相关的文章链接：  
  
  View的绘制：  
  	http://blog.csdn.net/wangjinyu501/article/details/9008271  
  	http://blog.csdn.net/qinjuning/article/details/7110211
  	
  Touch事件传递：  
  	http://blog.csdn.net/xiaanming/article/details/21696315
  	http://blog.csdn.net/wangjinyu501/article/details/22584465
  	

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


