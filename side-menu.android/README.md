Side Menu.Android 源码解析
====================================
> 本文为 [Android 开源项目源码解析](https://github.com/aosp-exchange-group/android-open-project-analysis) 中 Side Menu.Android 部分  
> 项目地址：[Side Menu.Android](https://github.com/Yalantis/Side-Menu.Android)，分析的版本：[2c23bff](https://github.com/Yalantis/Side-Menu.Android/commit/2c23bff1dbebb87b3a3291e3f7d629cc0d5efbfa)，Demo 地址：[side-menu-demo](https://github.com/aosp-exchange-group/android-open-project-demo/tree/master/side-menu-demo)    
> 分析者：[cpacm](https://github.com/cpacm)，校对者：无， 校对状态：未进行  

##1. 功能介绍  
一个提供不同选项的侧边菜单——`Side Menu`。

####1.1 特点
提供了一个翻页动画——`Flip Animation`。  
基于Android 5.0开发。  
使用ToolBar控制菜单。  

####1.2 要求
**（1）**界面布局使用DrawerLayout为容器，当通过xml来布局的话，需要把DrawerLayout作为父容器，内容布局作为其第一个子节点，而菜单布局则紧随其后作为第二个子节点，这样做就能把内容展示区和菜单区分开来，之后只需要分别在这两个区域里设置内容即可。  
**（2）**内容布局中，使用Toolbar并将其作为菜单界面的载体，通过Toolbar来控制菜单界面的打开和关闭。  
**（3）**内容布局中的信息界面需要继承ScreenShotable接口（如demo中的ContentFragment），以便切换信息。  
**（4）**主界面需要继承ViewAnimator.ViewAnimatorListener接口（如demo中的myActivity）。
 
##2. 流程图  
![流程图](image/side_menu.jpg "流程图")

##3. 详细介绍  
####3.1 类
*（一）ViewAnimator类* `Side Menu 的管理类`  
1、ViewAnimator的构造函数需要传入5个参数，分别为`作为主界面的ActionBarActivity`、`子按钮列表`、`继承ScreenShotable接口的信息界面`、`DrawerLayout`、`继承ViewAnimatorListener接口的主界面`。  
```java
public ViewAnimator(ActionBarActivity activity,
                        List<T> items,
                        ScreenShotable screenShotable,
                        final DrawerLayout drawerLayout,
                        ViewAnimatorListener animatorListener) {
        this.actionBarActivity = activity;
        this.list = items;
        this.screenShotable = screenShotable;
        this.drawerLayout = drawerLayout;
        this.animatorListener = animatorListener;
    }
```
2、showMenuContent()方法，打开菜单界面。  
简单地将其分成几部分：（1）在菜单未完全打开前设置按钮为不可用同时调用了ViewAnimatorListener接口中的disableHomeButton()方法。清空原先存放按钮视图的列表。（2）根据传入的按钮个数生成相应个数的按钮View,并为每个按钮添加点击事件，当事件发生时调用
```java
/**
 *参数分别为：选中的按钮，当前的信息界面，触摸点的Y坐标
 **/
animatorListener.onSwitch(slideMenuItem, screenShotable, topPosition)
```
方法并关闭菜单列表。（3）将其添加到存放按钮视图的列表中。调用AnimatorListener接口的`addViewToContainer(viewMenu)`方法（我们要在Activity中人为的将其添加到界面布局中）。在菜单打开动画未完成情况下，将其属性设为不可用。调用`animateView（）`方法使用FlipAnimation类来实现动画设置，Handler实现延时播放。  
3、hideMenuContent()方法，关闭菜单界面。  
为每个按钮视图调用`animateHideView()`方法来设置关闭的动画，并通过Handler进行延时播放。在动画结束的监听器中设置视图不可见，并但视图是最后一个按钮时调用
```java
animatorListener.enableHomeButton();//回调函数，使主界面的菜单键生效
drawerLayout.closeDrawers();
```
*（二）SlideMenuItem类* `选项按钮类`   
一个按钮容器类，里面存放着两个变量。主要用来设置菜单按钮。
```java
    private String name;//名称
    private int imageRes;//图片id
```
*（三）FlipAnimation类* `翻转动画`  
一个翻转动画工具类，继承自Animation类。根据传入的参数来实现不同的翻转效果。
```java
 /**
  * 参数分别为：起始角度，终止角度，中心点的X坐标，中心点的Y坐标
  **/
 public FlipAnimation(float fromDegrees, float toDegrees,float centerX, float centerY) {
 ...
 }
```
*（四）Resourceble接口* `选项接口`  
按钮选项必须继承该接口(如SlideMenuItem就继承了该接口)，用于存放资源。
```java
public interface Resourceble {
    public int getImageRes();
    public String getName();
}
```
*（五）ScreenShotable接口* `用于信息变更的接口`   
该接口包含两个方法：
```java
public interface ScreenShotable {
    public void takeScreenShot();
    public Bitmap getBitmap();
}
```
1、takeScreenShot()方法是在按钮被点击时触发。  
2、getBitmap()方法,获取当前显示的Bitmap。  

####3.2 类关系图
![类图](image/side_menu_class.jpg "类图")

##4.个人总结
一个很简单的一个项目，所以从难度上没有什么好讲的（自己的收获则是熟悉了在Android5.0下Toolbar的用法）。  
我认为问题最大的是它的扩展性太差，从ViewAnimator类的构造函数就能发现这个库一定要在相应的环境下（ActionBarActivity,DrawerLayout）才会产生作用。  
其次是动画类型不能让开发者进行二次定制，要想改变动画类型则要进library库进行修改。  
不过我认为它最大的意义在于提供了一种在Toolbar下自定义菜单的思路，这是最值得我们借鉴的~
