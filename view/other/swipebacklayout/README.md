Picasso 源码分析
====================================
> 本文为 [Android 开源项目源码解析](http://a.codekk.com) 中 SwipeBackLayout 部分  
> 项目地址：[SwipeBackLayout](https://github.com/ikew0ng/SwipeBackLayout)，分析的版本：，Demo 地址：
> 分析者：[Neocomb](https://github.com/Neocomb)，分析状态：进行中。校对者：，校对状态：未开始

###1. 功能介绍
####1.1 SwipeBackLayout
	这是一个让你的Activity具有滑动返回手势的库
####1.2 基本使用
	1. 确保当前Activity所使用的主题添加了 <item name="android:windowIsTranslucent">true</item> 来使背景透明
	2. 使Activity继承SwipeBackActivity 

###2. 总体设计
![类图](image/SwipeBackLayout.png)

###3. 流程图
#### 3.1 使Activity透明 ####
![时序图](image/Activity.png)

	1. 在SwipeBackActivity的onCreate()时，初始化SwipeBackActivityHelper，并调用其onActivityCreate()方法
	2. 在onActivityCreate()中，将Activity的背景设置为透明色，并加载SwipeBackLayout的布局，辅助实现滑动的效果
	3. 在SwipeBackActivity的onPostCreate()时，在decorView和contentView之间添加SwipeBackLayout
        public void attachToActivity(Activity activity) {
        mActivity = activity;
        TypedArray a = activity.getTheme().obtainStyledAttributes(new int[]{
                android.R.attr.windowBackground
        });
        int background = a.getResourceId(0, 0);
        a.recycle();

        ViewGroup decor = (ViewGroup) activity.getWindow().getDecorView();
        ViewGroup decorChild = (ViewGroup) decor.getChildAt(0);
        decorChild.setBackgroundResource(background);
        decor.removeView(decorChild);
        addView(decorChild);
        setContentView(decorChild);
        decor.addView(this);
    }

#### 3.2 处理触摸事件####
>SwipeBackLayout继承于FrameLayout，然后搭配专门处理拖拽事件的ViewDragHelper，完成整个Activity的手势效果。
>在较高版本的Support-v4包下，有提供ViewDragHelper，但是作者并没有使用，而是自定义了一个ViewDragHelper，功能类似。
>使用ViewDragHelper主要是实现ViewDragHelper.Callback的一个回调，同时将touch事件交递给ViewDragHelper处理。

SwipeBackLayout.java:

	- View的事件处理
	@Override
    public boolean onInterceptTouchEvent(MotionEvent event) {
        if (!mEnable) {
            return false;
        }
        try {
            return mDragHelper.shouldInterceptTouchEvent(event);
        } catch (ArrayIndexOutOfBoundsException e) {
            // FIXME: handle exception
            // issues #9
            return false;
        }
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        if (!mEnable) {
            return false;
        }
        mDragHelper.processTouchEvent(event);
        return true;
    }

	- Callback实现
		//判断当前View是否可以拖拽并且是否是可拖拽的边缘
		@Override
		public boolean tryCaptureView(View view, int pointId) {
            boolean ret = mDragHelper.isEdgeTouched(mEdgeFlag, i);
            if (ret) {
                if (mDragHelper.isEdgeTouched(EDGE_LEFT, i)) {
                    mTrackingEdge = EDGE_LEFT;
				...
        }

		//返回指定View在横向上能滑动的最大距离，大于0即可滑动
        @Override
        public int getViewHorizontalDragRange(View child) {
            return mEdgeFlag & (EDGE_LEFT | EDGE_RIGHT);
        }

		//返回指定View在纵向上能滑动的最大距离，大于0即可滑动
        @Override
        public int getViewVerticalDragRange(View child) {
            return mEdgeFlag & EDGE_BOTTOM;
        }

		//当Activity的位置发生改变时调用，计算滑动完成的百分比，当超出阀值时，finish掉Activity
        @Override
        public void onViewPositionChanged(View changedView, int left, int top, int dx, int dy) {
            super.onViewPositionChanged(changedView, left, top, dx, dy);
            if ((mTrackingEdge & EDGE_LEFT) != 0) {
                mScrollPercent = Math.abs((float) left
                        / (mContentView.getWidth() + mShadowLeft.getIntrinsicWidth()));
           ...

            if (mScrollPercent >= 1) {
                if (!mActivity.isFinishing())
                    mActivity.finish();
            }
        }

		//当Activity拖拽被释放时调用，判断当Activity的拖拽结束时的位置，使Activity复原或者移出屏幕
		//xvel,yvel分别为松开时，Activity在x和y方向上的速度
        @Override
        public void onViewReleased(View releasedChild, float xvel, float yvel) {
            final int childWidth = releasedChild.getWidth();
            final int childHeight = releasedChild.getHeight();

            int left = 0, top = 0;
            if ((mTrackingEdge & EDGE_LEFT) != 0) {
                left = xvel > 0 || xvel == 0 && mScrollPercent > mScrollThreshold ? childWidth
                        + mShadowLeft.getIntrinsicWidth() + OVERSCROLL_DISTANCE : 0;
     		...

            mDragHelper.settleCapturedViewAt(left, top);
            invalidate();
        }

		//计算Activity在水平方向上滑动的偏移，left为上一次的偏移，dx为本次偏移量
        @Override
        public int clampViewPositionHorizontal(View child, int left, int dx) { ...  }

        @Override
        public int clampViewPositionVertical(View child, int top, int dy) {
            int ret = 0;
            if ((mTrackingEdge & EDGE_BOTTOM) != 0) {
                ret = Math.min(0, Math.max(top, -child.getHeight()));
            }
            return ret;
        }

		//通知Activity的状态发生变化
        @Override
        public void onViewDragStateChanged(int state) {
            super.onViewDragStateChanged(state);
            if (mListeners != null && !mListeners.isEmpty()) {
                for (SwipeListener listener : mListeners) {
                    listener.onScrollStateChange(state, mScrollPercent);
                }
            }
        }


#### 参考 ####
[Touch事件传递](http://codekk.com/blogs/detail/54cfab086c4761e5001b253e) <br>
[http://blog.csdn.net/shaw1994/article/details/44536667](http://blog.csdn.net/shaw1994/article/details/44536667)