Retrofit 源码解析
======

> 本文为 [Android 开源项目源码解析](http://a.codekk.com) 中 Retrofit 部分  
> 项目地址：[Retrofit](https://github.com/square/retrofit)，分析的版本：[35ce778](https://github.com/square/retrofit/commit/e68011938e92d7f50f8e2a64ad0e57788549dd5c)，Demo 地址：[Retrofit Demo](https://github.com/android-cn/android-open-project-demo/tree/master/Retrofit-demo)    
> 分析者：[xxxzhi](https://github.com/xxxzhi)，分析状态：完成，校对者：[Trinea](https://github.com/trinea)，校对状态：未开始   

### 1. 功能介绍
#### 1.1 Retrofit
Retrofit 是 Github 上面 squre 组织开发的一个类型安全的 Http 客户端，它可以在 Java 和 Android 上面使用。Retrofit 将描述请求的接口转换为对象，然后再由该对象去请求后台。Retrofit 将请求对象化了。目前已经发布了 2.0beta 版本。

#### 1.2 特点
Retrofit 主要有以下功能特点

1. 将 Http 请求对象化，函数化。让接口的函数代表具体请求。
2. 利用注解的方式标记参数，将 HTTP 的请求方法，请求头，请求参数，请求体等等都用注解的方式标记，使用起来非常方便。
3. 支持 Multipart，以及文件上传（file upload）。
4. 直接将 Http 的 Response 转换成对象。用户可以根据 Response 的具体内容，更换转换器，或者自己新建转化器。
5. Retrofit 默认使用 OkHttp 开源库请求后台，用户也可以使用自定义的具体请求方式。方便扩展。
6. 自带提供了异步处理 Http 请求的方式。

#### 1.3 简单 Demo
这是一个简单的例子，访问[httpbin](https://httpbin.org/)网站。也可以看完整的[Retrofit Demo](https://github.com/android-cn/android-open-project-demo/tree/master/Retrofit-demo)
首先声明一个 java 接口

    public interface HttpbinService {
        @GET("/get?arg1=hello")
        Call<HttpbinRequest> testGet();
    }


使用方式

    Retrofit retrofit = new Retrofit.Builder()
                .baseUrl("https://httpbin.org")
                .addConverterFactory(GsonConverterFactory.create())
                .build();
                
    HttpbinService httpbinService = retrofit.create(HttpbinService.class);


使用 httpbinService 获取一个 Call，用来请求 HTTP 服务器。

    Call<HttpbinRequest> call = httpbinService.testGet();

因为接口返回的应该是一个 Call，用来请求后台 HTPP 服务器，所以我们在声明接口的似乎，返回参数应该是 Call<?>。由于 httpbin 返回的是一个 json 格式的数据，我们想要返回直接的自定义的模型数据，但是 retrofit 默认只会返回 ResponseBody，所以我们需要自己添加一个 GsonConverter 第三方库。在 build.graddle 中的 dependencies 添加：

     compile 'com.squareup.retrofit:converter-gson:2.0.0-beta2'


### 2. 总体设计
Retrofit 可以分为注解解析（Request 生成），请求执行，请求回调（异步处理），响应体转化几个部分。其中请求执行与请求回调可以算作一个部分，并且请求回调也可以没有，Call 有直接执行的接口 execute。

![Retrofit 总体结构][1]

1. 首先由解析部分（这部分也是 Request 生成部分），利用注解（Annotation）解析接口文件，将接口方法解析好，每个方法生成一个 Request。
2. 然后利用 Call 部分执行 Request。Retrofit 使用的是 okHttp 来请求，程序中将 Retrofit Request 转化为 OKHttp 开源库的 Request，转由 OkHttpClient 执行。
3. 在 Request 执行完后，得到 Response，使用 Converter 转化 Response 为用户需要的对象。比如将 json 格式的数据，利用 gson 转化为具体的 Object（也就是接口函数中的返回 Call 的模版参数的具体类型对象）
4. 利用回调将第三步得到的对象，将对象传回给 UI 线程，更新 UI。

这里面第三部与第四步是可以合在一起的，但是目前 Retrofit 提供的默认代码中，会通过 Call，加入 Callback，用户可以在 Callback 中处理结果。

注解（Annotation）是 Retrofit 预先定义的注解，包括 Http 的各个部分，比如 POST、GET、Query、QueryMap、Field 等等。

### 3. 流程图

![Retrofit 使用流程图][2]


其中生成 Call 的部分可以看下面关于这个适配器的类图。

### 4. 详细设计
#### 4.1 类图
首先是整个项目的类图
![Retrofit UML 图][3]

对于 Retrofit 项目中 CallAdapter 用着适配器模式也挺巧的，通过适配器将 Callback 回调接口运行在 UI 线程。下面时有关 CallAdapter，Call，Callback 的类图，其中也是连续用了两次代理模式。

![CallAdapter uml 图][4]

ExecutorCallback 代理的是用户自定义的 Callback。通过这种方式让 OkHttpCall 去执行 Call，让 ExecutorCallback 将用户自定义的 Callback 运行在指定线程上。


#### 4.2 类功能详细介绍
在 Retrofit 开源库中，Retrofit 类是用户最基础的访问入口。然后 Converter 部分是由用户自己扩展的，而 Paraser 部分的相关类 RequestBuilder，RequestFactory 等则主要是负责解析接口并且生成 Request，而 Call，CallAdapter 等主要是负责底层的 Http 请求，以及请求后线程转换。

##### 4.2.1 Retrofit
Retrofit 类是包含了一个构造器 Retrofit.Builder，由 Builder 指定 Retrofit 的相关参数，创建一个新的 Retrofit。Retrofit 中包含了很多重要的成员变量，而这些成员变量都是可以自设置的。

Retrofit 包含以下成员变量：
- baseUrl: Http 请求的基础 url，类型是 BaseUrl，包含了 url 函数返回 HttpUrl（OkHttp 的类），由 Retrofit.Builder.baseUrl 设置。
- client：OkHttp 库的 OkHttpClient 类型。由 Builder 的 client 函数设置，默认为`OkHttpClient()`。
- methodHandlerCache：Map 类型，MethodHandler 的缓存，从接口中解析出来，放在这个 map 里面。
- converterFactories：List 类型，包含了很多 converter 的创建工厂，用户可以通过 Builder 的 addConverterFactory 来添加。默认添加了 BuiltInConverters。
- callbackExecutor：回调函数的执行器，也就是回调函数执行的线程，Android 中默认为 MainThreadExecutor。
- adapterFactories：List 类型，包含了 CallAdapter.Factory，用户可以通过 Builder 的 addCallAdapterFactory 来添加。Android 中默认添加了 ExecutorCallAdapterFactory。使用 callbackExecutor 作为 Executor。
- validateEagerly：这个是设置的在创建动态代理对象之前，是否提前解析接口 Method，创建 MethodHandler 并添加到 Cache 中。

Retrofit 重要方法:
- create(final Class<T> service):T
这个是一个 public 模版方法，用户可以通过这个方法，传入接口 Class（T），获得接口 Class Http 请求的动态代理对象。这是该开源库的主入口，这个函数先验证接口以及其方法，然后创建一个匿名 InvocationHandler，在匿名 InvocationHandler 的 invoke 中首先去掉 Object 以及 Platform 默认的方法，然后调用 loadMethodHandler 解析对应的方法（接口方法），创建 MethodHandler 加入到 methodHandlerCache 中，得到 MethodHandler，最后调用 MethodHandler 的 invoke 方法得到返回结果（接口方法的返回类型）。动态代理请见[Java 动态代理][5]
- loadMethodHandler(Method method):MethodHandler<?> 
解析对应的方法（接口方法），创建 MethodHandler 加入到 methodHandlerCache 中，返回得到 MethodHandler。
- nextCallAdapter(CallAdapter.Factory skipPast, Type returnType,
      Annotation[] annotations):CallAdapter<?> 
该方法主要是从 callAdapterFactories 中获取新的 CallAdapter，它会跳过 skipPast，以及 skipPast 之前的 Factory，然后找到与 returnType 和 annotations 都匹配的 CallAdapterFactory。如果不匹配 CallAdapterFactory 的 get 会返回 null，所以搜索 Factories 的时候，直到搜索到返回非 null 就找到对应的了。

如果没有找到对应的 CallAdapterFactories，得到 CallAdapter，则该方法会抛出一个 IllegalArgumentException 异常，异常里面的 message 会是"Could not locate call adapter for "，如果遇到这个异常，则去判断对应的方法的返回类型是不是与 CallAdapterFactory 不匹配。
- requestConverter(Type type, Annotation[] annotations):Converter<T, RequestBody> 
也是模版方法，该方法返回 Converter。利用 converterFactories 创建一个与 RequestBody 对应的 Converter 对象。
如果没有找到对应的 ConverterFactory，得到 Converter，则该方法会抛出一个 IllegalArgumentException 异常，异常里面的 message 会是"Could not locate RequestBody converter for  "。同样，如果遇到这个异常，则去判断对应的方法的返回类型是不是与 ConverterFactory 不匹配。
- responseConverter(Type type, Annotation[] annotations): Converter<ResponseBody, T> 
与 requestConverter 类似，不过该方法对应的是 Response。


##### 4.2.2 MethodHandler
MethodHandler 是 retrofit 中连接了解析部分，执行部分，转换部分的一个关键的中间类。不过 MethodHandler 的代码量很少。它可以说是连接各个部分的桥梁，也是接口方法的描述类。它有包含了 retrofit，requestFactory，callAdapter，responseConverter 成员变量。主要方法如下

- create(Retrofit retrofit, Method method):MethodHandler<?> 
这是个静态方法。MethodHandler 的创建方法，在这个方法里面通过创建 CallAdapter，responseConverter，requestFactory，最后创建 MethodHandler。
- createCallAdapter(Method method, Retrofit retrofit): CallAdapter<?> 
这是个静态方法。通过 retrofit 的 newCallAdapter 创建 CallAdapter
-  createResponseConverter(Method method,Retrofit retrofit, Type responseType):Converter<ResponseBody, ?> 
这是个静态方法。通过 retrofit 的 responseConverter 方法得到 responseConverter
- invoke(Object... args):Object 
通过 callAdapter 的 adapter 将 OkHttpCall 转换成需要返回的 Call
```
  Object invoke(Object... args) {
    return callAdapter.adapt(new OkHttpCall<>(retrofit, requestFactory, responseConverter, args));
  }
```

##### 4.2.3 Converter 与 Converter.Factory
这两个类别都是在 Converter 文件下。Converter 是接口，Factory 抽象类，很简短。
```
public interface Converter<F, T> {
  T convert(F value) throws IOException;
  
  abstract class Factory {
    // 返回将 ResponseBody 转化为 Type 具体的对象的 Converter
    public Converter<ResponseBody, ?> fromResponseBody(Type type, Annotation[] annotations) {
      return null;
    }

    //返回将函数 Body 参数转化为 RequestBody 的 Converter
    public Converter<?, RequestBody> toRequestBody(Type type, Annotation[] annotations) {
      return null;
    }
  }
}
```
Factory 主要是负责生成两种 Converter。Retrofit 实现了一个简单的 BuiltInConverters。

##### 4.2.4 Call
这是 Retrofit 的框架基础接口。它是 Retrofit 的发送请求给服务器并且返回响应体的调用。每个 Call 都有自己的 HTTP 请求和相匹配的响应。
它有如下四个接口：

- execute 同步执行请求
`Response<T> execute() throws IOException; `
- enquene 异步执行请求，并且使用 Callback 作为请求结束后的回调。
`void enqueue(Callback<T> callback); `
- cancel 取消请求
`void cancel(); `
- clone 复制请求，如果需要很多相同的 Call，可以通过 clone 复制。
`Call<T> clone(); `


##### 4.2.5 CallAdapter
这是 Retrofit 的框架基础接口。CallAdapter 是将一个 Call 适配给另外一个 Call 的适配器接口。它有以下两个接口：
- responseType 返回请求后，转化的参数 Type 类型。
`Type responseType(); `
- adapt 适配，将一个 Call 转换成另外一个 Call。
`<R> T adapt(Call<R> call);`

##### 4.2.6 Callback
请求结构的回调接口。在 Call 的 enquene 接口中使用 有如下两个方法

- onResponse 返回响应体
  `void onResponse(Response<T> response, Retrofit retrofit);`
- onFailure 请求失败的时候，比如网络或者一些难以预料的异常。
  `void onFailure(Throwable t);`

##### 4.2.7 OkHttpCall
实现了 Call 接口，但同样是模版类。首先介绍一下 OkHttpCall 的主要函数：
- createRawCall

`private com.squareup.okhttp.Call createRawCall()`
根据由 requestFactory 根据 args 创建一个 Request，然后利用这个 Request 创建一个 okhttp.Call。

- parseResponse
`private Response<T> parseResponse(com.squareup.okhttp.Response rawResponse) throws IOException `

解析 okhttp.Response，
1. 首先将 body 读取出来作为 rawBody，然后用 OkHttpCall.NoContentResponseBody 作为新的 Body，创建新的 rawResponse。
2. 判断 Response.code()，如果不在 200 范围内，读取 rawBody 出来，返回一个错误的 retrofit 的 Response。如果 code 为 204 或 205（没有返回内容），则返回一个 body 为空的 retrofit 的 Response。
3. 如果 code 正常，则用 OkHttpCall.ExceptionCatchingRequestBody 包装一下 rawBody，  然后使用 responseConverter 将包装后的 catchingBody 转化为具体的返回类型数据。

OkHttpCall 是将 Request 放入到 okhttp 的 Call 里面执行，执行完成后，又将 okhttp 的 Call 返回的 Response 转化为 retrofit 的 Response，在此同时将 Body 里面的内容，通过 converter 转化为对应的对象。这个 OkHttpCall

##### 4.2.8 Response
这个类是包含了具体返回对象的响应体。里面包含了模版参数 T 类型的 body 对象，以及 okhttp 的 Response。

##### 4.2.9 注解类
在 Retrofit 里面创建了 Body 注解，Filed 注解（Field，FieldMap），请求方法注解（DELETE，GET，PATCH，POST，PUT），请求头注解（HEAD，Header，Headers），multipart 注解（Part，Multipart，PartMap），接口加码（FormUrlEncoded），Url，Streaming，查询（Query，QueryMap），参数路径（Path），HTTP

##### 4.2.10 RequestBuilderAction
这是一个抽象类，只有一个未实现的 perform 方法。

`abstract void perform(RequestBuilder builder, Object value); `

但是在 RequestBuilderAction 类里面有很多 RequestBuilderAction 的子类，分别对应注解类。Url，Header，Path，Query，QueryMap，Field，FieldMap，Part，PartMap，Body 都是在 RequestBuilderAction 的内部类，并且继承了 RequestBuilderAction。RequestBuilder 就是将对应注解的值给 RequestBuilder。

##### 4.2.11 RequestBuilder
这是一个 okhttp.Request 的创建类。负责设置 HTTP 请求的相关信息，创建 Request。它主要有以下方法：

- RequestBuilder
- setRelativeUrl
- addHeader
- addPathParam
- canonicalize static 方法
- canonicalize
- addQueryParam
- addFormField
- addPart
- setBody
- build


它的构造方法如下：
`RequestBuilder(String method, HttpUrl baseUrl, String relativeUrl, Headers headers, MediaType contentType, boolean hasBody, boolean isFormEncoded, boolean isMultipart)`

RequestBuilder 就是创建请求。

##### 4.2.12 RequestFactory
RequestFactory 是创建 Request，他有个 create 方法，

`  Request create(Object... args) {`

参数是接口函数对应的参数值，cerate 是创建 RequestBuilder，遍历 RequestFactory 的成员变量 requestBuilderActions，设置好 RequestBuilder，最后创建 Request 返回。

##### 4.2.13 RequestFactoryParser
这个类主要是接口函数 Method 的每个注解。入口函数是 parse。
```
  static RequestFactory parse(Method method, Type responseType, Retrofit retrofit) {
    RequestFactoryParser parser = new RequestFactoryParser(method);
    parser.parseMethodAnnotations(responseType);
    parser.parseParameters(retrofit);
    return parser.toRequestFactory(retrofit.baseUrl());
  }
```
先解析方法注解（应用到方法上的注解），比如说 FormUrlEncoded，Headers。得到对应的值。

然后再解析方法参数注解（应用到方法参数上的注解），在解析方法参数注解的时候，会生成一个 requestBuilderActions 数组，对应到每个参数。每个 Action 都对应了每个函数参数的处理。等到具体函数调用的时候，跟函数具体的参数值对应。也就是 RequestFactory 与 Builder 的工作了，这部分是等到运行的时候才能够确定的。


##### 4.2.14 BuiltInConverters，OkHttpResponseBodyConverter，VoidConverter，OkHttpRequestBodyConverter
BuiltInConverters 继承自 Converter.Factory，返回的 responseConverter 是 OkHttpResponseBodyConverter 或 VoidConverter，也就是接口方法返回的职能是 OkHttp 的 ResponseBody，或者 Void。
返回的 requestConverter 是 OkHttpRequestBodyConverter，接口方法的参数中如果使用 Body，那 Body 也只能是 OkHttp 的 RequestBody。

VoidConverter： 将 OkHttp 的 ResponseBody 转化为 Void。
OkHttpResponseBodyConverter：将 OkHttp 的 ResponseBody 转化为 OkHttp 的 ResponseBody。如果是 Streaming 标记的接口的话，利用 Utils.readBodyToBytesIfNecessary 缓冲整个 body。
OkHttpRequestBodyConverter：将 OkHttp 的 RequestBody 转化为 OkHttp 的 RequestBody。

##### 4.2.15 PlatForm.Android.MainThreadExecutor
一个 Executor，通过 android Handler 将 Runnable 执行在 UI 线程中。
```
    static class MainThreadExecutor implements Executor {
      private final Handler handler = new Handler(Looper.getMainLooper());

      @Override public void execute(Runnable r) {
        handler.post(r);
      }
    }
``` 

##### 4.2.16 Utils
这是 Retrofit 中的一个工具类，里面包含了很多范型的检查、操作。另外以及一些基本的工具性的功能。下面是它里面的函数：

- checkNotNull
`<T> T checkNotNull(T object, String message)`
检查非空，如果是 null，则抛出 NullPointerException，内容提示为 message。

- closeQuietly
` static void closeQuietly(Closeable closeable)`
静默地关闭 Closeable 对象。不会抛出异常

-  isAnnotationPresent
`static boolean isAnnotationPresent(Annotation[] annotations,Class<? extends Annotation> cls) `
判断 cls 是否是 annotations 里面的一个实例。如果在则返回 true。

- readBodyToBytesIfNecessary
`static ResponseBody readBodyToBytesIfNecessary(final ResponseBody body) throws IOException `
如果 body 非 null 的话，把整个 body 读取出来（读取到 buffer），返回再返回一个 ResponseBody。

- validateServiceInterface
` static <T> void validateServiceInterface(Class<T> service) `
验证接口是否有效，这个接口就是用户自定义的接口。如果不是接口，或者里面没有任何函数，则抛出 IllegalArgumentException 异常。

- getSingleParameterUpperBound
`public static Type getSingleParameterUpperBound(ParameterizedType type)`
该函数获取 type 的单个模版参数的上届。如果 type 有多个类型，函数会抛出异常，如果模版参数不是 WildcardType，则直接返回模版参数类型

- hasUnresolvableType
`public static boolean hasUnresolvableType(Type type)`
判断是否有不能分解的类型，比如有 TypeVariable，WildcardType 等
- getRawType
`public static Class<?> getRawType(Type type) `
这个方法是从 Gson 里面截取的，获取 type 的实际类型。

- methodError
`static RuntimeException methodError(Method method, String message, Object... args)`
`static RuntimeException methodError(Throwable cause, Method method, String message,Object... args)`
两个重载函数，抛出方法错误异常

- getCallResponseType
`static Type getCallResponseType(Type returnType) `
获取返回 Call 的返回类型，必须是模版参数类型，并且 Call 的模版参数不能是 retrofit.Response.class。返回 getSingleParameterUpperBound(returnType)

#### 4.3 扩展
Retrofit 是很适合扩展的，里面设计的 Call，以及 Converter 就是为了方便扩展使用。

##### 4.3.1 Converter
Retrofit 提供的默认的 Converter 只会返回 ResponseBody，如果我们想要返回具体的 Object，我们可以使用另外的第三方包，并且在创建 Retrofit 的时候添加对应的 ConverterFactory。这里有 6 个序列化第三方库:

- Gson: com.squareup.retrofit:converter-gson
- Jackson: com.squareup.retrofit:converter-jackson
- Moshi: com.squareup.retrofit:converter-moshi
- Protobuf: com.squareup.retrofit:converter-protobuf
- Wire: com.squareup.retrofit:converter-wire
- Simple XML: com.squareup.retrofit:converter-simplexml

##### 4.3.2 Rxjava
retrofit 也可以与[Rxjava](https://github.com/ReactiveX/RxJava)联合起来使用，之前的版本使用范例可以参考[http://randomdotnext.com/retrofit-rxjava/](http://randomdotnext.com/retrofit-rxjava/)

- adapter-Rxjava: com.squareup.retrofit:adapter-rxjava

正在开发中，主要是通过扩展 CallAdapter，将之前 Call，转换为 rxjava 需要的 Observable<?>。

### 5 杂谈
  Retrofit 整体框架的代码并不多，主要是围绕着 converter，CallAdapter 设计的整个框架。花了两三天时间耐耐心心地把代码也是挺有收获。Retrofit 用到的基本技术是动态代理，Java 注解，Java 范型。另外如果对设计模式很熟悉的话，读起来感觉就会事半功倍。整个架构设计的非常好。


  [1]: image/model.png
  [2]: image/flow-draw.png
  [3]: /tool-lib/network/retrofit/image/retrofit-uml.png
  [4]: /tool-lib/network/retrofit/image/call-adapter-uml.png
  [5]: http://a.codekk.com/detail/Android/Caij/%E5%85%AC%E5%85%B1%E6%8A%80%E6%9C%AF%E7%82%B9%E4%B9%8B%20Java%20%E5%8A%A8%E6%80%81%E4%BB%A3%E7%90%86

