编写步骤如下
---------
####一. 准备
- 寻找 Buddy  
在群里找到一个后期帮忙做校验的 Buddy  
- 填写时间计划表  
读完本文档后，根据自己的时间安排，填写[时间计划表](https://github.com/android-cn/android-open-project-analysis/blob/master/schedule.md)  

####二. 编写该开源库的使用示例
到 [android-open-project-demo](https://github.com/android-cn/android-open-project-demo) 项目下新建文件夹，用于后续上传该开源库使用示例工程代码。  
- 该文件夹以`开源库名-demo`命名，全小写，单词间用`-`连接；  
- 示例工程要求覆盖到该开源项目所有功能，不允许拷贝官方 Demo；  
- 若没有自己的 Code Format 文件，使用 [common](https://github.com/android-cn/android-open-project-demo/tree/master/common) 文件夹下 code format，code template 文件；  
- 文件夹下需要有名为 apk 的子文件夹，用于存放可运行 APK 文件；  
- 文件夹下需要有名为 README.md 的介绍文件，其中包含 demo 下载方式。  

**完成时间**  
- 示例工程需要在认领后`三天内`完成全部提交，中间每天需要有一定的提交。  
  
####三. 编写原理文档  
到 [android-open-project-analysis](https://github.com/android-cn/android-open-project-analysis) 项目下新建文件夹。  
- 该文件夹以`开源库名`命名，全小写，单词间用-连接；  
- 从 [common](https://github.com/android-cn/android-open-project-analysis/tree/master/common) 文件夹下拷贝模板文件 README.md.md，复制到上面的文件夹中，所有内容在 README.md 中用 [Markdown](https://github.com/android-cn/blog/blob/master/dev-tool/markdown.md) 编写。如果内容较多可分多个 md 文件，但 README.md 为总体的概括文件；  
- 文件夹下可有名为`image`子文件夹存放相关流程图等图片，`uml`子文件夹存放 uml 文件。  

####四、文档包含的模块及预计时间
具体编写的模块及各模块预计耗时请参考：[Common README](https://github.com/android-cn/android-open-project-analysis/blob/master/common/README.md)。  

####五、Buddy Check
Buddy 可在最终完成后开始 Check，包括 demo apk 功能是否完善以及原理文档是否清晰。  
