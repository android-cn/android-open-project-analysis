View 事件传递
----------------
> 本文为 [Android 开源项目源码解析](https://github.com/android-cn/android-open-project-analysis) 公共技术点中的 View 事件传递 部分  
 分析者：[Trinea](https://github.com/Trinea)，校对者：[Trinea](https://github.com/Trinea)，校对状态：完成  

本文后面后继续整理。  

推荐一篇我看到的对传递机制介绍最清楚的国外文章吧。本文略作翻译。  

### 1、基础知识 
(1) 所有Touch事件都被封装成了MotionEvent对象，包括Touch的位置、时间、历史记录以及第几个手指(多指触摸)等。  

(2) 事件类型分为ACTION_DOWN, ACTION_UP, ACTION_MOVE, ACTION_POINTER_DOWN, ACTION_POINTER_UP, ACTION_CANCEL，每个事件都是以ACTION_DOWN开始ACTION_UP结束。  

(3) 对事件的处理包括三类，分别为传递——dispatchTouchEvent()函数、拦截——onInterceptTouchEvent()函数、消费——onTouchEvent()函数和OnTouchListener  

### 2、传递流程
(1) 事件从Activity.dispatchTouchEvent()开始传递，只要没有被停止或拦截，从最上层的View(ViewGroup)开始一直往下(子View)传递。子View可以通过onTouchEvent()对事件进行处理。   

(2) 事件由父View(ViewGroup)传递给子View，ViewGroup可以通过onInterceptTouchEvent()对事件做拦截，停止其往下传递。   

(3) 如果事件从上往下传递过程中一直没有被停止，且最底层子View没有消费事件，事件会反向往上传递，这时父View(ViewGroup)可以进行消费，如果还是没有被消费的话，最后会到Activity的onTouchEvent()函数。   

(4) 如果View没有对ACTION_DOWN进行消费，之后的其他事件不会传递过来。   

(5) OnTouchListener优先于onTouchEvent()对事件进行消费。   
上面的消费即表示相应函数返回值为true。   

**更多请直接阅读PDF英文原文：[Mastering the Android Touch System](http://wugengxin.cn/download/pdf/android/PRE_andevcon_mastering-the-android-touch-system.pdf)**  

示例代码：[Demo@Github](https://github.com/devunwired/custom-touch-examples) 

附上两张原文中流程图：  
(1) View不处理事件流程图  
![view-ignore-touch-event-example](https://farm6.staticflickr.com/5529/13927155020_73bdfab805_o.jpg)  

(2) View处理事件流程图  
![view-process-touch-event-example](https://farm8.staticflickr.com/7062/14110505861_6569e33985_o.jpg)  