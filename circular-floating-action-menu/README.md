CircularFloatingActionMenu 实现原理解析
====================================
> 本文为 [Android 开源项目实现原理解析](https://github.com/android-cn/android-open-project-analysis) 中 circular-foating-action-menu 部分  
> 项目地址：[CircularFloatingActionMenu](https://github.com/oguzbilgener/CircularFloatingActionMenu)，分析的版本：[8efb1aa](https://github.com/oguzbilgener/CircularFloatingActionMenu/commit/8efb1aab2b361ed9019fa4af6e5d43e77777bcb6)，Demo 地址：[circular-floating-action-menu](https://github.com/android-cn/android-open-project-demo/tree/master/CircularFloatingActionMenu-demo)    
> 分析者：[cpacm](https://github.com/cpacm)，校对者：[dkmeteor](https://github.com/dkmeteor)，校对状态：未完成   

###1. 功能介绍  
“一个灵感来自Path路径的Android上可定制圆形浮动菜单动画”  

一个可定制的圆形浮动菜单，我们可以往里面添加子项使其充当一个可缩放的菜单按钮。
####1.1 可定制
可以根据提供的基础库自己编辑动画类，实现自制的动画效果。
####1.2 多方面
不止是在ImageView上可实现，大部分的view都可以充当菜单按钮从而呼出子菜单
![Alt text](https://github.com/android-cn/android-open-project-analysis/blob/master/circular-floating-action-menu/demo.gif "图片样例")

###2. 总体设计
###2.1 核心类功能介绍
主要分成两部分，一部分是构成菜单的view部分，另一部分是动画的操作类

首先是view的部分，主要是三个部件组成:
(1)SubActionButton 选项按钮，即按菜单键弹出来的选项按钮。  

这个类继承自FrameLayout控件，实现一个自定义图标的功能  
可以根据构造函数传进来的参数来选择不同风格的图案底纹，
然后将其传给menu菜单以便控制.  

(2)FloatingActionButton 菜单按钮，点击即可唤出SubActionButton按钮  

这个类跟SubActionButton基本相似，同样可以通过内部自定义的build构造器来定制自己的按钮。  

（3）FloatingActionMenu 那么最重要的类来了，它存放着所有的按钮以及动画操作。
基本结构图如下
![Alt text](https://github.com/android-cn/android-open-project-analysis/blob/master/circular-floating-action-menu/menu.jpg "menu")  

接下来是动画部分
(1)MenuAnimationHandler
这是是所有动画类的父类，它主要定义了菜单打开，关闭，以及运行结束后状态的保存的方法  
 
    restoreSubActionViewAfterAnimation(FloatingActionMenu.Item subActionItem, ActionType actionType)
	animateMenuOpening(Point center)
	animateMenuClosing(Point center)  
	
（2）DefaultAnimationHandler
这一个默认的动画类，当我们不对动画做修改时就会默认使用这个类里面的动画效果。我们也可以参考这个类来进行设计新的动画效果
动画效果主要是通过ObjectAnimator.ofPropertyValuesHolder(menu.getSubActionItems().get(i).view, pvhX, pvhY, pvhR, pvhsX, pvhsY, pvhA);
来实现

###2.2 如何使用
    // Set up the white button on the lower right corner
        // more or less with default parameter
        ImageView fabIconNew = new ImageView(this);
        fabIconNew.setImageDrawable(getResources().getDrawable(R.drawable.ic_action_new_light));
        FloatingActionButton rightLowerButton = new FloatingActionButton.Builder(this)
                .setContentView(fabIconNew)
                .build();

        SubActionButton.Builder rLSubBuilder = new SubActionButton.Builder(this);
        ImageView rlIcon1 = new ImageView(this);
        ImageView rlIcon2 = new ImageView(this);
        ImageView rlIcon3 = new ImageView(this);
        ImageView rlIcon4 = new ImageView(this);

        rlIcon1.setImageDrawable(getResources().getDrawable(R.drawable.ic_action_chat_light));
        rlIcon2.setImageDrawable(getResources().getDrawable(R.drawable.ic_action_camera_light));
        rlIcon3.setImageDrawable(getResources().getDrawable(R.drawable.ic_action_video_light));
        rlIcon4.setImageDrawable(getResources().getDrawable(R.drawable.ic_action_place_light));

        // Build the menu with default options: light theme, 90 degrees, 72dp radius.
        // Set 4 default SubActionButtons
        FloatingActionMenu rightLowerMenu = new FloatingActionMenu.Builder(this)
                .addSubActionView(rLSubBuilder.setContentView(rlIcon1).build())
                .addSubActionView(rLSubBuilder.setContentView(rlIcon2).build())
                .addSubActionView(rLSubBuilder.setContentView(rlIcon3).build())
                .addSubActionView(rLSubBuilder.setContentView(rlIcon4).build())
                .setAnimationHandler(new SliderAnimationHandler())
                .attachTo(rightLowerButton)
                .build();
如以上代码所示  

（1）先建立一个view来作为一个总容器，设置好图片，然后作为菜单的按钮  

（2）建立好选项菜单的视图，添加属性后，添加到FloatingActionMenu中的ArrayList<item>数组中，并同时绑定上面的菜单按钮。  

（3）如果使用自己定义的动画，setAnimationHandler(new SliderAnimationHandler())。  

这样子，一个简单的案例就做好了

![流程图](https://github.com/android-cn/android-open-project-analysis/blob/master/circular-floating-action-menu/流程图.jpg "流程图")
###3. 流程图
![设计流程图](https://github.com/android-cn/android-open-project-analysis/blob/master/circular-floating-action-menu/circlemenu.jpg "流程图")
  
总体的设计流程图如上图所示，中间最复杂的可能是计算view位置的地方。
###4. 详细设计

##SubActionButton
首先是构造函数
```java
public SubActionButton(Activity activity, LayoutParams layoutParams, int theme, Drawable backgroundDrawable, View contentView, LayoutParams contentParams) {
        super(activity);
        setLayoutParams(layoutParams);
        // If no custom backgroundDrawable is specified, use the background drawable of the theme.
        if(backgroundDrawable == null) {
            if(theme == THEME_LIGHT) {
                backgroundDrawable = activity.getResources().getDrawable(R.drawable.button_sub_action_selector);
            }
            else if(theme == THEME_DARK) {
                backgroundDrawable = activity.getResources().getDrawable(R.drawable.button_sub_action_dark_selector);
            }
            else if(theme == THEME_LIGHTER) {
                backgroundDrawable = activity.getResources().getDrawable(R.drawable.button_action_selector);
            }
            else if(theme == THEME_DARKER) {
                backgroundDrawable = activity.getResources().getDrawable(R.drawable.button_action_dark_selector);
            }
            else {
                throw new RuntimeException("Unknown SubActionButton theme: " + theme);
            }
        }
        else {
            //通过mutate()方法解决Drawable共用一个内存空间的问题
            backgroundDrawable = backgroundDrawable.mutate().getConstantState().newDrawable();
        }
        //设置背景（考虑版本问题）
        setBackgroundResource(backgroundDrawable);
        if(contentView != null) {
            //添加view(即菜单的选项视图)
            setContentView(contentView, contentParams);
        }
        setClickable(true);
    }
```

从构造函数可以看的出来，选项按钮有四个主题可以选择，分别是下面的四种颜色
```java
    public static final int THEME_LIGHT = 0;
    public static final int THEME_DARK = 1;
    public static final int THEME_LIGHTER = 2;
    public static final int THEME_DARKER = 3;
```

之后是设定ImageView到这个按钮上，并且设定与父view的距离。（通过setMargins()）  
这个我们在创建subActionButton时就要调用。核心函数是addView(contentView,params)。这个方法能够在视图上再添加一个view，作为子视图。
```java
/**
     * Sets a content view with custom LayoutParams that will be displayed inside this SubActionButton.
     * @param contentView
     * @param params
     */
    public void setContentView(View contentView, LayoutParams params) {
        if(params == null) {
            params = new LayoutParams(LayoutParams.WRAP_CONTENT, LayoutParams.WRAP_CONTENT, Gravity.CENTER);
            final int margin = getResources().getDimensionPixelSize(R.dimen.sub_action_button_content_margin);
            params.setMargins(margin, margin, margin, margin);
        }

        contentView.setClickable(false);
        this.addView(contentView, params);
    }
```

最后就是一个建造器了，专门生成用于生成该类的建造器，静态全局
```java
 /**
     * A builder for {@link com.cpacm.library.SubActionButton} in conventional Java Builder format
     * 菜单选项的建造器
     */
    public static class Builder {
        ...
       public SubActionButton build() {
            return new SubActionButton(activity,
                    layoutParams,
                    theme,
                    backgroundDrawable,
                    contentView,
                    contentParams);
        }
    }
```
传入activity，视图特性配置，主题的id,背景图，imageview（子视图），imageview（子视图）的特性配置。用这些来配置选项按钮。
  
##FloatingActionButton
菜单按钮其实跟选项按钮的代码模式差不多，也是由设定子视图和一个建造器组成。  
不过它多了几个方法：    
设定位置，如左下，右下等方位
```java
/**
     * Sets the position of the button by calculating its Gravity from the position parameter
     * @param position one of 8 specified positions.
     * @param layoutParams
     */
    public void setPosition(int position, FrameLayout.LayoutParams layoutParams) {
        int gravity;
        switch(position) {
            ...//具体代码请自行查看源代码
        }
        layoutParams.gravity = gravity;
        setLayoutParams(layoutParams);
    }
```

将视图绑定到activity的主视图中。这样我们就能在activity的主视图中操作这个view了。


FloatingActionButton的建造器
```java
/**
     * A builder for {@link com.cpacm.library.FloatingActionButton} in conventional Java Builder format
     */
    public static class Builder {
    ...
        public FloatingActionButton build() {
                return new FloatingActionButton(activity,
                                               layoutParams,
                                               theme,
                                               backgroundDrawable,
                                               position,
                                               contentView,
                                               contentParams);
            }
    }
```
比SubActionButton多了一个位置的属性。

##FloatingActionMenu
这个类也是由一个建造器生成，那么我们从建造器开始说起  
我们先看看生成Menu的代码：
```java
FloatingActionMenu rightLowerMenu = new FloatingActionMenu.Builder(this)
                .addSubActionView(rLSubBuilder.setContentView(rlIcon1).build())
                .addSubActionView(rLSubBuilder.setContentView(rlIcon2).build())
                .addSubActionView(rLSubBuilder.setContentView(rlIcon3).build())
                .addSubActionView(rLSubBuilder.setContentView(rlIcon4).build())
                .setAnimationHandler(new SliderAnimationHandler())
                .attachTo(rightLowerButton)
                .build();
```

* Builder(this)将activity传入menu中
* addSubActionView 添加选项按钮到activity的视图中。在FloatingActionMenu中管理SubActionView是一个Item的list集合，每次加一个按钮就往里面添加。Item是一个辅助类，里面包括一个视图，x坐标,y坐标,长度,宽度。
* setAnimationHandler 则是设定动画。
* attachTo是将menu与activity的视图绑定。（即把菜单按钮的视图添加到activity的视图中）  

FloatingActionMenu类主要是管理菜单按钮和选项按钮的位置和状态（开和关）  
（1）首先是通过view的onClick监听器来控制状态  

（2）开关主要是两种状态，开的时候会获得菜单按钮的中心位置center（getActionViewCenter()）和计算item的位置（calculateItemPositions()）。然后发送动画的请求到AnimationHandler中（animationHandler.animateMenuOpening(center)）。
```java
    /**
     * Simply opens the menu by doing necessary calculations.
     * @param animated if true, this action is executed by the current {@link MenuAnimationHandler}
     */
    public void open(boolean animated) {
        ...//具体代码请自行查看源代码
    }
```
其中item的x,y是记录视图的终点位置，然后经过动画把view移到x,y的位置上。  

stateChangeListener为状态变化的监听器，开关都会响应相应的方法。主要在AnimationHandler中添加具体方法。
```java
/**
     * A listener to listen open/closed state changes of the Menu
     */
    public static interface MenuStateChangeListener {
        public void onMenuOpened(FloatingActionMenu menu);
        public void onMenuClosed(FloatingActionMenu menu);
    }
```
（3）计算位置
```java
    /**
     * Calculates the desired positions of all items.
     */
    private void calculateItemPositions() {
        ...//具体代码请自行查看源代码
    }
```

##DefaultAnimationHandler
动画实现的主要类，继承自MenuAnimationHandler    
主要通过Animator来实现属性动画。    
里面有一个restoreSubActionViewAfterAnimation的方法，它主要是恢复选项按钮到未打开的状态。 
```java
    /**
     * Restores the specified sub action view to its final state, accoding to the current actionType
     * Should be called after an animation finishes.
     * @param subActionItem
     * @param actionType
     */
    protected void restoreSubActionViewAfterAnimation(FloatingActionMenu.Item subActionItem, ActionType actionType) {
        ...//具体代码请自行查看源代码
    }
```
Animator属性动画以及其他动画的实现请参考我写的博客  
[Android的动画效果](http://www.cnblogs.com/cpacm/p/4067283.html)

![uml](https://github.com/android-cn/android-open-project-analysis/blob/master/circular-floating-action-menu/menu_uml.jpg "uml")



###5. 杂谈
动画的类型有点少，以及在屏幕尺寸异常的机子上测试时（如mx3的1800x1080）会出现子选项偏离中心菜单键的问题，原因出在view的位置计算上，它没有考虑到一些特殊机型的机子。


