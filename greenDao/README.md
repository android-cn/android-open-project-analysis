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

###3.使用greenDao
这一小节带你熟悉一个简单的greenDao示例工程，它就是github上的那个.它由两个子工程组成:DaoExample和DaoExampleGenerator.
DaoExample工程check下来后，可以运行到android设备上，如你看到的，是记录笔记的简单应用.你可以编辑一些文本生成一条记录，通过点击一条记录，来删除它.
####3.1事先生成代码和创建表单
来看一下DaoExample里面的代码，在目录src-gen中，你可以找到一些生成的代码
1）Note.java是java类，他包含note所有的数据
2）NoteDao.java是DAO（数据访问对象）类，提供操作Note对象的接口

你可以使用DaoExampleGenerator工程来生成Note和NoteDao.来看DaoExample，使用DaoMaster类，你可以方便的获取SQLiteOpenHelper：
```
new DaoMaster.DevOpenHelper(this, "notes-db", null)
```
这样，你不需要编写"CREATE TABLE"的SQL语句.greenDao已经做了这些工作.

####3.2插入和删除notes
从上面代码，我们已经获得一个notes的数据库表，我们可以插入一些notes到数据库.这在NoteActivity中有相关代码.在onCreate中我们创建一个DAO对象
```
daoMaster = new DaoMaster(db);
daoSession = daoMaster.newSession();
noteDao = daoSession.getNoteDao();
```
再来看下addNote方法，看如何插入一条note到数据库中
```
Note note = new Note(null, noteText, comment, new Date());
noteDao.insert(note);
Log.d("DaoExample", "Inserted new note, ID: " + note.getId());
```
创建一个java对象，调用DAO的insert方法.当insert方法返回，刚插入记录的数据库id会赋值给note对象，如log的打印记录可以验证.
删除一条note也很简单；看一下onListItemClick 方法:
```
noteDao.deleteByKey(id);
```
你可以看其他的DAO类中的方法，像loadAll和update

####3.3数据模型和代码生成
如果你想扩展note对象或者创建新的实体，你需要看DaoExampleGenerator工程，里面只有一个类，其中包含定义数据模型的代码:
```
Schema schema = new Schema(1, "de.greenrobot.daoexample");

Entity note= schema.addEntity("Note");
note.addIdProperty();
note.addStringProperty("text").notNull();
note.addStringProperty("comment");
note.addDateProperty("date");

new DaoGenerator().generateAll(schema, "../DaoExample/src-gen");
```
可以看到，你创建了一个Schema对象，通过他来添加实体（entity），一个实体类对应数据库的一个表.实体包含属性，一个属性对应数据库表的一列.
完成schema的定义后，你可以触发生成代码.这样可以生成类似Note.java和NoteDao.java的文件.

###4.介绍
![introduce img](image/introduce.png)
greenDao是android上的一个对象/关系映射(ORM)工具，它提供面向对象的接口来使用关系型数据库sqlite.类似greenDao的ORM工具为你做完了许多重复的工作（原本是你来写这些重复代码），并为你的数据提供简单的接口.

####4.1DAO相关类的生成工程
![code-generation-project img](image/generator.png)
为了在你的android工程里面使用greenDao，你需要创建第二个工程，“代码生成”工程，它的任务是生成你工程对应的数据库操作类.这个代码生成工程，是一个java工程（不是andorid工程）.确保greenDAO-generator.jar和Freemarker两个jar包已经导入到工程中.创建一个可运行的java类，定义你的实体类，运行生成代码.

####4.2生成的核心类
代码生成完成后，你可以在你android工程中使用greenDao.不要忘了包含greenDao的jar包(greenDao.jar).
![core-class img](image/core-class.png)

**DaoMaster**:使用greenDao的入口点.DaoMaster保存了一个数据库对象（SQLiteDatabase）并管理实体对应的DAO类(不是对象).它提供静态方法来创建或删除表.它的内部类OpenHelper和DevOpenHelper继承了SQLiteOpenHelper，它们创建数据库表.

**DaoSession**:管理所有的实体相关的DAO对象，你可以通过getter方法获取DAO对象.它也提供通用的接口insert、update、refresh和delete来操作entity.最后，DaoSession对象和标识范围（identity scope）保持联系