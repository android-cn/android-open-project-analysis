${项目名} 实现原理解析
====================================
> 本文为 [Android 开源项目实现原理解析](https://github.com/android-cn/android-open-project-analysis) 中 HoloGraphLibrary部分  
> 项目地址：[HoloGraphLibrary](https://github.com/Androguide/HoloGraphLibrary)，分析的版本：[ccc2771](https://github.com/Androguide/HoloGraphLibrary/commit/028cd2ae6916308bbb96472aafa9ecd8b1343d5c"Commit id is 28cd2ae6916308bbb96472aafa9ecd8b1343d5c")，Demo 地址：[HoloGraphLibrary Demo](https://github.com/android-cn/android-open-project-demo/tree/master/holo-graph-library-demo)    
> 分析者：[AaronPlay](https://github.com/AaronPlay)，校对者：[${校对者}](${校对者 Github 地址})，校对状态：未完成   

`复制一份到自己的项目文件夹下，然后根据自己项目替换掉 ${} 内容，删掉本句内容。`  

###1. 功能介绍  
功能介绍，包括功能或优点等  
HoloGraphLibrary是一个封装了折线图、柱状图以及饼状图等基本的制表图。在柱状图还有一个功能，可以使原来的条形数据发生叠加。

优点：图形设计友好，使用方便。

###2. 详细设计
###2.1 核心类功能介绍
核心类、函数功能介绍及核心功能流程图，流程图可使用 StartUML、Visio 或 Google Drawing。  
####2.1.1折线图：
LinePoint.java：折线的最基本元素，属性包括二维坐标，路径以及区域等属性。

Line.java : 由点构成线，里面封装了一个包含LinePoint的数组。

LineGraph.java: 继承View，负责折线图的绘制。
- onTouchEvent()：可对折线图的点击触发进行针对性修改，然后重新绘制。

####2.1.2饼状图
PieSlice.java:构成饼状图的基本元素——块。封装了颜色，值，标题，路径以及区域等属性。

PieGraph.java:继承View, 负责绘制饼状图。
- onTouchEvent()：可对饼状图的点击触发进行针对性修改，然后重新绘制。

####2.1.3柱状图：
BarStackSegment.java: 柱体的堆片段，用于后续的操作引起的变化。

Bar.java:构成柱状图的基本元素。封装了颜色，名字，等属性。

BarGraph.java:继承View，负责柱状图的绘制。
- onTouchEvent()：可对柱状图的点击触发进行针对性修改，然后重新绘制。

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
并自行校验优化一遍，确认无误后，让`校对 Buddy`进行校对，`校对 Buddy`校队完成后将  
`校对状态：未完成`  
变为：  
`校对状态：已完成`  

**完成时间**  
- `两天内`完成  

**到此便大功告成，恭喜大家^_^**  
