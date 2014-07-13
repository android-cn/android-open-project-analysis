# ListViewAnimations分析
---
#功能介绍
项目主页[https://github.com/nhaarman/ListViewAnimations/](https://github.com/nhaarman/ListViewAnimations/)     
demo apk可在[android-open-project-demo](https://github.com/android-cn/android-open-project-demo/tree/master/listview-animations-demo/apk)下载
	    
正如名字所言，这是一个针对ListView的动画库。通过它我们可以很方便的给ListView（或GridView）增加item动画。  
该库依赖于NineOldAndroids来支持3.0以下的版本。就算不支持3.0以下版本，也需要添加。否则使用时可能会出现问题。  
可通过简单的API调用实现以下**功能**：  
 1. **item展示动画**  
	- 内置动画（底部滑入动画、左边滑入动画、右边滑入动画、item渐变显示、item放大动画）  
	- 其他自定义动画等等  
 2. **item操作**  
	- 滑动删除（以及删除撤销）  
	- item拖动  
	- item（可以是多个item）删除动画  
	- item展开以及动画  

详细用法请参考[https://github.com/nhaarman/ListViewAnimations/wiki](https://github.com/nhaarman/ListViewAnimations/wiki)  
或者demo [https://github.com/android-cn/android-open-project-demo/tree/master/listview-animations-demo](https://github.com/android-cn/android-open-project-demo/tree/master/listview-animations-demo)

#详细分析
##item展示动画
最通俗的说法就是在AbsListview（抽象出来的一个列表基类，包括`ListView`、`GridView`都是它的子类）**调用getView的时候播放一个预定义的动画**。至于**在什么时候播放**、**如何播放**、**播放什么动画**等，下面继续解析。   
众所周知，AbsListview有一个setAdapter用来设置里面每一项item的元素。先看一下调用代码，这里以右边滑入（`SwingRightInAnimationAdapter`）动画为例：  
```java
YourCustomAdapter mYourCustomAdapter = new YourCustomAdapter(this);
SwingRightInAnimationAdapter swingRightInAnimationAdapter = new SwingRightInAnimationAdapter(mYourCustomAdapter);
swingRightInAnimationAdapter.setAbsListView(mListView);
mListview.setAdapter(swingRightInAnimationAdapter);
```
与之前的Adapter区别在于设置mListview  adapter前先用swingRightInAnimationAdapter对mYourCustomAdapter进行处理，然后传入mYourCustomAdapter，同时，给swingRightInAnimationAdapter传入调用的AbsListview。 
回到之前说的本质，就是在getView的时候播放动画，也就说，swingRightInAnimationAdapter肯定对getView方法进行了处理。找到`SwingRightInAnimationAdapter`类的代码，发现没有getView这个方法，继续找父类。它的父类是`SingleAnimationAdapter`，它也没有。继续找父类，`AnimationAdapter`，太棒了，里面有getView方法，我们看下内容是什么。  
 
  ![getView](https://github.com/android-cn/android-open-project-analysis/raw/master/listview-animations/images/items_animation_getview.png)
  mHasParentAnimationAdapter是做什么用的呢。由于动画库可以同时叠加使用多个动画，先看下如何使用多个动画  
```java
YourCustomAdapter mAdapter = new YourCustomAdapter (this);
SwingRightInAnimationAdapter swingRightInAnimationAdapter = new SwingRightInAnimationAdapter(mAdapter);
SwingBottomInAnimationAdapter swingBottomInAnimationAdapter = new SwingBottomInAnimationAdapter(swingRightInAnimationAdapter);

swingRightInAnimationAdapter.setAbsListView(mListview);
mListview.setAdapter(swingBottomInAnimationAdapter);
```
与上面的单个动画相比，在于swingRightInAnimationAdapter不是直接传给mListview的setAdapter，而是，传给swingBottomInAnimationAdapter的构造函数，然后，swingRightInAnimationAdaptermListview的setAdapter。  
可以肯定的是mAdapter，swingRightInAnimationAdapter，swingBottomInAnimationAdapter的getView方法都被调用了，上述的调用顺序为swingBottomInAnimationAdapter.getView、swingRightInAnimationAdapte.getView、mAdapter.getView。既然动画是在getView里面进行播放的，那岂不是要播放多次？为了解决这个问题，这里用mHasParentAnimationAdapter来进行标示，里层的getView有些操作是不进行的（比如播放动画），这个变量不需要人为维护，在`AnimationAdapter`里面进行了处理，代码如下。   

 ![getView](https://github.com/android-cn/android-open-project-analysis/raw/master/listview-animations/images/items_animation_animationadapter_constructor.png)
也就是说只要传进去的adapter是`AnimationAdapter`（可以播放动画的adapter），那这个adapter就不进行某些操作。  
回到之前的那段多个动画组合的代码swingRightInAnimationAdapter是不进行某些操作的，动画播放等等是在swingBottomInAnimationAdapter的getView进行的，也就是最外层的`AnimationAdapter`进行动画操作。  
切回到`AnimationAdapter`的getView方法，这里假设Adapter是最完成的AnimationAdapter，也就是说mHasParentAnimationAdapter为false。大致的流程很简单，判断AbsListview有没有设置，然后取消之前convertView的动画（AbsListView里面的getView是循环利用view的）。接着调用父类的getView获得View对象。然后调用animateViewIfNecessary播放动画。 
**以上，我们就解决了在什么时候播放动画的问题**。  
**如何播放动画以及什么时候播放动画呢**，继续看  

  ![getView](https://github.com/android-cn/android-open-project-analysis/raw/master/listview-animations/images/items_animation_animateviewifnecessary.png)  
153行判断是否该播放动画，如果大于上次播放后的最后位置，则不播放，这就是为什么在滑动时只有往后滑才会播放动画，往前滑不会播放动画的原因。然后`AnimationAdapter`还提供了一个函数setShouldAnimate来设置是否播放动画。如果满足播放动画的条件，则调用animateView播放动画，同时保存位置。  
 
  ![getView](https://github.com/android-cn/android-open-project-analysis/raw/master/listview-animations/images/items_animation_animateview.png)
animationView是播放动画的重头戏。  
173行调用抽象函数getAnimators（这个就是AnimationAdapter子类实现动画的地方）获取动画。然后后面计算动画延时时间并进行播放。
`AlphaInAnimationAdapter`、`ScaleInAnimationAdapter`等都是继承自`AnimationAdapter`然后重写getAnimation函数。`SwingBottomInAnimationAdapter`、`SwingRightInAnimationAdapter`、`SwingLeftInAnimationAdapter`继承自`SingleAnimationAdapter`，`SingleAnimationAdapter`继承自`AnimationAdapter`。
也就是说我们只要继承`AnimationAdapter`并重写getAnimation就可以自定义动画啦。也可以继承自`ResourceAnimationAdapter`从而加载xml布局中的动画。   
OK，三个问题都解决了。  
以上就是AbsListview的item动画，原理很简单，就是在getView里面在适当的实际播放动画而已。   
以下是类图（使用PowerDesigner生成）  

![getView](https://github.com/android-cn/android-open-project-analysis/raw/master/listview-animations/images/items_animation_class_diagram.png)   
综上，动画分析就此结束。   
