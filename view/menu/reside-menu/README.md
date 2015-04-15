ResideMenu 源码解析
====================================
> 本文为 [Android 开源项目源码解析](https://github.com/android-cn/android-open-project-analysis) 中 ResideMenu 部分  
> 项目地址：[ResideMenu](https://github.com/SpecialCyCi/AndroidResideMenu)，分析的版本：[70d46c2](https://github.com/SpecialCyCi/AndroidResideMenu/tree/70d46c2ed49c8123847bb07d2f0761e57bb3bc04)，Demo 地址：[ResideMenu-demo](https://github.com/android-cn/android-open-project-demo/tree/master/photoview-demo)    
> 分析者：[dkmeteor](https://github.com/dkmeteor)，校对者：[???]()，校对状态：未完成   


###1. 功能介绍

#####特性(Features)：
通过手势滑动缩放主界面展现的侧边栏，类似QQ5.0+版本侧滑菜单的出现方式，支持左右双侧边栏。

动效文字比较难以描述，请以演示效果Gif为例：

#####优势：
- 性能优异，几乎没有额外的重绘和性能损耗。
- 良好的结构设计，易于扩展和改写。
- 与DrawerLayout或SlidingMenu相比，完全是另一种风格，第一次见到时给人眼前一亮的感觉。
- 事件分发做了很好的处理，可以方便的与其它控件集成。


###2. 总体设计

这个库本身只包含3个Class,类结构比较简单，但是View层级之间的逻辑比较复杂，下面会尽量用示意图和流程图的方式进行介绍，方便大家理解。

View层级结构图：

TODO

滑动时动画效果示意图：

TODO
    
###3. 事件分发流程图

TODO

###4. 详细设计
###4.1 核心类功能介绍

---
##### 4.1.1 ResideMenu
核心类

TODO 重点

- private AnimatorSet buildScaleDownAnimation(View target,float targetScaleX,float targetScaleY)
- private AnimatorSet buildScaleUpAnimation(View target,float targetScaleX,float targetScaleY)
- private AnimatorSet buildMenuAnimation(View target, float alpha)
- private void initValue(Activity activity)
- public void attachToActivity(Activity activity)
- protected boolean fitSystemWindows(Rect insets)
- public void openMenu(int direction)
- public void closeMenu()
- private void showScrollViewMenu(ScrollView scrollViewMenu)
- private void hideScrollViewMenu(ScrollView scrollViewMenu)
- private void setScaleDirectionByRawX(float currentRawX)
- private float getTargetScale(float currentRawX)
- public boolean dispatchTouchEvent(MotionEvent ev)

---
TODO 不重要的设置类方法

- private void setShadowAdjustScaleXByOrientation()
- public void addMenuItem(ResideMenuItem menuItem)
- public void addMenuItem(ResideMenuItem menuItem, int direction)
- public void setMenuItems(List<ResideMenuItem> menuItems)
- public void setMenuItems(List<ResideMenuItem> menuItems, int direction)
- private void rebuildMenu()
- public List<ResideMenuItem> getMenuItems()
- public List<ResideMenuItem> getMenuItems(int direction) 
- public void setMenuListener(OnMenuListener menuListener)
- public void setDirectionDisable(int direction)
- public void setSwipeDirectionDisable(int direction)
- private boolean isInDisableDirection(int direction)
- private void setScaleDirection(int direction)
- public void addIgnoredView(View v)
- public void removeIgnoredView(View v)
- public void clearIgnoredViewList()
- private boolean isInIgnoredView(MotionEvent ev)

##### 4.1.2 ResideMenuItem
略
##### 4.1.3 TouchDisableView
该类本身的功能非常单纯，重载onInterceptTouchEvent和isTouchDisabled方法并固定返回false，拦截该View及其内部所有子View的Touch事件。
请结合事件分发流程图一起阅读源码。

###5. 杂谈
TODO 如果Folder-ResideMenu能完成，就补充到这里来。
