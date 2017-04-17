${项目名} 源码解析
====================================
> 本文为 [Android 开源项目源码解析](http://a.codekk.com) 中 ${项目名} 部分  
 项目地址：[${项目名}](${项目原地址})，分析的版本：[${commitId}.substring(0, 7)](${项目原地址}/commit/${commitId} "Commit id is ${commitId}")，Demo 地址：[${项目名} Demo](https://github.com/android-cn/android-open-project-demo/tree/master/${项目 Demo 地址})    
 分析者：[${分析者}](${分析者 Github 地址})，分析状态：未完成，校对者：[Trinea](https://github.com/trinea)，校对状态：未开始   

建议大家看下 [Volley](../volley/README.md)、[UIL](../universal-image-loader/README.md)、[Dagger](../dagger/README.md)、[Event Bus](../event-bus/README.md) 的分析，了解分析该到什么程度，以及类似流程图和总体设计该怎么做。  

`复制一份到自己的项目文件夹下，然后根据自己项目替换掉 ${} 内容，删掉本行及上面两行。`  

### 1. 功能介绍  
功能介绍，包括功能或优点等  

**完成时间**  
- `一天内`完成  

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
