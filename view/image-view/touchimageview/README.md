TouchImageView 源码解析
====================================
> 本文为 [Android 开源项目源码解析](http://a.codekk.com) 中 TouchImageView 部分  
 项目地址：[TouchImageView](https://github.com/MikeOrtiz/TouchImageView)，分析的版本：[6dbeac4](https://github.com/MikeOrtiz/TouchImageView/commit/6dbeac4f11936185ba374c73144ac431c23c9aab "Commit id is 6dbeac4f11936185ba374c73144ac431c23c9aab")，Demo 地址：[TouchImageView Demo](https://github.com/aosp-exchange-group/android-open-project-demo/tree/master/touchimageview-demo)    
 分析者：[truistic](https://github.com/truistic)，分析状态：未完成，校对者：[Trinea](https://github.com/trinea)，校对状态：未开始   


### 1. 功能介绍  
TouchImageView 是基于 ImageView 实现的，支持图片缩放的 ImageView 控件。其主要功能如下：
* 双指缩放
* 双击放大/还原
* 图片拖拽、平移
* 图片缩放到最大/最小边界时，有回弹动画（体验较好）
* 支持和 ViewPager 一起使用
* 支持和 ImageLoader/Picasso 等图片加载和缓存库一起使用（Glide 不支持）

### 2. 详细设计
### 2.1 类详细介绍
类及其主要函数功能介绍、核心功能流程图，流程图可使用 [Google Drawing](https://docs.google.com/drawings)、[Visio](http://products.office.com/en-us/visio/flowchart-software)、[StarUML](http://staruml.io/)。  
### 2.2 类关系图
类关系图，类的继承、组合关系图，可是用 [StarUML](http://staruml.io/) 工具。  

**完成时间**  
- 根据项目大小而定，目前简单根据项目 Java 文件数判断，完成时间大致为：`文件数 * 7 / 10`天，特殊项目具体对待  

### 3. 流程图
主要功能流程图  
- 如 Retrofit、Volley 的请求处理流程，Android-Universal-Image-Loader 的图片处理流程图  
- 可使用 [Google Drawing](https://docs.google.com/drawings)、[Visio](http://products.office.com/en-us/visio/flowchart-software)、[StarUML](http://staruml.io/) 等工具完成，其他工具推荐？？  
- 非所有项目必须，不需要的请先在群里反馈  

**完成时间**  
- `两天内`完成  

### 4. 总体设计
整个库分为哪些模块及模块之间的调用关系。  
- 如大多数图片缓存会分为 Loader 和 Processer 等模块。  
- 可使用 [Google Drawing](https://docs.google.com/drawings)、[Visio](http://products.office.com/en-us/visio/flowchart-software)、[StarUML](http://staruml.io/) 等工具完成，其他工具推荐？？  
- 非所有项目必须，不需要的请先在群里反馈。  

**完成时间**  
- `两天内`完成  

### 5. 杂谈
该项目存在的问题、可优化点及类似功能项目对比等，非所有项目必须。  

**完成时间**  
- `两天内`完成  

### 6. 修改完善  
在完成了上面 5 个部分后，移动模块顺序，将  
`2. 详细设计` -> `2.1 核心类功能介绍` -> `2.2 类关系图` -> `3. 流程图` -> `4. 总体设计`  
顺序变为  
`2. 总体设计` -> `3. 流程图` -> `4. 详细设计` -> `4.1 类关系图` -> `4.2 核心类功能介绍`  
并自行校验优化一遍，确认无误后将文章开头的  
`分析状态：未完成`  
变为：  
`分析状态：已完成`  

本期校对会由专门的`Buddy`完成，可能会对分析文档进行一些修改，请大家理解。  

**完成时间**  
- `两天内`完成  

**到此便大功告成，恭喜大家^_^**  
