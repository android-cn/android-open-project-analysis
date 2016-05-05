Sticky-List-Headers源码解析    
----------------
>Author：Hyman Lee  
Email：hyman.dev@gmail.com  
Github：[MrBigBang](https://github.com/MrBigBang)   
> 项目地址：[StickyListHeaders](https://github.com/emilsjolander/StickyListHeaders)，Demo 地址：[sticky-list-headers-demo](https://github.com/aosp-exchange-group/android-open-project-demo/tree/master/sticky-list-headers-demo)

###1.功能介绍  
这个开源库可以实现的UI效果：粘性头部列表以及可隐藏展开列表。效果很赞，“开发者头条”这个app上有用到。  
主要特点：  
（1）使用静态代理模式，对adapter进行封装，外界使用过程中定义的adapter只需要按照常规实现即可，但必须实现StickyListHeadersAdapter接口，可以选择性实现SectionIndexer接口。  
（2）WrapperView继承自ViewGroup，其内部包含一个item（View）、一个header（View）和一个divider（drawable），但是header和divider是不能共存的有一个必为null。并实现了对item、header、divider的复用。  
###2.总体设计
StickyListHeaders这个开源库结构还是比较简单的，总体结构图如下：  
![UML](./img/uml.png)  
（1）主体为<code>**StickyListHeadersListView**</code>这个类，这个类继承值FrameLayout，其中有两个布局，一个就是ListView的子类<code>WrapperViewList</code>，另一个就是所谓的粘性Header（_下面统称为StickyHeader，防止和WrapperView中的header混淆_）。  
&emsp;&emsp;这个类内部定义了三个接口，供外界实现交互： 
<font color=red>   
&emsp;&emsp;- OnHeaderClickListener  
&emsp;&emsp;- OnStickyHeaderOffsetChangedListener  
&emsp;&emsp;- OnStickyHeaderChangedListener  
</font>
&emsp;&emsp;主要负责：测量StickyHeader（_有个<code>measureHeader()</code>方法_），在回调函数onDispatchDrawOccurred中绘制StickyHeader，控制内部StickyHeader的显示及变化（_通过<code>swapHeader()</code>方法_），以及控制点击事件分发到StickyHeader还是WrapperViewList。  
（2）<code>**WrapperViewList**</code>继承自ListView，在绘制自身的时候会根据mTopClippingLength的值是否为0将canvas进行相应的裁剪，留出空间给StickyListHeadersListView绘制StickyHeader以及paddingTop，同时这样子处理可以保证StickyHeader绘制在ListView的上面。  
&emsp;&emsp;主要负责确定点击时的选中区域，通过<code>performItemClick</code>方法确定点击背景大小为WrapperView中的item大小，并通过反射获取点击时出现点击背景效果的区域Rect，在<code>dispatchDraw</code>方法中调用<code>positionSelectorRect</code>确定该Rect的top大小。  
（3）<code>**AdapterWrapper**</code>继承自BaseAdapter，实现了<code>StickyListHeadersAdapter</code>接口，其中有个<code>StickyListHeadersAdapter</code>接口的实例（_其实就是外界定义的adapter，后面就统称为mDelegate_），该mDelegate负责代理实现<code>AdapterWrapper</code>的功能。  
&emsp;&emsp;主要负责通过getView方法将从mDelegate中的getView方法获得的item和getHeaderView方法获得的header组装成一个WrapperView，但是其中会通过方法<code>previousPositionHasSameHeader(position)</code>判断当前位置的WrapperView是否是一个新Section的第一个元素。如果是第一个就会通过<code>configureHeader(wv, position)</code>方法获取一个header，这个方法实现中会对header进行复用；如果不是第一个，就调用<code>recycleHeaderIfExists(wv)</code>方法将这个header缓存在mHeaderCache（List&lt;View&gt;）中。  
（4）<code>**DistinctMultiHashMap**</code>是一个建立一对多关系的集合（_基于LinkedHashMap_）。<code>**DualHashMap**</code>是一个实现可以通过key获取value也可以通过value获取key的数据结构（_基于两个HashMap_）。  
（5）<code>**ExpandableStickyListHeadersAdapter**</code>和<code>**ExpandableStickyListHeadersListView**</code>实现了点击列表某个Section第一个元素对应的header时会进行展开或收起的功能，主要是使用（4）中的两个数据结构将headerId与对应Section所有WrapperView建立一对多的存储关系。在外界调用的时候设置OnHeaderClickListener根据点击的header当前状态进行展开或收起，其中ExpandableStickyListHeadersListView中定义了一个接口<code>IAnimationExecutor</code>，让调用者自行定义展开和收起的动画效果。  
###3.杂谈  
#####（1）巧妙处理点：  
&emsp;&emsp;其中源码中对<code>StickyListHeadersListView</code>显示出来的divider进行了巧妙的处理，将<code>WrapperViewList</code>的divider设置为null。  
<pre><code> @TargetApi(Build.VERSION_CODES.HONEYCOMB)
    public StickyListHeadersListView(Context context, AttributeSet attrs, int defStyle) {
        super(context, attrs, defStyle);

        mTouchSlop = ViewConfiguration.get(getContext()).getScaledTouchSlop();

        // Initialize the wrapped list
        mList = new WrapperViewList(context);

        // null out divider, dividers are handled by adapter so they look good with headers
        mDivider = mList.getDivider();
        mDividerHeight = mList.getDividerHeight();
        mList.setDivider(null);
        mList.setDividerHeight(0);
        ...
    }</code></pre>
&emsp;&emsp;并将其取出来设置给mAdapter（<code>AdapterWrapper</code>）:  
<pre><code>public void setAdapter(StickyListHeadersAdapter adapter) {
        ...
        mAdapter.setDivider(mDivider, mDividerHeight);

        mList.setAdapter(mAdapter);
        clearHeader();
    }</code></pre>  
&emsp;&emsp;在<code>AdapterWrapper</code>中，通过调用WrapperView的<code>update(View item, View header, Drawable divider, int dividerHeight)</code>方法将divider以及dividerHeight设置给WrapperView。 
<pre><code>@Override
	public WrapperView getView(int position, View convertView, ViewGroup parent) {
		...
		wv.update(item, header, mDivider, mDividerHeight);
		return wv;
	}</code></pre>  
&emsp;&emsp;更新完divider和dividerHeight后会调用<code>invalidate()</code>方法进行重新绘制，在下面代码中会完成divider的绘制。  
<pre><code>@Override
	protected void dispatchDraw(Canvas canvas) {
		super.dispatchDraw(canvas);
		if (mHeader == null && mDivider != null&&mItem.getVisibility()!=View.GONE) {
			// Drawable.setBounds() does not seem to work pre-honeycomb. So have
			// to do this instead
			if (Build.VERSION.SDK_INT < Build.VERSION_CODES.HONEYCOMB) {
				canvas.clipRect(0, 0, getWidth(), mDividerHeight);
			}
			mDivider.draw(canvas);
		}
	}</code></pre>  
&emsp;&emsp;divider之所以给<code>WrapperView</code>自己绘制并取消<code>ListView</code>本身的divider，这样做就可以方便控制header和divider只显示其一。  
#####（2）通过一些自定义属性可以改变展现效果：  
&emsp;&emsp;- hasStickyHeaders，是否显示粘性header效果。  
&emsp;&emsp;- isDrawingListUnderStickyHeader，表示ListView的绘制区域是从屏幕可见区域的top开始还是从StickyHeader的bottom处开始。