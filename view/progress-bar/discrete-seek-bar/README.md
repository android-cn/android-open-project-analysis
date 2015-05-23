${项目名} 源码解析
====================================
> 本文为 [Android 开源项目源码解析](https://github.com/android-cn/android-open-project-analysis) 中 DiscreteSeekBar 部分  
> 项目地址：[anderWeb/discreteSeekBar](https://github.com/AnderWeb/discreteSeekBar)，分析的版本：[f54f0cd6](https://github.com/AnderWeb/discreteSeekBar/commit/f54f0cd64cd33da9effe9103d80bcc408178d171 "Commit id is f54f0cd64cd33da9effe9103d80bcc408178d171")，Demo 地址：[discrete-seek-bar-demo](https://github.com/aosp-exchange-group/android-open-project-demo/tree/master/discrete-seek-bar-demo)    
> 分析者：[wangeason](https://github.com/wangeason})，分析状态：已完成，校对者：[Trinea](https://github.com/trinea)，[huxian99](https://github.com/huxian99) 校对状态：未开始   

###1. 功能介绍  

DiscreteSeekBar实现了类似Material design风格的Discrete Slider。DiscreteSeekBar可以在2.1以上的应用中使用，可以直接在xml中配置，使用方法类似SeekBar，很简单。

可以在xml中配置显示的格式，也可以在代码中自定义显示的数字或者指定显示字符。

###2. 总体设计

这是一个材料设计的seekbar，其主要的几个类分别对应了这个seekbar的几个主要组件。

ThumbDrawable -> seekBar没有被按下时，seekBar上的那个圆点；

TrackRectDrawable -> seekBar的进度条，设置不同的宽度和颜色后，被用作背景和前景的显示；

MarkerDrawable -> Thumb被按下时的显示状态的背景，这个类里面封装了动画打开和关闭，提供了颜色设置的接口；

Marker -> 是个ViewGroup，包含一个TextView和一个MarkerDrawable。Thumb被按下时的显示状态, 增加了文字显示，宽度计算等，实现了文本设置的接口；

PopupIndicator.Floater -> FrameLayout, 封装了一个Marker在里面，实现Marker的滑动；

PopupIndicator -> 最终被集成到DiscreteSeekBar类，保证Floater始终和Thumb
的x坐标相同；

DiscreteSeekBar -> 集成了上述的组件的实例、OnProgressChangeListener接口、从xml中获取设置、提供api方法。总之就是让这个View看着像一个SeekBar；

###3. 详细设计  
###3.1 类关系图

![classes](classes.gif)

###3.2核心类功能介绍  
####3.2.1 StateDrawable  
抽象类StateDrawable继承自Drawable，根据状态切换Drawable的颜色
```java
//根据状态判断是否应该刷新当前的色彩
private boolean updateTint(int[] state)
//抽象方法，具体的子类实现具体的画法
abstract void doDraw(Canvas canvas, Paint paint);
```

####3.2.2 AlmostRippleDrawable  
继承自抽象类StateDrawable，顾名思义，总是有Ripple效果。  
通过一个mUpdater的Runnable对象，不断地做drawCircle，来达到低版本的Ripple效果。  

####3.2.3 MarkerDrawable  
```java
private void computePath(Rect bounds)
```  
这个Marker实际是用path画了一个3个圆角，一个直角的正方形, 再用Matrix调整它的角度和相对位置。 把这个动画的完成度作为入参来调整这个图形，完成从圆形到marker的变化。

```java
public void animateToPressed()
```  
动画变换到marker，动画完成时，调用打开完成的接口MarkerAnimationListener.onOpeningComplete  

```java
public void animateToNormal()
```  
动画变换到关闭状态，动画完成时，调用关闭完成的接口MarkerAnimationListener.onClosingComplete

```java
private void updateAnimation(float factor)
```  
根据开始动画的时间和动画的方向计算动画的完成度，并调用computePath方法画Marker  

```java
private static int blendColors(int color1, int color2, float factor)
```  
根据设置的开始和结束颜色，还有动画完成度，调出当前颜色，注意：这里的factor和updateAnimation中的factor不一样，已经是计算结果了。

####3.2.4 ThumbDrawable

seekBar上的圆形按钮，在按下以后调用animateToPressed,100ms以后不会再绘制，直到再次调用animateToNormal，因为按下以后会绘制marker。 这里有个疑问为什么要在100ms以后，作者的解释是：This special delay is meant to help avoiding frame glitches while the Marker is added to the Window。 这100ms用来绘制Marker，避免同时绘制Thumb出现掉帧，感觉卡顿。

####3.2.5 TrackRectDrawable
绘制矩形，用来画ProgressBar和Track

####3.2.6 TrackOvalDrawable
没有调用和实现，应该是作者准备用来做圆形seekBar的。

####3.2.7 Marker
```java
public void resetSizes(String maxValue)
```  
这个方法根据seekbar的最大值来确定marker的宽度，如果有负数的时候会出现bug，如果改成最大和最小值中的最大位数就不会可以在显示整数的时候避免这个问题。  

```java
public void animateOpen()
```
在onAttachedToWindow()被调用，动画打开Marker。同时PopupIndicator中的showIndicator()和invokePopup()都会调用该方法。  

```java
public void animateClose()
```
在PopupIndicator中被调用，动画关闭Marker，并设置mNumber为INVISIBLE。  

```java
public void setColors(int startColor, int endColor)
```  
设置MarkerDrawable的动画开始和结束时的颜色。  

####3.2.8 Floater  
PopupIndicator的内部类，用来实现Marker的滑动
```java
public void setFloatOffset(int x)
```  
通过设置childview的Marker的左右偏移，来实现Floater的滑动，被PopupIndicator调用，实现Marker的滑动。

<Li>MarkerAnimationListener的实现  
在onClosingComplete()中，把PopupIndicator中的Floater删除了。  

####3.2.9 PopupIndicator  
用来管理Floater的指示器  
```java
public void updateSizes(String maxValue)
```  
主要调用Marker的resetSizes(String maxValue)方法，来确定Marker的宽度。  

```java
private void measureFloater()
```  
通过调用Floater的measure方法，设置其宽度为full-width，作者说这里有待改进

```java
public void setValue(CharSequence value)
```  
设置Marker的TextView值  

```java
public void showIndicator(View parent, Rect touchBounds)
```  
在指定位置显示出Indicator，Rect touchBounds为DiscreteSeekBar传入的Thumb的Bounds。在DiscreteSeekBar的showFloater()中调用。

```java
private WindowManager.LayoutParams createPopupLayout(IBinder token)  
private void updateLayoutParamsForPosiion(View anchor, WindowManager.LayoutParams p, int yOffset)  
```
创建和编辑Floater的LayoutParams  

```java
private void invokePopup(WindowManager.LayoutParams p)
```  
添加Floater，动画打开Marker  

####3.2.10 DiscreteSeekBar  
onTouchEvent事件主要调用了如下三个方法  
```java
private boolean startDragging(MotionEvent ev, boolean ignoreTrackIfInScrollContainer)
private void updateDragging(MotionEvent ev)
private void stopDragging()
```  
顾名思义，这3个方法主要就是为了`拖动Thumb`的。  

```java
public interface OnProgressChangeListener
```  
用户实现，监听DiscreteSeekBar数值变化  

```java
public static abstract class NumericTransformer {
    public abstract int transform(int value);

    public String transformToString(int value) {
        return String.valueOf(value);
    }

    public boolean useStringTransform() {
        return false;
    }
}
```  
其中抽象方法public abstract int transform(int value)根据DiscreteSeekBar的数值value，返回要在Marker中显示的数值，而且这个返回的数值会被设置的正则Formatter转换，默认的Formatter是DEFAULT_FORMATTER = "%d"，这个Formatter可以通过setIndicatorFormatter方法或者xml设置

useStringTransform()方法默认返回false，调用transform(int value)方法根据Formatter转换，如果return true，则会调用transformToString(int value)方法，表示这个字符不会被Formatter转换。  

```java
public void setNumericTransformer
```  
设置Transformer，并刷新Marker上的显示

```java
public void setMax(int max) 
public void setMin(int min)
```  
这两个方法用来设置DiscreteSeekBar的最大最小值，并且刷新如果用按键控制SeekBar时的步进值。

```java
public void setProgress(int progress)  
```  
设置进度，其中检查了这个progress，使它过大和过小时都能正常运行

```java
private void notifyProgress(int value, boolean fromUser)
```
调用 onProgressChanged接口，并且调用一个空的方法onValueChanged，当有DiscreteSeekBar的继承类的时候，可以复写onValueChanged，而不必实现onProgressChanged。

```java
private void notifyBubble(boolean open)
```
作用类似于 onValueChanged，只不过这个监听的是 Floater、Marker的消失和显示

```java
private void updateFromDrawableState()
```  
根据Drawable的状态来设置不同drawable的动画。这个方法被onLayout和drawableStateChanged调用。  

```java
private void updateThumbPos(int posX)
```  
根据滑动的拖动的位置绘制进度条（mScruber）和mThumb  

###4. 杂谈
1.  Marker的宽度的设定太死板，当设定的min值得长度大于max值得时候min值就没法完整显示了。
2.  进度条上没有标记点，可以考虑在背景trackBar上设置一些点，这些点着重突出出来，在scruber滑过这些点的时候，出一些效果。这个我准备完成以后再和大家分享。


