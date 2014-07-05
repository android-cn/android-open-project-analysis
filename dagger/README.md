# Dagger

#### 简介

在开发程序的时候，会用到各种对象，很多对象在使用之前都需要进行初始化。例如你要操作一个SharedPreference，你需要调用getSharedPreferences(String name,int mode)来获取一个对象，然后才能使用它。而如果这个对象会在多个Activity中被使用，你就需要在每个使用的场景中都写下同样的代码。这不仅麻烦，而且增加了出错的可能。dagger的用途就是：让你**不需要初始化对象。**换句话说，任何对象声明完了就能直接用。

#### 原理

dagger是使用**依赖注入**的方式，使用Annotation给需要注入的对象做标记，通过inject()方法自动注入所有对象，从而完成自动的初始化。
示例代码：

```java
public class MainActivity extends Activity {
    // 通过@Inject对对象进行标记
    @Inject SharedPreferences sharedPreferences;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        // 注入依赖
        ObjectGraph.create(AppModule.class).inject(this);

        // 获取name的值并输出
        System.out.println(sharedPreferences.getString("name", ""));
    }
}
```

_依赖注入(Dependency Injection)：在类A中要用到一个B的对象（依赖），需要通过新建B的实例或其他一些**主动**的方式来获取对象，然后才能调用。而通过外部的方式自动将B的对象分配给A（注入），实现**相对被动**的获取对象，这个过程称为**依赖注入**。希望更多了解依赖注入可以自行Google。_

#### 使用方式

以一个简单的“老板和程序员”App为例。你想实现Boss对象的自动注入，那么首先你要告诉程序它要怎么初始化一个Boss。在dagger中，为Boss类的构造方法添加一个@Inject注解，程序就会在需要的时候找到这个被标记的构造方法并调用它，从而获取一个Boss对象。

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

_需要注意的是，如果构造函数含有参数，Dagger会在构造对象的时候先去获取这些参数（不然谁来传参？），所以你要保证这些参数的构造方法也有@Inject标记，或者能够通过@Provides注解（下面会介绍）来获取到。_

然后，在声明Boss对象的时候，在前面同样添加@Inject注解。程序会在依赖注入的过程中自动初始化被注解的对象。

```java
public class MainActivity extends Activity {
    @Inject Boss boss;
    ...
}
```

最后，创建ObjectGraph类并执行inject()方法并将当前MainActivity作为参数传入，Boss的对象就被注入到了MainActivity中。

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

到此为止，使用Dagger将一个Boss对象注入到MainActivity的流程就完成了。上面这段代码中出现了两个雷：ObjectGraph和AppModule。其中ObjectGraph是由Dagger提供的类，可以简单理解为将一个工具类，它的create函数中参数为所有的Module，本文不详述，如果有兴趣可以跟进我之后的Dagger详解。AppModule是一个自定义类，代码如下：

```java
@Module(injects = MainActivity.class)
public class AppModule {
}
```

可以看到，AppModule是一个空类，只有一行注解。@Module注解表示，这个类是一个Module，Module的作用是提供信息，让ObjectGraph知道应该怎样注入所有的依赖。例如，上面这段代码中声明了可注入对象的信息：MainActivity.class（使用显示声明这样看起来很麻烦、多此一举的方式和Dagger的原理有关，本文不详述）。

#### 自定义依赖

对构造方法进行注解是很好用的实现依赖的途径，然而它并不适用于所有情况。

* 接口（Interface）是没有构造方法的
* 第三方库提供的类，它们的构造方法不能被注解
* 有些类需要灵活选择初始化的配置，而不是使用一个单一的构造方法

对于这样的情况，可以使用@Provides注解来提供专用的初始化方法，实现自定义依赖。

```java
@Provides
Coder provideCoder(Boss boss) {
    return new Coder(boss);
}
```

_同样，@Provides注解的方法如果含有参数，它的所有参数也要保证能够被Dagger获取到。_

所有带有@Provides注解的方法都需要被封装到带有@Module注解的类中。

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
Dagger支持单例，方式也十分简单：

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
// @Provides注解提供初始化方法的单例模式
@Singleton
@Provides
Coder provideCoder(Boss boss) {
    return new Coder(boss);
}
```

通过上面的方法添加@Singleton注解之后，对象只会被初始化一次，之后的每次都会被直接注入相同的对象。

#### Qualifier（限定符）

如果有两类程序员，他们的能力值power分别是5和1000，应该怎样让Dagger对他们做出区分呢？使用@Qualifier注解

首先，创建一个@interface：

```java
@Qualifier
@Documented
@Retention(RUNTIME)
public @interface Level {
  String value() default "";
}
```

然后，为这两类程序员分别设置@Provides方法，并使用@Qualifier对他们做出不同的标记：

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

1. 所有含有依赖注入的类，需要被显示声明在相应的Module中。
2. 一个Module中所有@Provides方法的参数都必须在这个Module种提供相应的@Provides方法，或者在@Module注解后添加“complete = false”注明这是一个不完整Module（即它会被其他Module所扩展）。
3. 一个Module中所有的@Provides方法都要被它声明的注入对象所使用，或者在@Module注解后添加“library = ture”注明（即它是为了扩展其他Module而存在的）。

_如果需要对Dagger有更多了解，可以参看[官方文档](http://square.github.io/dagger/)，或者关注我之后的详解文章。_

_我的Github地址：https://github.com/rengwuxian_
