XUtils 实现原理解析
====================================
> 本文为 [Android 开源项目实现原理解析](https://github.com/android-cn/android-open-project-analysis) 中 XUtils 部分，  [Demo地址](https://github.com/android-cn/android-open-project-demo)  
> 分析者：[Caij](https://github.com/Caij)，校对者：[maogy](https://github.com/maogy)，校对状态：未完成   


###1. 功能介绍  
xUtils一个Android公共库框架，主要包括四个部分：View，Db, Http, Bitmap 四个模块。
- View模块主要的功能是通过注解绑定UI，资源，事件。减少代码冗余。
- Db模块是一个数据库orm框架， 简单的语句就能进行数据的操作。
- Http模块主要访问网络，支持同步，异步方式的请求，支持文件的下载，上传。
- Bitmap模块是加载图片以及图片的处理， 支持加载本地，网络图片。而且支持图片的内存和本地缓存。

###2. 详细设计
####2.1 核心类功能介绍
#####2.1.1 View模块
注解和反射知识是这个模块的主要内容
- ViewUtils，其实主要功能就是通过反射和注解将Ui和资源、事件和资源绑定。
- EventListenerManager view和事件方法的绑定， 其中的设计是通过动态代理。

#####2.1.2 Db模块
注解、反射和数据库操作知识这个模块的主要内容
- DbUtils，主要功能数据库的创建，数据库的增删改查。
- SqlInfoBuilder， sql语句的组合。
- Selector，WhereBuilder， sql条件语句的组合。

#####2.1.3 Http模块
Handler异步通信，Http网络请求， IO流。
- HttpUtils，支持异步同步访问网络数据， 断点下载文件和上传文件。
- HttpHandler，获取网络数据逻辑的实现。
- InternalHandler 实现线程的通信

#####2.1.4 Bitmap模块  
- BitmapUtils，图片的异步加载，支持本地和网络图片， 图片的压缩处理， 图片的内存缓存已经本地缓存。
- BitmapLoadTask， 加载图片的异步任务。
- BitmapCache， 图片的下载， 缓存， 压缩。
- BitmapGlobalConfig， 配置， 包括线程池， 缓存的大小。

####2.2 类关系图
#####2.2.1 View模块
 ![View类图](image/ViewClass.png)
 
#####2.2.2 Db模块
类模快和关系层次较少， 所以不绘制类图

#####2.2.3Http模块
 ![Http类图](image/HttpClass.png)
 
#####2.2.4Bitmap模块
 ![Bitmap类图](image/BitmapClass.png)
 
###3. 流程图
主要功能流程图  
####3.1 View模块
![View时序图](image/ViewSequence.png)
- 主要的顺序就是在ViewUtils的`inject(View)`将需要的绑定数据的对象传入，`injectObject(Object, ViewFinder)` 主要通过反射获取对象的成员变量和方法， 
然后获取成员变量和方法的注解的值， 将成员变量赋值， 事件和方法绑定， 在EventListenerManager中是通过代理将事件和方法绑定。

####3.2 DB模块
![Db流程图](image/DbSequence.png)
- `DbUtils`中`getInstance()`获取XUtils的实例，里面的操作就是检查数据库版本和升级，然后就是创建数据库（单例模式， 如果存在数据库不会重复创建）。
 `createDatabase()`通过配置创建数据库。save，find，update，delete 都是然后通过`SqlInfoBuilder`或者Selector组合对象的sql语句， 然后通过系统自带数据库api进行数据库操作。
 `SqlInfoBuilder`的原理也是反射加注解。
 
####3.3 Http模块
![Http流程图](image/HttpSequence.png)
- 1.HttpUtils通过send或者down获取网络请求。
- 2.HttpHandler异步任务读取数据，doInBackground()中访问网络， 开始的时候调用publishProgress()，
sendRequest()，handleResponse()将网络数据包装入ResponseInfo，
updateProgress()是在DownloadHandler数据读写时候的回调， 次方法又调用publishProgress()，
publishProgress()通过Handler回调onProgressUpdate() ,
onProgressUpdate()调用RequestCallback，完成回调流程。
- 3.DownloadHandler， handleEntity()将网络数据转化为需要的数据格式。 在读写数据的时候会回调HttpHandler的updateProgress(), 如果当用户选择停止的时候直接停止数据读写。

####3.4 Bitmap模块
![Bitmap流程图](image/BitmapSequence.png)
- 1.BitmapUtils，display。
- 2.BitmapGlobalConfig 获取缓存。 如果图片在运行内存缓存中存在， 就直接回调DefaultBitmapLoadCallBack。
- 3.如果图片在运行内存缓存中不存在， 则开启异步任务BitmapLoadTask， 在doInBackground中优先从sd缓存中读取， 再从网络读取。
- 4.下载的过程在BitmapCache中，  下载完优先存入sd缓存， 再加入运行内存缓存。
- 5. 其中机制和http模块类似，有些细节可以看demo里面的源码， 很多都写了注释。

###4. 总体设计
 
- 1.View和Bb模块 主要是以反射加注解为主。 
- 2.Http和Bitmap模块的构架。

![整体构建思路](image/design.png)

 其实一些优秀的开源框架原理差不多。

###5. 杂谈
主要和Volley框架相比
####相同点：
- 1.都采用了缓存机制。  
- 2.都是通过handler进行线程通信
- 3.Bitmap 模块都采用运行内存缓存， 本地缓存， 图片的压缩处理。 

####不同点：
- 1. Volley的Http请求在 android 2.3 版本之前是通过HttpClient ，在之后的版本是通过URLHttpConnection。xUtils都是通过HttpClient请求网络（bitmap模块图片下载是通过URLHttpConnection）。 在2.3以后URLHttpConnection也很稳定， 扩展和维护性好， 速度也快， 推荐采用URLHttpConnection。
- 2.Volley在Http请求数据下载完成后是先缓存进byte[]， 然后是分配给不同的请求自己转化为自己需要的格式。xUtils是直接转化为想要的格式。 觉得各有优劣， Volley这样做的扩展性比较好， 但是不能存在大数据请求，否则就OOM。xUtils不缓存入byte[] 就支持大数据的请求， 速度比Volley稍快，但扩展性就低。
- 3.Volley最终是将网络请求的数据缓存进sd卡文件， xUtils是缓存在运行内存中。 如果频繁访问相同的网络地址， xUtils比Volley更快。
- 4.Volley访问网络数据时直接开启固定个数线程访问网络， 在run方法中执行死循环， 阻塞等待请求队列。 xUtils是开启线程池来管理线程。Volley请求数据更快，消耗资源更大，xUtils反之。
  

###6. 修改完善  
在完成了上面 5 个部分后，移动模块顺序，将  
`2. 详细设计` -> `2.1 核心类功能介绍` -> `2.2 类关系图` -> `3. 流程图` -> `4. 总体设计`  
顺序变为  
`2. 总体设计` -> `3. 流程图` -> `4. 详细设计` -> `4.1 类关系图` -> `4.2 核心类功能介绍`  
并自行校验优化一遍，确认无误后，让`校对 Buddy`进行校对，`校对 Buddy`校队完成后将  
`校对状态：未完成`  
变为：  
`校对状态：已完成`  

不足： 作为新手， 感觉还是没有站在更高的角度去分析， 组织能力也不是太好， 如果觉得有不对的可以联系我， 大家一起交流交流 ， QQ：`271945881` 。 也可以联系`Trinea`。 谢谢！
