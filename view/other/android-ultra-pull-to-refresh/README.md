android-Ultra-Pull-To-Refresh 源码解析
====================================
> 本文为 [Android 开源项目源码解析](https://github.com/android-cn/android-open-project-analysis) 中 android-Ultra-Pull-To-Refresh 部分  
> 项目地址：[android-Ultra-Pull-To-Refresh](https://github.com/liaohuqiu/android-Ultra-Pull-To-Refresh)，分析的版本：[508c632](https://github.com/liaohuqiu/android-Ultra-Pull-To-Refresh/tree/508c63266de51ad8c010ac9912f7592b2f2da8fc)，Demo 地址：[android-Ultra-Pull-To-Refresh Demo](https://github.com/android-cn/android-open-project-demo/tree/master/android-ultra-pull-to-refresh-demo)    
> 分析者：[Grumoon](https://github.com/grumoon)，校对者：，校对状态：未完成   
 

###1. 功能介绍  
下拉刷新，几乎是每个Android应用都会需要的功能。android-Ultra-Pull-To-Refresh（以下简称 UltraPTR ）便是一个强大的Andriod下拉刷新框架。    
主要特点：  
(1).继承于 ViewGroup，Content 可以包含任何View。  
(2).简洁完善的 Header 抽象，方便进行拓展，构建符合需求的头部。
> 对比 [Android-PullToRefresh](https://github.com/chrisbanes/Android-PullToRefresh) 项目，UltraPTR 没有实现 **加载更多** 的功能，但我认为 **下拉刷新** 和 **加载更多** 不是同一层次的功能， **下拉刷新** 有更广泛的需求，可以适用于任何页面。而 **加载更多** 的功能应该交由具体的 Content 自己去实现。这应该是和Google官方推出 SwipeRefreshLayout 是相同的设计思路，但对比 SwipeRefreshLayout，UltraPTR 更灵活，更容易拓展。


###2. 详细设计
###2.1 类详细介绍
核心类、函数功能介绍及核心功能流程图，流程图可使用 StartUML、Visio 或 Google Drawing。  
###2.2 类关系图
类关系图，类的继承、组合关系图，可是用 StartUML 工具。  

**完成时间**  
- 根据项目大小而定，目前简单根据项目 Java 文件数判断，完成时间大致为：`文件数 * 7 / 10`天，特殊项目具体对待  

###3. 流程图
主要功能流程图  
- 如 Retrofit、Volley 的请求处理流程，Android-Universal-Image-Loader 的图片处理流程图  
- 可使用 StartUML、Visio 或 Google Drawing 等工具完成，其他工具推荐？？  
- 非所有项目必须，不需要的请先在群里反馈  

**完成时间**  
- `两天内`完成  

###4. 总体设计
整个库分为哪些模块及模块之间的调用关系。  
- 如大多数图片缓存会分为 Loader 和 Processer 等模块。  
- 可使用 StartUML、Visio 或 Google Drawing 等工具完成，其他工具推荐？？  
- 非所有项目必须，不需要的请先在群里反馈。  

**完成时间**  
- `两天内`完成  

###5. 杂谈
该项目存在的问题、可优化点及类似功能项目对比等，非所有项目必须。  

**完成时间**  
- `两天内`完成  

###6. 修改完善  
在完成了上面 5 个部分后，移动模块顺序，将  
`2. 详细设计` -> `2.1 核心类功能介绍` -> `2.2 类关系图` -> `3. 流程图` -> `4. 总体设计`  
顺序变为  
`2. 总体设计` -> `3. 流程图` -> `4. 详细设计` -> `4.1 类关系图` -> `4.2 核心类功能介绍`  
并自行校验优化一遍，确认无误后，让`校对 Buddy`进行校对，`校对 Buddy`校对完成后将  
`校对状态：未完成`  
变为：  
`校对状态：已完成`  

**完成时间**  
- `两天内`完成  

**到此便大功告成，恭喜大家^_^**  
