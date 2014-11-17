#### 如何认领
- 开源库认领  
以开源库名为入群理由加入 协作 QQ 群，群号在 Android 开源交流 QQ 群 [63224677](http://shang.qq.com/wpa/qunwpa?idkey=fb2eaf0c4b4a8c838ad15e6bdd69d901f038a50f4a77360845b9e6d7ee0ba3ee "入群理由必须填写群简介问题答案") 公告内  
然后直接在 [android-open-project-analysis](https://github.com/android-cn/android-open-project-analysis) 主页最后修改提交  
- 寻找 Buddy  
在群里找到一个后期帮忙做校验的 Buddy  
- 填写时间计划表  
读完本文档后，根据自己的时间安排，填写[时间计划表](https://github.com/android-cn/android-open-project-analysis/wiki/Schedule)  

**完成时间**  
- `开始当天`完成  

编写步骤如下
---------
#### 一. 编写该开源库的使用示例
到 [android-open-project-demo](https://github.com/android-cn/android-open-project-demo) 项目下新建文件夹，用于后续上传该开源库使用示例工程代码  
- 该文件夹以 “开源库名-demo” 命名，全小写，单词间用`-`连接  
- 示例工程要求覆盖到该开源项目所有功能，不允许拷贝官方 Demo  
- 若没有自己的 Code Format 文件需要统一使用 [common](https://github.com/android-cn/android-open-project-demo/common/) 文件夹下 code format，code template 文件 
- 文件夹下需要有名为 apk 的子文件夹，用于存放可运行 APK 文件  

**完成时间**  
- 示例工程需要在认领后`三天内`完成全部提交，中间每天需要有一定的提交  
- Buddy 在后续`一天内`完成校验，校验内容包括项目的完整运行，APK 包含主要的功能  
  
  
#### 二. 编写原理文档  
到 [android-open-project-analysis](https://github.com/android-cn/android-open-project-analysis) 项目下新建文件夹  
- 该文件夹以 “开源库名” 命名，全小写，单词间用-连接  
- 从 [common](https://github.com/android-cn/android-open-project-analysis/tree/master/common) 文件夹下拷贝模板文件 README.md，复制到上面的文件夹中，以下所有内容在 README.md 中用 [Markdown](https://github.com/android-cn/blog/blob/master/dev-tool/markdown.md) 编写  
- 文件夹下可存放相关图片  
##### 1. 作者介绍
对应开源库版本  
##### 1. 功能介绍  
提交功能介绍，包括功能、优点等  

**完成时间**  
- `一天内`完成  
- Buddy 在后续`一天内`完成校验 

##### 2. 详细设计
包括核心类功能介绍，类关系图  
- 核心类介绍，包括类功能及核心函数功能  
- 类关系图，类的继承及调用关系图，工程太小此步骤可忽略  
- 可使用 StartUML 工具，其他工具推荐？？  

**完成时间**  
- 根据项目大小而定，目前简单根据项目 Java 文件数判断，完成时间大致为：`文件数 * 7 / 10`天，特殊项目具体对待  
`完成时间确定后在 GitBook 填写完成完成时间`  
- 需要 Buddy 用上面 `2/3 的完成时间`完成校验    

##### 3. 流程图
功能流程图  
- 如 Retrofit、Volley 的请求处理流程，Android-Universal-Image-Loader 的图片处理流程图  
- 可使用 Visio 等工具完成，其他工具推荐？？  
- 非所有项目必须，不需要的请先在群里反馈  

**完成时间**  
- `两天内`完成  
- Buddy 在后续`一天内`完成校验  

##### 4. 总体设计
整个库分为哪些模块及模块之间的调用关系  
- 如大多数图片缓存会分为 Loader 和 Processer 等模块  
- 可使用 Visio 等工具完成，其他工具推荐？？  
- 非所有项目必须，不需要的请先在群里反馈  

**完成时间**  
- `两天内`完成  
- Buddy在后续`一天内`完成校验   

##### 5. 杂谈
该项目存在的问题及可优化点等，非所有项目必须

##### 6. 修改完善
按照反顺序，从总体设计 -> 流程图 -> 详细设计 -> 功能介绍 -> 示例工程自行校验优化一遍。  
如此便大功告成^_^
