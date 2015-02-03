Android 开源项目源码解析
====================================
这是一个协作项目，最终多数开源库原理解析会在这里分享出来。  
欢迎大家`Star`、`Fork`，让更多的人知道并加入进来！  
## 一、开源项目源码分析第一期成果
**在线网站：[www.codekk.com](http://codekk.com/open-source-project-analysis)**  

分析文档 | 作者 
:------------- | :------------- 
[Volley 源码解析](http://codekk.com/open-source-project-analysis/detail/Android/grumoon/Volley%20%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90) | [grumoon](https://github.com/grumoon)
[Universal Image Loader 源码分析](http://codekk.com/open-source-project-analysis/detail/Android/huxian99/Android%20Universal%20Image%20Loader%20%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90) | [huxian99](https://github.com/huxian99)
[Dagger 源码解析](http://codekk.com/open-source-project-analysis/detail/Android/%E6%89%94%E7%89%A9%E7%BA%BF/Dagger%20%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90) | [扔物线](https://github.com/rengwuxian)
[EventBus 源码解析](http://codekk.com/open-source-project-analysis/detail/Android/Trinea/EventBus%20%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90) | [Trinea](https://github.com/Trinea)
[xUtils 源码解析](http://codekk.com/open-source-project-analysis/detail/Android/Caij/xUtils%20%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90) | [Caij](https://github.com/Caij)
[ViewPagerindicator 源码解析](http://codekk.com/open-source-project-analysis/detail/Android/lightSky/ViewPagerindicator%20%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90) | [lightSky](https://github.com/lightSky)
[HoloGraphLibrary 源码解析](http://codekk.com/open-source-project-analysis/detail/Android/AaronPlay/HoloGraphLibrary%20%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90) | [aaronplay](https://github.com/AaronPlay)
[CircularFloatingActionMenu 源码解析](http://codekk.com/open-source-project-analysis/detail/Android/cpacm/CircularFloatingActionMenu%20%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90) | [cpacm](https://github.com/cpacm)
[PhotoView 源码解析](http://codekk.com/open-source-project-analysis/detail/Android/dkmeteor/PhotoView%20%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90) | [dkmeteor](https://github.com/dkmeteor)
[Lock Pattern 源码解析](http://codekk.com/open-source-project-analysis/detail/Android/%E7%88%B1%E6%97%A9%E8%B5%B7/Android%20Lock%20Pattern%20%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90) | [爱早起](https://github.com/liang7)
[公共技术点之 Java 动态代理](http://codekk.com/open-source-project-analysis/detail/Android/Caij/%E5%85%AC%E5%85%B1%E6%8A%80%E6%9C%AF%E7%82%B9%E4%B9%8BJava%20%E5%8A%A8%E6%80%81%E4%BB%A3%E7%90%86) | [Caij](https://github.com/Caij)
[公共技术点之 View 绘制流程](http://codekk.com/open-source-project-analysis/detail/Android/lightSky/%E5%85%AC%E5%85%B1%E6%8A%80%E6%9C%AF%E7%82%B9%E4%B9%8BView%20%E7%BB%98%E5%88%B6%E6%B5%81%E7%A8%8B) | [lightSky](https://github.com/lightSky)
[公共技术点之 Java 注解 Annotation](http://codekk.com/open-source-project-analysis/detail/Android/Trinea/%E5%85%AC%E5%85%B1%E6%8A%80%E6%9C%AF%E7%82%B9%E4%B9%8BJava%20%E6%B3%A8%E8%A7%A3%20Annotation) | [Trinea](https://github.com/Trinea)
[公共技术点之依赖注入](http://codekk.com/open-source-project-analysis/detail/Android/%E6%89%94%E7%89%A9%E7%BA%BF/%E5%85%AC%E5%85%B1%E6%8A%80%E6%9C%AF%E7%82%B9%E4%B9%8B%E4%BE%9D%E8%B5%96%E6%B3%A8%E5%85%A5) | [扔物线](https://github.com/rengwuxian)
[公共技术点之 View 事件传递](http://codekk.com/open-source-project-analysis/detail/Android/Trinea/%E5%85%AC%E5%85%B1%E6%8A%80%E6%9C%AF%E7%82%B9%E4%B9%8BView%20%E4%BA%8B%E4%BB%B6%E4%BC%A0%E9%80%92) | [Trinea](https://github.com/Trinea)

## 二、加入我们
- 如果你在用各种开源库  
- 如果你有兴趣去深挖它们优雅的实现  
- 如果你希望知其然知其所以然提高自己  
 
那么加入我们吧！！！  
> 欢迎加入 QQ 交流群：[383537512(入群理由必须填写群简介问题答案)](http://shang.qq.com/wpa/qunwpa?idkey=69b7c4278fc3a33690d4847ed7f9a72b9e4feb51221265a7326cf5261ccd5862 "入群理由必须填写群简介问题答案")(二群有空位)  
> [377723625](http://shang.qq.com/wpa/qunwpa?idkey=12ba39b0c3f5d27620ab0cb63ff80507a8a30fd743a11fad028e7742a871e0dc "入群理由必须填写群简介问题答案")(一群已满) [63224677](http://shang.qq.com/wpa/qunwpa?idkey=fb2eaf0c4b4a8c838ad15e6bdd69d901f038a50f4a77360845b9e6d7ee0ba3ee "入群理由必须填写群简介问题答案")(三群已满) [148844489](http://shang.qq.com/wpa/qunwpa?idkey=5dc2f22b2f9fe3b6136f9cad29399713b118bfaa9a2330e410757362a37572bc "入群理由必须填写群简介问题答案")(四群已满) [214742675](http://jq.qq.com/?_wv=1027&k=Zl6Yyj "入群理由必须填写群简介问题答案")(五群已满) [185715999](http://jq.qq.com/?_wv=1027&k=fJlrh1 "入群理由必须填写群简介问题答案")(六群已满) 请不要重复加群  

#### 我们不重复造轮子不表示我们不需要知道轮子该怎么造及如何更好的造！ 

## 三、具体计划
请先读完下面步骤  
###1. 加入开源交流 QQ 群
见上面几行，入群理由必须为群简介问题答案。  

###2. 开始认领
问群主`协作 QQ 群`群号，以自己`有兴趣分析的开源库库名`申请加入`协作 QQ 群`。  

###3. 寻找 Buddy  
加入`协作 QQ 群`后，在群里找到一个后期帮忙做校验的 Buddy  

###4. 开始
了解编写步骤：[编写步骤](https://github.com/android-cn/android-open-project-analysis/blob/master/procedure.md)  
填写自己的时间安排：[时间计划表](https://github.com/android-cn/android-open-project-analysis/blob/master/schedule.md)
