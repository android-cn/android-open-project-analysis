GreenDao 源码解析
----------------
> 本文为 [Android 开源项目实现原理解析](https://github.com/android-cn/android-open-project-analysis) 中 GreenDao 部分  
> 项目地址：[GreenDao](https://github.com/greenrobot/greenDAO)，分析的版本：[07524fc](https://github.com/greenrobot/greenDAO/commit/07524fc2c45426c184110c2d4c78477c224a7f99 "Commit id is 07524fc2c45426c184110c2d4c78477c224a7f99")，Demo 地址：[GreenDao Demo](https://github.com/android-cn/android-open-project-demo/tree/master/greendao-demo)    
> 分析者：[maogy](https://github.com/maogy)，校对者：[Caij](https://github.com/Caij)，校对状态：未完成   

###1. 概述
####1.0 先说下我写这些文字的思路
greenDao开源项目的所有资料在官网上都有描述，而且是最权威的，所以本文有些内容是引用官网翻译过来，官网地址：[greenDao官网](http://greendao-orm.com/)

####1.1 greenDao  
GreenDao帮助android开发者处理数据，将数据存储到sqlite.sqlite是一个极好的嵌入式关系数据库，但基于它开发需要做许多附加的工作.写sql语句和解析查询结果是体力活.greenDao将这些工作为你做完：将java对象映射成数据库表（我们常说的ORM).这样我们可以用简单、面向对象的api来存储、更新、删除和查询java对象.为我们节省时间，将重点放在解决问题上。

###1.2 greenDao设计目标
性能最大化（可能是android上最快的orm库）
易于使用的api
对android高度优化
最小的内存使用
库大小较小，提供必需的功能

###2.功能
####2.1 ORM（对象关系映射）
greenDao的本质是提供一个面向对象的接口来存储数据到sqlite中.你只需定义数据模型，greenDao会创建java数据对象（entities）和DAOs（数据访问对象）.这样会让你少写许多仅仅是将数据块移来移去的厌烦代码.此外greenDao提供一些高级ORM特性，像session缓存、预先加载、活跃的实体

####2.2 性能
greenDao在性能方面严格把关.数据库很适合存储大量数据，因此速度很关键.使用greenDao，大量的数据可以在每秒几千条的速度下插入、更新和加载.
和ORMLite比较，同样数量的数据实体，greenDao插入和更新实体的速度是ORMLite的2倍，加载操作要快4.5倍.对典型的应用，加载速度是最重要的.下图是官方提供的，时间：10-23-2011
![compare img](image/greenDAO-performance.png)

考虑到greenDao内核一些特性，如session缓存和智能预加载技术，也提升了库的性能

####2.3 较小的库
greenDao的核心库小于100k，因此将greenDao加入到工程不会增加太大APK大小.

####2.4 活跃的数据实体
你可以配置实体（entity）是活跃的，这样他

####2.5 支持协议缓存（protocol buffers），如google protocol buffer
我理解是有格式的byte数组，greenDao支持类似GPB对象直接存入数据库.如果你和服务器通过GPB传递消息，你不需要另外的映射.所有的增删改查操作都支持GPB对象.这是greenDao的一个特有功能点.

####2.6代码生成
greenDao会生成java数据对象（entity）和DAO对象，DAO对象和数据对象一一对应.

####2.7源码开放
源码放在github上，源码另外也包含JUnit的测试用例，这些用例使用了greenDao的所有功能，因此它是一个很好学习greenDao的方式