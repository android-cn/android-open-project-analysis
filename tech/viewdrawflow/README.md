公共技术点之 View 绘制流程
----------------
> 本文为 [Android 开源项目源码解析](http://a.codekk.com) 公共技术点中的 View 绘制流程 部分  
 分析者：[lightSky](https://github.com/lightSky)  

#### View 绘制机制  
##### 1. View 树的绘图流程
当 Activity 接收到焦点的时候，它会被请求绘制布局,该请求由 Android framework 处理.绘制是从根节点开始，对布局树进行 measure 和 draw。整个 View 树的绘图流程在`ViewRoot.java`类的`performTraversals()`函数展开，该函数所做 的工作可简单概况为是否需要重新计算视图大小(measure)、是否需要重新安置视图的位置(layout)、以及是否需要重绘(draw)，流程图如下：  
![viewdrawflow img](image/view_mechanism_flow.png)  

**View 绘制流程函数调用链**  
![view_draw_method_chain img](image/view_draw_method_chain.png)  
图片来自 https://plus.google.com/+ArpitMathur/posts/cT1EuBbxEgN  
需要说明的是，用户主动调用 request，只会出发 measure 和 layout 过程，而不会执行 draw 过程

##### 2. 概念 

**measure 和 layout**  

从整体上来看 Measure 和 Layout 两个步骤的执行：
![MeasureLayout img](image/measure_layout.png)  
树的遍历是有序的，由父视图到子视图，每一个 ViewGroup 负责测绘它所有的子视图，而最底层的 View 会负责测绘自身。  

**具体分析**  
measure 过程由`measure(int, int)`方法发起，从上到下有序的测量 View，在 measure 过程的最后，每个视图存储了自己的尺寸大小和测量规格。
layout 过程由`layout(int, int, int, int)`方法发起，也是自上而下进行遍历。在该过程中，每个父视图会根据 measure 过程得到的尺寸来摆放自己的子视图。  

 measure 过程会为一个 View 及所有子节点的 mMeasuredWidth 和 mMeasuredHeight 变量赋值，该值可以通过 `getMeasuredWidth()`和`getMeasuredHeight()`方法获得。而且这两个值必须在父视图约束范围之内，这样才可以保证所有的父视图都接收所有子视图的测量。如果子视图对于 Measure 得到的大小不满意的时候，父视图会介入并设置测量规则进行第二次 measure。比如，父视图可以先根据未给定的 dimension 去测量每一个子视图，如果最终子视图的未约束尺寸太大或者太小的时候，父视图就会使用一个确切的大小再次对子视图进行 measure。 

**measure 过程传递尺寸的两个类**  
- ViewGroup.LayoutParams （View 自身的布局参数）  
- MeasureSpecs 类（父视图对子视图的测量要求）

ViewGroup.LayoutParams  
这个类我们很常见，就是用来指定视图的高度和宽度等参数。对于每个视图的 height 和 width，你有以下选择：  
- 具体值   
- MATCH_PARENT 表示子视图希望和父视图一样大(不包含 padding 值)   
- WRAP_CONTENT 表示视图为正好能包裹其内容大小(包含 padding 值)    

ViewGroup 的子类有其对应的 ViewGroup.LayoutParams 的子类。比如 RelativeLayout 拥有的 ViewGroup.LayoutParams 的子类 RelativeLayoutParams。  
有时我们需要使用 view.getLayoutParams() 方法获取一个视图 LayoutParams，然后进行强转，但由于不知道其具体类型，可能会导致强转错误。其实该方法得到的就是其所在父视图类型的 LayoutParams，比如 View 的父控件为 RelativeLayout，那么得到的 LayoutParams 类型就为 RelativeLayoutParams。  

MeasureSpecs  
测量规格，包含测量要求和尺寸的信息，有三种模式:    

- UNSPECIFIED  
父视图不对子视图有任何约束，它可以达到所期望的任意尺寸。比如 ListView、ScrollView，一般自定义 View 中用不到，

- EXACTLY  
父视图为子视图指定一个确切的尺寸，而且无论子视图期望多大，它都必须在该指定大小的边界内，对应的属性为 match_parent 或具体值，比如 100dp，父控件可以通过`MeasureSpec.getSize(measureSpec)`直接得到子控件的尺寸。

- AT_MOST  
父视图为子视图指定一个最大尺寸。子视图必须确保它自己所有子视图可以适应在该尺寸范围内，对应的属性为 wrap_content，这种模式下，父控件无法确定子 View 的尺寸，只能由子控件自己根据需求去计算自己的尺寸，这种模式就是我们自定义视图需要实现测量逻辑的情况。
 
##### 3. measure 核心方法  
- measure(int widthMeasureSpec, int heightMeasureSpec)  
该方法定义在`View.java`类中，为 final 类型，不可被复写，但 measure 调用链最终会回调 View/ViewGroup 对象的 `onMeasure()`方法，因此自定义视图时，只需要复写 `onMeasure()` 方法即可。

- onMeasure(int widthMeasureSpec, int heightMeasureSpec)  
该方法就是我们自定义视图中实现测量逻辑的方法，该方法的参数是父视图对子视图的 width 和 height 的测量要求。在我们自身的自定义视图中，要做的就是根据该 widthMeasureSpec 和 heightMeasureSpec 计算视图的 width 和 height，不同的模式处理方式不同。

- setMeasuredDimension()  
测量阶段终极方法，在 `onMeasure(int widthMeasureSpec, int heightMeasureSpec)` 方法中调用，将计算得到的尺寸，传递给该方法，测量阶段即结束。该方法也是必须要调用的方法，否则会报异常。在我们在自定义视图的时候，不需要关心系统复杂的 Measure 过程的，只需调用`setMeasuredDimension()`设置根据 MeasureSpec 计算得到的尺寸即可，你可以参考 [ViewPagerIndicator](http://a.codekk.com/detail/Android/lightSky/ViewPagerindicator%20%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90) 的 onMeasure 方法。  

下面我们取 ViewGroup 的 `measureChildren（int widthMeasureSpec, int heightMeasureSpec)` 方法对复合 View 的 Measure 流程做一个分析：
MeasureChild 的方法调用流程图：  
![MeasureLayout img](image/measurechildflow.png)    

**源码分析**
```java
    /**
     * 请求所有子 View 去 measure 自己，要考虑的部分有对子 View 的测绘要求 MeasureSpec 以及其自身的 padding
     * 这里跳过所有为 GONE 状态的子 View，最繁重的工作是在 getChildMeasureSpec 方法中处理的
     *
     * @param widthMeasureSpec  对该 View 的 width 测绘要求
     * @param heightMeasureSpec 对该 View 的 height 测绘要求
     */
    protected void measureChildren(int widthMeasureSpec, int heightMeasureSpec) {
        final int size = mChildrenCount;
        final View[] children = mChildren;
        for (int i = 0; i < size; ++i) {
            final View child = children[i];
            if ((child.mViewFlags & VISIBILITY_MASK) != GONE) {
                measureChild(child, widthMeasureSpec, heightMeasureSpec);
            }
        }
    }
    
    protected void measureChild(View child, int parentWidthMeasureSpec,
            int parentHeightMeasureSpec) {
        final LayoutParams lp = child.getLayoutParams();//获取 Child 的 LayoutParams

        final int childWidthMeasureSpec = getChildMeasureSpec(parentWidthMeasureSpec,// 获取 ChildView 的 widthMeasureSpec
                mPaddingLeft + mPaddingRight, lp.width);
        final int childHeightMeasureSpec = getChildMeasureSpec(parentHeightMeasureSpec,// 获取 ChildView 的 heightMeasureSpec
                mPaddingTop + mPaddingBottom, lp.height);

        child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
    }
   
   /** 
     * 该方法是 measureChildren 中最繁重的部分，为每一个 ChildView 计算出自己的 MeasureSpec。
     * 目标是将 ChildView 的 MeasureSpec 和 LayoutParams 结合起来去得到一个最合适的结果。
     *
     * @param spec 对该 View 的测绘要求
     * @param padding 当前 View 在当前唯独上的 paddingand，也有可能含有 margins
     *
     * @param childDimension 在当前维度上（height 或 width）的具体指
     * @return 子视图的 MeasureSpec 
     */
    public static int getChildMeasureSpec(int spec, int padding, int childDimension) {
    		
    		.........
	  
	    // 根据获取到的子视图的测量要求和大小创建子视图的 MeasureSpec
	    return MeasureSpec.makeMeasureSpec(resultSize, resultMode);  
    }
    
   /**
     *
     * 用于获取 View 最终的大小，父视图提供了宽、高的约束信息
     * 一个 View 的真正的测量工作是在 onMeasure(int, int) 中，由该方法调用。
     * 因此，只有 onMeasure(int, int) 可以而且必须被子类复写
     *
     * @param widthMeasureSpec 在水平方向上，父视图指定的的 Measure 要求
     * @param heightMeasureSpec 在竖直方向上，控件上父视图指定的 Measure 要求
     *
     */
    public final void measure(int widthMeasureSpec, int heightMeasureSpec) {
      ...
      
      onMeasure(widthMeasureSpec, heightMeasureSpec);
      
      ...
    }
    
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        setMeasuredDimension(getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec),
                getDefaultSize(getSuggestedMinimumHeight(), heightMeasureSpec));
    }
    
```


##### 4. layout 相关概念及核心方法  
首先要明确的是，子视图的具体位置都是相对于父视图而言的。View 的 onLayout 方法为空实现，而 ViewGroup 的 onLayout 为 abstract 的，因此，如果自定义的 View 要继承 ViewGroup 时，必须实现 onLayout 函数。  

在 layout 过程中，子视图会调用`getMeasuredWidth()`和`getMeasuredHeight()`方法获取到 measure 过程得到的 mMeasuredWidth 和 mMeasuredHeight，作为自己的 width 和 height。然后调用每一个子视图的`layout(l, t, r, b)`函数，来确定每个子视图在父视图中的位置。    

** LinearLayout 的 onLayout 源码分析**
```java
  @Override
    protected void onLayout(boolean changed, int l, int t, int r, int b) {
        if (mOrientation == VERTICAL) {
            layoutVertical(l, t, r, b);
        } else {
            layoutHorizontal(l, t, r, b);
        }
    }
    
    /**
     * 遍历所有的子 View，为其设置相对父视图的坐标
     */
    void layoutVertical(int left, int top, int right, int bottom) {
	for (int i = 0; i < count; i++) {
	            final View child = getVirtualChildAt(i);
	            if (child == null) {
	                childTop += measureNullChild(i);
	            } else if (child.getVisibility() != GONE) {//不需要立即展示的 View 设置为 GONE 可加快绘制
	                final int childWidth = child.getMeasuredWidth();//measure 过程确定的 Width
	                final int childHeight = child.getMeasuredHeight();//measure 过程确定的 height
	                
	                ...确定 childLeft、childTop 的值
	
	                setChildFrame(child, childLeft, childTop + getLocationOffset(child),
	                        childWidth, childHeight);
	            }
	        }
	}
	
    private void setChildFrame(View child, int left, int top, int width, int height) {        
        child.layout(left, top, left + width, top + height);
    }	
      
    View.java
    public void layout(int l, int t, int r, int b) {
    	...
    	setFrame(l, t, r, b)
    }
    
    /**
     * 为该子 View 设置相对其父视图上的坐标
     */
     protected boolean setFrame(int left, int top, int right, int bottom) {
     	...
     }
```
##### 5. 绘制流程相关概念及核心方法    
先来看下与 draw 过程相关的函数：  

- View.draw(Canvas canvas)：
由于 ViewGroup 并没有复写此方法，因此，所有的视图最终都是调用 View 的 draw 方法进行绘制的。在自定义的视图中，也不应该复写该方法，而是复写 `onDraw(Canvas)` 方法进行绘制，如果自定义的视图确实要复写该方法，那么请先调用 `super.draw(canvas)`完成系统的绘制，然后再进行自定义的绘制。

- View.onDraw()：  
View 的`onDraw（Canvas）`默认是空实现，自定义绘制过程需要复写的方法，绘制自身的内容。  

- dispatchDraw()
发起对子视图的绘制。View 中默认是空实现，ViewGroup 复写了`dispatchDraw()`来对其子视图进行绘制。该方法我们不用去管，自定义的 ViewGroup 不应该对`dispatchDraw()`进行复写。


绘制流程图  
![MeasureLayout img](image/draw_method_flow.png)    

**- View.draw(Canvas) 源码分析**  
```java
 /**
     * Manually render this view (and all of its children) to the given Canvas.
     * The view must have already done a full layout before this function is
     * called.  When implementing a view, implement
     * {@link #onDraw(android.graphics.Canvas)} instead of overriding this method.
     * If you do need to override this method, call the superclass version.
     *
     * @param canvas The Canvas to which the View is rendered.  
     *
     * 根据给定的 Canvas 自动渲染 View（包括其所有子 View）。在调用该方法之前必须要完成 layout。当你自定义 view 的时候，
     * 应该去是实现 onDraw(Canvas) 方法，而不是 draw(canvas) 方法。如果你确实需要复写该方法，请记得先调用父类的方法。
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
        
         // skip step 2 & 5 if possible (common case)
        final int viewFlags = mViewFlags;
        if (!verticalEdges && !horizontalEdges) {
            // Step 3, draw the content
            if (!dirtyOpaque) onDraw(canvas);

            // Step 4, draw the children
            dispatchDraw(canvas);

            // Step 6, draw decorations (scrollbars)
            onDrawScrollBars(canvas);

            if (mOverlay != null && !mOverlay.isEmpty()) {
                mOverlay.getOverlayView().dispatchDraw(canvas);
            }

            // we're done...
            return;
        }
        
        // Step 2, save the canvas' layers
        ...
        
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

由上面的处理过程，我们也可以得出一些优化的小技巧：当不需要绘制 Layer 的时候第二步和第五步会跳过。**因此在绘制的时候，能省的 layer 尽可省，可以提高绘制效率**  

**ViewGroup.dispatchDraw() 源码分析**

```java
dispatchDraw(Canvas canvas){

...

 if ((flags & FLAG_RUN_ANIMATION) != 0 && canAnimate()) {//处理 ChildView 的动画
 	final boolean buildCache = !isHardwareAccelerated();
            for (int i = 0; i < childrenCount; i++) {
                final View child = children[i];
                if ((child.mViewFlags & VISIBILITY_MASK) == VISIBLE) {//只绘制 Visible 状态的布局，因此可以通过延时加载来提高效率
                    final LayoutParams params = child.getLayoutParams();
                    attachLayoutAnimationParameters(child, params, i, childrenCount);// 添加布局变化的动画
                    bindLayoutAnimation(child);//为 Child 绑定动画
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

	controller.start();// 启动 View 的动画
}

 // 绘制 ChildView
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

protected boolean drawChild(Canvas canvas, View child, long drawingTime) {
        return child.draw(canvas, this, drawingTime);
}

/**
     * This method is called by ViewGroup.drawChild() to have each child view draw itself.
     * This draw() method is an implementation detail and is not intended to be overridden or
     * to be called from anywhere else other than ViewGroup.drawChild().
     */
    boolean draw(Canvas canvas, ViewGroup parent, long drawingTime) {
        ...
    }

```
- drawChild(canvas, this, drawingTime)  
直接调用了 View 的`child.draw(canvas, this,drawingTime)`方法，文档中也说明了，除了被`ViewGroup.drawChild()`方法外，你不应该在其它任何地方去复写或调用该方法，它属于 ViewGroup。而`View.draw(Canvas) `方法是我们自定义控件中可以复写的方法，具体可以参考上述对`view.draw(Canvas)`的说明。从参数中可以看到，`child.draw(canvas, this, drawingTime)` 肯定是处理了和父视图相关的逻辑，但 View 的最终绘制，还是 `View.draw(Canvas)`方法。

- invalidate()  
请求重绘 View 树，即 draw 过程，假如视图发生大小没有变化就不会调用`layout()`过程，并且只绘制那些调用了`invalidate()`方法的 View。

- requestLayout()  
当布局变化的时候，比如方向变化，尺寸的变化，会调用该方法，在自定义的视图中，如果某些情况下希望重新测量尺寸大小，应该手动去调用该方法，它会触发`measure()`和`layout()`过程，但不会进行 draw。  

参考资料  
[how-android-draws](http://developer.android.com/guide/topics/ui/how-android-draws.html)  
http://blog.csdn.net/wangjinyu501/article/details/9008271  
http://blog.csdn.net/qinjuning/article/details/7110211  
http://blog.csdn.net/qinjuning/article/details/8074262
