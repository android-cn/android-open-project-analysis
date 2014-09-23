EventBus 源码解析
----------------
本文为 [Android 开源项目原理解析](https://github.com/android-cn/android-open-project-analysis) 中 EventBus 部分  
项目地址：[Greenrobot EventBus](https://github.com/greenrobot/EventBus)，Demo 地址：[EventBus Demo](https://github.com/android-cn/android-open-project-demo)  
分析者：[Trinea](https://github.com/trinea)&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;校对者：[Trinea](https://github.com/trinea)  

http://www.cnblogs.com/angeldevil/p/3715934.html  
http://www.pocketdigi.com/20131227/1235.html  
http://blog.csdn.net/huangyanan1989/article/details/10858695  
https://github.com/greenrobot/EventBus  
https://github.com/kevintanhongann/EventBusSample  

###1. 功能介绍
EventBus 是一个优化的 Android 发布订阅事件总线。通过解耦发布者和订阅者简化 Android 事件传递，这里的事件既仅包括 Android 四大组件间传递，也包括异步返回的 UI 线程更新等  
传统的事件传递方式包括：Handler、BroadCastReceiver、Interface 回调，相比之下 EventBus 的优点是代码简洁，使用简单  
传统的方式是定义一个统一的 Interface，每个接受者实现这个接口，然后发布者发布信息调用这个统一的接口  
###2. 详细设计
###2.1 核心类功能介绍
###2.2 类关系图
###3. 流程图
###4. 总体设计
###5. 与 Otto 对比