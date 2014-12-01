CircularFloatingActionMenu 实现原理解析
====================================
> 本文为 [Android 开源项目实现原理解析](https://github.com/android-cn/android-open-project-analysis) 中 circular-foating-action-menu 部分  
> 项目地址：[CircularFloatingActionMenu](https://github.com/oguzbilgener/CircularFloatingActionMenu)，分析的版本：[e9ccdad](https://github.com/android-cn/android-open-project-demo/commit/1306e632d5a7734cd8451f4e10dff763f9ab4097)，Demo 地址：[circular-foating-action-menu](https://github.com/android-cn/android-open-project-demo/tree/master/CircularFloatingActionMenu-demo)    
> 分析者：[cpacm](https://github.com/cpacm)，校对者：[${校对者}](${校对者 Github 地址})，校对状态：未完成   

###1. 功能介绍  
“一个灵感来自Path路径的Android上可定制圆形浮动菜单动画”  

一个可定制的圆形浮动菜单，我们可以往里面添加子项使其充当一个可缩放的菜单按钮。
####1.1 可定制
可以根据提供的基础库自己编辑动画类，实现自制的动画效果。
####1.2 多方面
不止是在ImageView上可实现，大部分的view都可以充当菜单按钮从而呼出子菜单
![Alt text](https://github.com/android-cn/android-open-project-analysis/blob/master/circular-floating-action-menu/demo.gif "图片样例")

###2. 详细设计
###2.1 核心类功能介绍
主要分成两部分，一部分是构成菜单的view部分，另一部分是动画的操作类
首先是view的部分，主要是三个部件组成:
(1)SubActionButton 选项按钮，即按菜单键弹出来的选项按钮。  

这个类继承自FrameLayout控件，实现一个自定义图标的功能
可以根据构造函数传进来的参数来选择不同风格的图案底纹
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
动画效果主要是通过
    bjectAnimator.ofPropertyValuesHolder(menu.getSubActionItems().get(i).view, pvhX, pvhY, pvhR, pvhsX, pvhsY, pvhA);
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

###3. 流程图
主要功能流程图  
![流程图](https://github.com/android-cn/android-open-project-analysis/blob/master/circular-floating-action-menu/流程图.jpg "流程图")

###4. 总体设计
整个库没有复杂的关系，只是在应用的时候调用导入一下library就行了。

###5. 杂谈
动画的类型有点少，以及不支持分辨率奇葩的机型，如魅族3


###6. 修改完善  
在完成了上面 5 个部分后，移动模块顺序，将  
`2. 详细设计` -> `2.1 核心类功能介绍` -> `2.2 类关系图` -> `3. 流程图` -> `4. 总体设计`  
顺序变为  
`2. 总体设计` -> `3. 流程图` -> `4. 详细设计` -> `4.1 类关系图` -> `4.2 核心类功能介绍`  
并自行校验优化一遍，确认无误后，让`校对 Buddy`进行校对，`校对 Buddy`校队完成后将  
`校对状态：未完成`  
变为：  
`校对状态：已完成`  

**完成时间**  
- `两天内`完成  

**到此便大功告成，恭喜大家^_^**  
