Android 开源项目源码解析
====================================
这是一个协作项目，最终多数开源库原理解析会在 [www.codekk.com](http://www.codekk.com) 分享出来。  
欢迎大家`Star`、`Fork`，让更多的人知道并加入进来！  

这里是稳定版，最新版见：[源码分析开发版](https://github.com/aosp-exchange-group/android-open-project-analysis)  

`codeKK`是我们对外的账号，专注于开源项目源码解析、开源项目分享、Android 职位推荐。  
> 我们的网站：[www.codekk.com](http://codekk.com)  
 我们的微博：[code-kk](http://weibo.com/codek2)  
 我们的微信：codekk，二维码如下：  
 ![img](https://raw.githubusercontent.com/aosp-exchange-group/about/master/weixin-qrcode.jpg) 

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
 
那么加入我们吧！！！  见：[源码分析开发版](https://github.com/aosp-exchange-group/android-open-project-analysis)

#### 我们不重复造轮子不表示我们不需要知道轮子该怎么造及如何更好的造！
