公共技术点之 Java 注解 Annotation
----------------
> 本文为 [Android 开源项目源码解析](http://a.codekk.com) 公共技术点中的 注解 部分  
 分析者：[Trinea](https://github.com/Trinea)，校对者：[Trinea](https://github.com/Trinea)，校对状态：完成  

不少开源库都用到了注解的方式来简化代码提高开发效率。  
本文简单介绍下 **Annotation 示例、概念及作用、分类、自定义、解析，并对几个 Android 开源库 Annotation 原理进行简析**。  
 
### 1. Annotation 示例  
Override Annotation  

    @Override
    public void onCreate(Bundle savedInstanceState);
Retrofit Annotation  

    @GET("/users/{username}")
    User getUser(@Path("username") String username);
Butter Knife Annotation  

    @InjectView(R.id.user) EditText username;
ActiveAndroid Annotation  

    @Column(name = “Name") public String name;
Retrofit 为符合 RESTful 规范的网络请求框架  
Butter Knife 为 View 及事件等依赖注入框架  
Active Android 为 ORM 框架  
更多见：[Android 开源项目汇总](https://github.com/Trinea/android-open-project)  

### 2. Annotation 概念及作用  
#### 2.1 概念  
> An annotation is a form of metadata, that can be added to Java source code. Classes, methods, variables, parameters and packages may be annotated. Annotations have no direct effect on the operation of the code they annotate.  

能够添加到 Java 源代码的语法元数据。类、方法、变量、参数、包都可以被注解，可用来将信息元数据与程序元素进行关联。Annotation 中文常译为“注解”。  

#### 2.2 作用  
a. 标记，用于告诉编译器一些信息  
b. 编译时动态处理，如动态生成代码  
c. 运行时动态处理，如得到注解信息  
这里的三个作用实际对应着后面自定义 Annotation 时说的 @Retention 三种值分别表示的 Annotation   

    public class Person {

        private int    id;
        private String name;

        public Person(int id, String name) {
            this.id = id;
            this.name = name;
        }

        public boolean equals(Person person) {
            return person.id == id;
        }

        public int hashCode() {
            return id;
        }

        public static void main(String[] args) {

            Set<Person> set = new HashSet<Person>();
            for (int i = 0; i < 10; i++) {
                set.add(new Person(i, "Jim"));
            }
            System.out.println(set.size());
        }
    }
上面的运行结果是多少？  

### 3. Annotation 分类  
#### 3.1 标准 Annotation，Override, Deprecated, SuppressWarnings  
标准 Annotation 是指 Java 自带的几个 Annotation，上面三个分别表示重写函数，不鼓励使用(有更好方式、使用有风险或已不在维护)，忽略某项 Warning  
#### 3.2 元 Annotation，@Retention, @Target, @Inherited, @Documented  
元 Annotation 是指用来定义 Annotation 的 Annotation，在后面 Annotation 自定义部分会详细介绍含义  
#### 3.3 自定义 Annotation  
自定义 Annotation 表示自己根据需要定义的 Annotation，定义时需要用到上面的元 Annotation  
这里是一种分类而已，也可以根据作用域分为源码时、编译时、运行时 Annotation，后面在自定义 Annotation 时会具体介绍  

### 4. Annotation 自定义  
#### 4.1 调用

    public class App {

        @MethodInfo(
            author = “trinea.cn+android@gmail.com”,
            date = "2014/02/14",
            version = 2)
        public String getAppName() {
            return "trinea";
        }
    }
这里是调用自定义 Annotation——MethodInfo 的示例。  
MethodInfo Annotation 作用为给方法添加相关信息，包括 author、date、version。  

#### 4.2 定义

    @Documented
    @Retention(RetentionPolicy.RUNTIME)
    @Target(ElementType.METHOD)
    @Inherited
    public @interface MethodInfo {

        String author() default "trinea@gmail.com";

        String date();

        int version() default 1;
    }
    
这里是 MethodInfo 的实现部分  
(1). 通过 @interface 定义，注解名即为自定义注解名  
(2). 注解配置参数名为注解类的方法名，且：  
a. 所有方法没有方法体，没有参数没有修饰符，实际只允许 public & abstract 修饰符，默认为 public，不允许抛异常  
b. 方法返回值只能是基本类型，String, Class, annotation, enumeration 或者是他们的一维数组  
c. 若只有一个默认属性，可直接用 value() 函数。一个属性都没有表示该 Annotation 为 Mark Annotation  
(3). 可以加 default 表示默认值  

#### 4.3 元 Annotation  
@Documented 是否会保存到 Javadoc 文档中  
@Retention 保留时间，可选值 SOURCE（源码时），CLASS（编译时），RUNTIME（运行时），默认为 CLASS，SOURCE 大都为 Mark Annotation，这类 Annotation 大都用来校验，比如 Override, SuppressWarnings  
@Target 可以用来修饰哪些程序元素，如 TYPE, METHOD, CONSTRUCTOR, FIELD, PARAMETER 等，未标注则表示可修饰所有  
@Inherited 是否可以被继承，默认为 false  

### 5. Annotation 解析  
#### 5.1 运行时 Annotation 解析  
(1) 运行时 Annotation 指 @Retention 为 RUNTIME 的 Annotation，可手动调用下面常用 API 解析

    method.getAnnotation(AnnotationName.class);
    method.getAnnotations();
    method.isAnnotationPresent(AnnotationName.class);
    
其他 @Target 如 Field，Class 方法类似  
getAnnotation(AnnotationName.class) 表示得到该 Target 某个 Annotation 的信息，因为一个 Target 可以被多个 Annotation 修饰  
getAnnotations() 则表示得到该 Target 所有 Annotation  
isAnnotationPresent(AnnotationName.class) 表示该 Target 是否被某个 Annotation 修饰  
(2) 解析示例如下：  

    public static void main(String[] args) {
        try {
            Class cls = Class.forName("cn.trinea.java.test.annotation.App");
            for (Method method : cls.getMethods()) {
                MethodInfo methodInfo = method.getAnnotation(
    MethodInfo.class);
                if (methodInfo != null) {
                    System.out.println("method name:" + method.getName());
                    System.out.println("method author:" + methodInfo.author());
                    System.out.println("method version:" + methodInfo.version());
                    System.out.println("method date:" + methodInfo.date());
                }
            }
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
    
以之前自定义的 MethodInfo 为例，利用 Target（这里是 Method）getAnnotation 函数得到 Annotation 信息，然后就可以调用 Annotation 的方法得到响应属性值  

#### 5.2 编译时 Annotation 解析  
(1) 编译时 Annotation 指 @Retention 为 CLASS 的 Annotation，甴编译器自动解析。需要做的  
a. 自定义类集成自 AbstractProcessor   
b. 重写其中的 process 函数  
这块很多同学不理解，实际是编译器在编译时自动查找所有继承自 AbstractProcessor 的类，然后调用他们的 process 方法去处理  
(2) 假设 MethodInfo 的 @Retention 为 CLASS，解析示例如下：  

    @SupportedAnnotationTypes({ "cn.trinea.java.test.annotation.MethodInfo" })
    public class MethodInfoProcessor extends AbstractProcessor {

        @Override
        public boolean process(Set<? extends TypeElement> annotations, RoundEnvironment env) {
            HashMap<String, String> map = new HashMap<String, String>();
            for (TypeElement te : annotations) {
                for (Element element : env.getElementsAnnotatedWith(te)) {
                    MethodInfo methodInfo = element.getAnnotation(MethodInfo.class);
                    map.put(element.getEnclosingElement().toString(), methodInfo.author());
                }
            }
            return false;
        }
    }
    
SupportedAnnotationTypes 表示这个 Processor 要处理的 Annotation 名字。  
process 函数中参数 annotations 表示待处理的 Annotations，参数 env 表示当前或是之前的运行环境  
process 函数返回值表示这组 annotations 是否被这个 Processor 接受，如果接受后续子的 rocessor 不会再对这个 Annotations 进行处理  

### 6. 几个 Android 开源库 Annotation 原理简析  
#### 6.1 Annotation — Retrofit  
(1) 调用

    @GET("/users/{username}")
    User getUser(@Path("username") String username);
    
(2) 定义  

    @Documented
    @Target(METHOD)
    @Retention(RUNTIME)
    @RestMethod("GET")
    public @interface GET {
      String value();
    }

从定义可看出 Retrofit 的 Get Annotation 是运行时 Annotation，并且只能用于修饰 Method  
(3) 原理  
    
    private void parseMethodAnnotations() {
        for (Annotation methodAnnotation : method.getAnnotations()) {
        Class<? extends Annotation> annotationType = methodAnnotation.annotationType();
        RestMethod methodInfo = null;
      
        for (Annotation innerAnnotation : annotationType.getAnnotations()) {
            if (RestMethod.class == innerAnnotation.annotationType()) {
                methodInfo = (RestMethod) innerAnnotation;
                break;
            }
        }
        ……
        }
    }   
    
[RestMethodInfo.java](https://github.com/square/retrofit/blob/master/retrofit/src/main/java/retrofit/RestMethodInfo.java) 的 parseMethodAnnotations 方法如上，会检查每个方法的每个 Annotation， 看是否被 RestMethod 这个 Annotation 修饰的 Annotation 修饰，这个有点绕，就是是否被 GET、DELETE、POST、PUT、HEAD、PATCH 这些 Annotation 修饰，然后得到 Annotation 信息，在对接口进行动态代理时会掉用到这些 Annotation 信息从而完成调用。  

Retrofit 原理涉及到[动态代理](http://a.codekk.com/detail/Android/Caij/%E5%85%AC%E5%85%B1%E6%8A%80%E6%9C%AF%E7%82%B9%E4%B9%8B%20Java%20%E5%8A%A8%E6%80%81%E4%BB%A3%E7%90%86)，这里原理都只介绍 Annotation，具体原理分析请见 [Android 开源项目实现原理解析](http://a.codekk.com)   

#### 6.2 Annotation — Butter Knife  
(1) 调用

    @InjectView(R.id.user) 
    EditText username;
    
(2) 定义

    @Retention(CLASS) 
    @Target(FIELD)
    public @interface InjectView {
      int value();
    }

可看出 Butter Knife 的 InjectView Annotation 是编译时 Annotation，并且只能用于修饰属性  
(3) 原理  

    @Override 
    public boolean process(Set<? extends TypeElement> elements, RoundEnvironment env) {
        Map<TypeElement, ViewInjector> targetClassMap = findAndParseTargets(env);

        for (Map.Entry<TypeElement, ViewInjector> entry : targetClassMap.entrySet()) {
            TypeElement typeElement = entry.getKey();
            ViewInjector viewInjector = entry.getValue();

            try {
                JavaFileObject jfo = filer.createSourceFile(viewInjector.getFqcn(), typeElement);
                Writer writer = jfo.openWriter();
                writer.write(viewInjector.brewJava());
                writer.flush();
                writer.close();
            } catch (IOException e) {
                error(typeElement, "Unable to write injector for type %s: %s", typeElement, e.getMessage());
            }
        }

        return true;
    }


[ButterKnifeProcessor.java](https://github.com/JakeWharton/butterknife/blob/master/butterknife/src/main/java/butterknife/internal/ButterKnifeProcessor.java) 的 process 方法如上，编译时，在此方法中过滤 InjectView 这个 Annotation 到 targetClassMap 后，会根据 targetClassMap 中元素生成不同的 class 文件到最终的 APK 中，然后在运行时调用 ButterKnife.inject(x) 函数时会到之前编译时生成的类中去找。
这里原理都只介绍 Annotation，具体原理分析请见 [Android 开源项目实现原理解析](http://a.codekk.com)   

#### 6.3 Annotation — ActiveAndroid  
(1) 调用  
  
    @Column(name = “Name") 
    public String name;
  
(2) 定义

    @Target(ElementType.FIELD)
    @Retention(RetentionPolicy.RUNTIME)
    public @interface Column {
      ……
    }
  
可看出 ActiveAndroid 的 Column Annotation 是运行时 Annotation，并且只能用于修饰属性。   
(3) 原理      

    Field idField = getIdField(type);
    mColumnNames.put(idField, mIdName);

    List<Field> fields = new LinkedList<Field>(ReflectionUtils.getDeclaredColumnFields(type));
    Collections.reverse(fields);

    for (Field field : fields) {
        if (field.isAnnotationPresent(Column.class)) {
            final Column columnAnnotation = field.getAnnotation(Column.class);
            String columnName = columnAnnotation.name();
            if (TextUtils.isEmpty(columnName)) {
                columnName = field.getName();
            }

            mColumnNames.put(field, columnName);
        }
    }
 
[TableInfo.java](https://github.com/pardom/ActiveAndroid/blob/master/src/com/activeandroid/TableInfo.java) 的构造函数如上，运行时，得到所有行信息并存储起来用来构件表信息。  

这里原理都只介绍 Annotation，具体原理分析请见 [Android 开源项目实现原理解析](http://a.codekk.com)   

