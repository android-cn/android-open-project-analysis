FlyRefresh 源码解析
====================================

> 本文为 [FlyRefresh](https://github.com/android-cn/android-open-project-analysis) 中 FlyRefresh 部分  
> 项目地址：[FlyRefresh](https://github.com/race604/FlyRefresh)，分析的版本：[5299e8b](https://github.com/race604/FlyRefresh/commit/5299e8b969aab63fc1fde0a5423b19a61cded53b "Commit id is 5299e8b969aab63fc1fde0a5423b19a61cded53b]")，Demo 地址：[fly-refresh-demo](https://github.com/aosp-exchange-group/android-open-project-demo/tree/master/fly-refresh-demo)  

> 分析者：[skyacer](http://github.com/skyacer)，分析状态：未完成，校对者：[Trinea](https://github.com/trinea)，校对状态：未开始  

##1. 功能介绍  

FlyRefresh 是一个非常漂亮的下拉刷新的框架，下拉后会有纸飞机在顶部转一圈然后如果有新的item则会增加一条。添加方法只需要在你的布局中添加 FlyRefreshLayout 即可，使用的RecyclerView实现。

###1.1 **完成时间**  

- `2015-07-27`更新 



###1.2 **集成指南**  


在 gradle 中

``` xml

dependencies {

    compile 'com.race604.flyrefresh:library:1.0.1''

}

```

### 1.3 **使用指南**

>####1.在你的布局 XML 声明一个`FlyRefreshLayout`  



>``` xml

  		<com.race604.flyrefresh.FlyRefreshLayout
  		android:id="@+id/fly_layout"
  		android:layout_width="match_parent"
  		android:layout_height="match_parent">

   		 <android.support.v7.widget.RecyclerView
    		  android:id="@+id/list"
    		  android:layout_width="match_parent"
    		  android:layout_height="match_parent"
    		  android:paddingTop="24dp"
   		   android:background="#FFFFFF"/>
		</com.race604.flyrefresh.FlyRefreshLayout>

>```
>####2.在你的Activity或者Fragment中引入`FlyRefreshLayout`，然后你需要设置下拉监听
>```  java
      flyrefreshLayout.setOnPullRefreshListener(new FlyRefreshLayout.OnPullRefreshListener() {
            @Override
            public void onRefresh(FlyRefreshLayout flyRefreshLayout) {
              //刷新时需要完成的逻辑
              }

            @Override
            public void onRefreshAnimationEnd(FlyRefreshLayout flyRefreshLayout) {
                //结束刷新，也就是动画结束的时候需要展示的内容
            }
>``` 

>`recyclerView` 是需要刷新的列表
>```
        java
        LinearLayoutManager linearLayoutManager = new LinearLayoutManager(this);
        linearLayoutManager.setOrientation(LinearLayoutManager.VERTICAL);
        recyclerView.setLayoutManager(linearLayoutManager);


>``` 

### 1.4 **总体设计分析**
由于效果是动画实现，原作者采用把GIF图分解成一帧一帧的图片，分解的方法如下：
>```
convert -coalesce animation.gif frame.png
>```
总体设计是一个下拉刷新的效果;
页面上分为两个部分：头部和内容部分;
头部叠放在内容块的下面;
内容块可以下拉，放手会回弹，并且会触发飞机飞出的动画;
头部块会随着下拉过程中有动画产生（之后会重点来介绍）;
![structure](image/structure.jpg)  
布局主要分为上下两部分，上部为头部，虚线括起来的为内容部分。不触摸屏幕的时候，内容部分会覆盖头部的下半部分，留出头部的`Normal height` 的高度。内容区域可以上滑，最多覆盖到`Shrink height`高度；下滑最多可以把头部区域留出`Expended height`，下滑超过`Normal height`的时候，放手会自动弹回。内容区域可以滑动的距离为`Expended_height - Shrink_height`
这是一个比较通用的布局模式，只要重载这个布局，基本上可以涵盖了所有下刷新的模式。例如`Shrink_height=0`的话，头部可以全部收起来的；如果`Shrink_height==Normal height`的话，就是一个有固定头部的下拉控件；如果`Expended_height > Normal height > Shrink_height`，就是头部可以扩展收缩的下拉控件。
头部动画部分，这里会有一些不同的设计，变化最大的部分。但是有一个共同点，就是头部显示会根据内容块的滑动情况来变化。在软件上，设计出接口，不同的动画，实现此接口就可以。`FlyRefresh` 的动画只是这个接口的一个具体实现。如果要实现其他的刷新动画，并不需要做多大的改动。


##2. 详细设计

###2.1 类详细介绍

>#### 1.[`HeaderController`](https://github.com/race604/FlyRefresh/blob/master/library/src/main/java/com/race604/flyrefresh/HeaderController.java)

>>`HeaderController`是控制刷新框架头部移动的类

>>>`HeaderController`主要涉及到手指滑动头部的移动以及相对于正常位置时偏移量的改变，并且高度是由最顶部往下递增的。

>>>**(1) 主要成员变量含义**  

>>>

1.`mHeight` 正常位置的高度

2.`mMaxHegiht` 可以往下拉的最大高度
 
3.`mMinHegiht` 可以往上推的最小高度

4.`mIsInTouch` 手指是否触碰屏幕

5.`mStartPos` 手指触碰屏幕之前的初始坐标

6.`mCurrentPos` 当前的y方向坐标

>>>**(2) 主要方法含义**  
>>>

1.`public int willMove(float deltaY)` 返回相对于normal位置需要移动的y方向的偏移量，在这个方法中如果偏移量加上初始位置大于正常位置时，偏移量会增加两倍，往上的速度也会相应增加，反之，如果偏移量加上初始位置小于正常位置，往下的偏移量会减少两倍，往下的速度也会相应减少。虽然有点难以理解，但能看到的是刷新框架在振动时，往上振动的力度要大于往下振动的力度，但最终仍会趋于正常位置。

2.`public boolean isOverHeight()` 判断当前位置是否在normal位置以下

3.`public float getMovePercentage()` 获得偏移的百分比

4.`public boolean needSendRefresh()` 判断是否需要开始刷新了(当下拉移动百分比超过0.9f即刷新)


>#### 2.[`PullHeaderLayout`](https://github.com/race604/FlyRefresh/blob/master/library/src/main/java/com/race604/flyrefresh/PullHeaderLayout.java)

>>这是一个基类，主要实现了布局和滑动的功能。这个基类继承自`ViewGroup`,而`ViewGroup`可以包括其他视图，它是一个视图的集合，对应到这个框架上来，也定义了最重要的两个子视图，一个是固定悬浮按钮`FloatingActionButton`还有一个纸飞机的`ImageView`，这个类最大的作用是对Touch事件的处理，因为需要时刻的判断View所处的状态，如果`mContent`可以整体滑动时候，layout就要截获Touch事件，并且把这个Touch事件传递给子视图，这样才能完成刷新的功能。
在这个类中还用到了两个辅助的类，一个是之前的`HeaderController`，还有一个是内部类`ScrollChecker `,这个主要检查`ContentView `是否可以滑动。

>>>**(1) 主要成员变量和常量含义**  

>>>

1.`STATE_IDLE` 闲置状态

2.`STATE_DRAGE` 拉拽状态
 
3.`STATE_FLING` 手指离开屏幕但屏幕还在移动

4.`STATE_BOUNCE` 上下振动状态

5.`mHeaderView` 头部视图

6.`mContent` 内容视图，也就是可伸展的部分，有一部分被头部遮挡。

7.`mScrollChecker` 滑动检查，判断mContent是否可以伸展。

>>>**(2) 主要方法含义**  
>>>

1.`public void setActionDrawable(Drawable actionDrawable)` 这个方法初始化了用到的视图，固定悬浮按钮和纸飞机。

2.`protected void onFinishInflate()`这个方法是渲染视图结束时的回调，这里对两个子视图`mHead`和`mContent`进行了处理，保证了子视图数量在两个以内，并且保证了`mHead`和`mContent`保持对子视图的引用。 

3.`public boolean dispatchTouchEvent(MotionEvent ev)`这个方法显然是对触摸事件的分发，这里着重说一下`ACTION_DOWN`和`ACTION_MOVE`两种情况，第一种`ACTION_DOWN`的时候它会判断控件是不是在上下振动的状态，如果是则立刻停止，这一点很符合我们的习惯。然后是`ACTION_MOVE`,这里做了之前提到的拦截Touch事件的处理：
>```  java
     if (!mHeaderController.isInTouch()) {
      mHeaderController.onTouchDown(ev.getX(), ev.getY());
      offsetY = mHeaderController.getOffsetY();
      }
     willMovePos(offsetY);

>```
这一段代码也就是当recyclerView到顶部以后要进入刷新头部时候执行的。其中最重要的还是`willMovePos(offsetY)`，这个方法把偏移量`offsetY`传给`willMovePos`然后再通过`private void movePos(float delta)`传给头部子视图。

>#### 3.[`FlyRefreshLayout`](https://github.com/race604/FlyRefresh/blob/master/library/src/main/java/com/race604/flyrefresh/FlyRefreshLayout.java)

>>这个类是继承于`PullHeaderLayout`，这个类主要为了简化使用，在这个类中添加了动画头部`MountanScenceView `和刷新的接口`OnPullRefreshListener`，这个类中实现了对纸飞机动画的实现，其中包括三个步骤：
1. 随着下拉，逆时针转动；
2. 放手的时候，触发刷新，发射出去；
3. 刷新完成，飞机飞回来，回到原来的位置。
