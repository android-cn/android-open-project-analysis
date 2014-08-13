# Dagger——java和Android的高速依赖注入工具

#### 简介

Dagger是一款Java平台的依赖注入库（如果你还不了解依赖注入，务必先看[这篇文章](http:www.baidu.com)）。Java的依赖注入库中，最有名的应该属Google的Guice。Guice的功能非常强大，但它是通过在运行时读取注解来完成依赖的注入的，注解的读取需要依靠Java的反射机制，这对于对运行时性能非常敏感的Android来说是一个硬伤。基于此，Dagger应运而生。Dagger同样使用注解来实现依赖注入，不过由于原理的不同(_Dagger是在编译前读取注解并生成用于在运行时注入依赖的额外代码，因此运行时并不会用到反射_），Dagger对于程序的性能没有丝毫影响，因此更加适用于Android应用的开发。

#### 使用方式

本文将以一个简单的“老板和程序员”App为例。

你想把一个Boss对象注入到一个Activity中，那么首先你要告诉程序一个Boss对象应该怎样被生成。在Boss类的构造方法前添加一个@Inject注解，Dagger就会在需要的时候获取Boss对象的时候，调用这个被标记的构造方法，从而获取一个Boss对象。

```java
public class Boss {
    ...

    @Inject
    public Boss() {
        ...
    }

    ...
}
```

_需要注意的是，如果构造函数含有参数，Dagger会在构造对象的时候先去获取这些参数（不然谁来传参？），所以你要保证它的参数也能被Dagger获取到。（Dagger获取对象的方式有两种：除了注解构造方法之外，也可以通过提供相应的@Provides注解方法来获取，下面会讲到）_

通过@Inject注解了构造方法之后，在Activity中的Boss对象声明之前也添加@Inject注解。这个注解的目的是告诉Dagger哪些对象应该被注入依赖。

```java
public class MainActivity extends Activity {
    @Inject Boss boss;
    ...
}
```

最后，我们在合适的位置（本例是在onCreate()方法中）调用ObjectGraph.inject()方法，Dagger就会自动获取依赖并注入到当前对象（MainActivity）。

```java
public class MainActivity extends Activity {
    @Inject Boss boss;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        ObjectGraph.create(AppModule.class).inject(this);
    }
    ...
}
```

到此为止，使用Dagger将一个Boss对象注入到MainActivity的流程就完成了。上面这段代码中出现了两个类：ObjectGraph和AppModule。其中ObjectGraph是由Dagger提供的类，可以简单理解为一个工具类，它的create函数中参数为所有的Module，本文不详述，如果有兴趣可以跟进我之后的Dagger详解。AppModule是一个自定义类，代码如下：

```java
@Module(injects = MainActivity.class)
public class AppModule {
}
```

可以看到，AppModule是一个空类，只有一行注解。@Module注解表示这个类是一个Module，Module的作用是提供信息，让ObjectGraph知道应该怎样注入所有的依赖。例如，上面这段代码中声明了可注入对象的信息：MainActivity.class（使用显式声明这样的看起来很麻烦、多此一举的方式和Dagger的原理有关，本文不详述）。

#### 自定义依赖

对构造方法进行注解是很好用的实现依赖的途径，然而它并不适用于所有情况。

* 接口（Interface）是没有构造方法的
* 第三方库提供的类，它们的构造方法不能被注解
* 有些类需要灵活选择初始化的配置，而不是使用一个单一的构造方法

对于以上三种情况，可以使用@Provides注解来提供自定义的初始化方法，实现自定义依赖。

```java
@Provides
Coder provideCoder(Boss boss) {
    return new Coder(boss);
}
```

_同样，@Provides注解的方法如果含有参数，它的所有参数也要保证能够被Dagger获取到。_

所有带有@Provides注解的方法都需要被封装到Module中：

```java
@Module
public class AppModule {
    @Provides
    Coder provideCoder(Boss boss) {
        return new Coder(boss);
    }
}
```

#### 单例
Dagger支持单例（事实上单例也是依赖注入最常用的场景），使用方式也很简单：

```java
// @Inject注解构造方法的单例模式
@Singleton
public class Boss {
    ...

    @Inject
    public Boss() {
        ...
    }

    ...
}
```

```java
// @Provides注解提供对象的单例模式
@Provides
@Singleton
Coder provideCoder(Boss boss) {
    return new Coder(boss);
}
```

通过上面的方法添加@Singleton注解之后，对象只会被初始化一次，之后的每次都会被直接注入相同的对象。

#### Qualifier（限定符）

如果有两类程序员，他们的能力值power分别是5和1000，应该怎样让Dagger对他们做出区分呢？使用@Qualifier注解

首先，创建一个@Qualifier注解，用于区分两类程序员：

```java
@Qualifier
@Documented
@Retention(RUNTIME)
public @interface Level {
  String value() default "";
}
```

然后，为这两类程序员分别设置@Provides方法，并使用@Qualifier注解对他们做出不同的标记：

```java
@Provides @Level("low") Coder provideLowLevelCoder() {
    Coder coder = new Coder();
    coder.setName("战五渣");
    coder.setPower(5);
    return coder;
}

@Provides @Level("high") Coder provideHighLevelCoder() {
    Coder coder = new Coder();
    coder.setName("大神");
    coder.setPower(1000);
    return coder;
}
```

最后，在使用的时候也用上相应的@Qualifier注解。

```java
@Inject @Level("low") Coder lowLevelCoder;
@Inject @Level("high") Coder highLevelCoder;
```

#### 编译时检查
实质上，Dagger会在编译时对代码进行检查，并在检查不通过的时候报编译错误（为什么？这和Dagger的原理有关，有兴趣的话可以关注我之后发布的Dagger详解）。检查内容主要有三点：

1. 所有含有依赖注入的类，需要被显式 声明在相应的Module中。
2. 一个Module中所有@Provides方法的参数都必须在这个Module种提供相应的@Provides方法，或者在@Module注解后添加“complete = false”注明这是一个不完整Module（即它会被其他Module所扩展）。
3. 一个Module中所有的@Provides方法都要被它声明的注入对象所使用，或者在@Module注解后添加“library = ture”注明（即它是为了扩展其他Module而存在的）。

_如果需要对Dagger有更多了解，可以参看[官方文档](http://square.github.io/dagger/)，或者关注我之后的详解文章。_

_另外，我们公司[友邻小区](http://hiyoulin.com)招聘[Android开发](http://www.lagou.com/jobs/153948.html)一名，地址北京海淀紫竹桥。_

_我的Github地址：https://github.com/rengwuxian_
