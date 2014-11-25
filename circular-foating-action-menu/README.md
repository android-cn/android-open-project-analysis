${项目名} 实现原理解析
====================================
> 本文为 [Android 开源项目实现原理解析](https://github.com/android-cn/android-open-project-analysis) 中 circular-foating-action-menu 部分  
> 项目地址：[CircularFloatingActionMenu](https://github.com/oguzbilgener/CircularFloatingActionMenu)，分析的版本：[e9ccdad](https://github.com/android-cn/android-open-project-demo/commit/1306e632d5a7734cd8451f4e10dff763f9ab4097)，Demo 地址：[circular-foating-action-menu](https://github.com/android-cn/android-open-project-demo/tree/master/CircularFloatingActionMenu-demo)    
> 分析者：[cpacm](https://github.com/cpacm)，校对者：[${校对者}](${校对者 Github 地址})，校对状态：未完成   

###1. 功能介绍  
“一个灵感来自Path路径的Android上可定制圆形浮动菜单动画”
一个可定制的圆形浮动菜单，我们可以往里面添加子项使其充当一个可缩放的菜单按钮。
##1.1 可定制
可以根据提供的基础库自己编辑动画类，实现自制的动画效果。
##1.2 多方面
不止是在ImageView上可实现，大部分的view都可以充当菜单按钮从而呼出子菜单



**完成时间**  
- `一天内`完成  

###2. 详细设计
###2.1 核心类功能介绍
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
并自行校验优化一遍，确认无误后，让`校对 Buddy`进行校对，`校对 Buddy`校队完成后将  
`校对状态：未完成`  
变为：  
`校对状态：已完成`  

**完成时间**  
- `两天内`完成  

**到此便大功告成，恭喜大家^_^**  
