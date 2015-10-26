Retrofit源码解析
======

> 本文为 [Android 开源项目源码解析](https://github.com/android-cn/android-open-project-analysis) 中 Retrofit 部分  
> 项目地址：[Retrofit](https://github.com/square/retrofit)，分析的版本：[35ce778](https://github.com/square/retrofit/commit/e68011938e92d7f50f8e2a64ad0e57788549dd5c)，Demo 地址：[Retrofit Demo](https://github.com/android-cn/android-open-project-demo/tree/master/Retrofit-demo)    
> 分析者：[xxxzhi](https://github.com/xxxzhi)，

###1. 功能介绍
####1.1 Retrofit
Retrofit是Github上面squre组织开发的一个类型安全的Http客户端，它可以在Java和Android上面使用。Retrofit将描述请求的接口转换为对象，然后再由该对象去请求后台。Retrofit将请求对象化了。目前已经发布了2.0beta版本。

####1.2 特点
Retrofit主要有以下功能特点

1. 将Http请求对象化，函数化。让接口的函数代表具体请求。
2. 利用注解的方式标记参数，将HTTP的请求方法，请求头，请求参数，请求体等等都用注解的方式标记，使用起来非常方便。
3. 支持Multipart，以及文件上传（file upload）。
4. 直接将Http的Response转换成对象。用户可以根据Response的具体内容，更换转换器，或者自己新建转化器。
5. Retrofit默认使用OkHttp开源库请求后台，用户也可以使用自定义的具体请求方式。方便扩展。
6. 自带提供了异步处理Http请求的方式。

####1.3简单Demo
这是一个简单的例子，访问[httpbin](https://httpbin.org/)网站。也可以看完整的[Retrofit Demo](https://github.com/android-cn/android-open-project-demo/tree/master/Retrofit-demo)
首先声明一个java接口

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


使用httpbinService获取一个Call，用来请求HTTP服务器。

    Call<HttpbinRequest> call = httpbinService.testGet();

因为接口返回的应该是一个Call，用来请求后台HTPP服务器，所以我们在声明接口的似乎，返回参数应该是Call<?>。由于httpbin返回的是一个json格式的数据，我们想要返回直接的自定义的模型数据，但是retrofit默认只会返回ResponseBody，所以我们需要自己添加一个GsonConverter第三方库。在build.graddle中的dependencies添加：

     compile 'com.squareup.retrofit:converter-gson:2.0.0-beta2'


###2. 总体设计
Retrofit可以分为注解解析（Request生成），请求执行，请求回调（异步处理），响应体转化几个部分。其中请求执行与请求回调可以算作一个部分，并且请求回调也可以没有，Call有直接执行的接口execute。

![Retrofit总体结构][1]

1. 首先由解析部分（这部分也是Request生成部分），利用注解（Annotation）解析接口文件，将接口方法解析好，每个方法生成一个Request。
2. 然后利用Call部分执行Request。Retrofit使用的是okHttp来请求，程序中将Retrofit Request转化为OKHttp开源库的Request，转由OkHttpClient执行。
3. 在Request执行完后，得到Response，使用Converter转化Response为用户需要的对象。比如将json格式的数据，利用gson转化为具体的Object（也就是接口函数中的返回Call的模版参数的具体类型对象）
4. 利用回调将第三步得到的对象，将对象传回给UI线程，更新UI。

这里面第三部与第四步是可以合在一起的，但是目前Retrofit提供的默认代码中，会通过Call，加入Callback，用户可以在Callback中处理结果。

注解（Annotation）是Retrofit预先定义的注解，包括Http的各个部分，比如POST、GET、Query、QueryMap、Field等等。

###3. 流程图

![Retrofit使用流程图][2]


其中生成Call的部分可以看下面关于这个适配器的类图。

###4. 详细设计
####4.1 类图
首先是整个项目的类图
![Retrofit UML图][3]

对于Retrofit项目中CallAdapter用着适配器模式也挺巧的，通过适配器将Callback回调接口运行在UI线程。下面时有关CallAdapter，Call，Callback的类图，其中也是连续用了两次代理模式。

![CallAdapter uml图][4]

ExecutorCallback代理的是用户自定义的Callback。通过这种方式让OkHttpCall去执行Call，让ExecutorCallback将用户自定义的Callback运行在指定线程上。


####4.2 类功能详细介绍
在Retrofit开源库中，Retrofit类是用户最基础的访问入口。然后Converter部分是由用户自己扩展的，而Paraser部分的相关类RequestBuilder，RequestFactory等则主要是负责解析接口并且生成Request，而Call，CallAdapter等主要是负责底层的Http请求，以及请求后线程转换。

#####4.2.1 Retrofit
Retrofit类是包含了一个构造器Retrofit.Builder，由Builder指定Retrofit的相关参数，创建一个新的Retrofit。Retrofit中包含了很多重要的成员变量，而这些成员变量都是可以自设置的。

Retrofit包含以下成员变量：
- baseUrl: Http请求的基础url，类型是BaseUrl，包含了url函数返回HttpUrl（OkHttp的类），由Retrofit.Builder.baseUrl设置。
- client：OkHttp库的OkHttpClient类型。由Builder的client函数设置，默认为`OkHttpClient()`。
- methodHandlerCache：Map类型，MethodHandler的缓存，从接口中解析出来，放在这个map里面。
- converterFactories：List类型，包含了很多converter的创建工厂，用户可以通过Builder的addConverterFactory来添加。默认添加了BuiltInConverters。
- callbackExecutor：回调函数的执行器，也就是回调函数执行的线程，Android中默认为MainThreadExecutor。
- adapterFactories：List类型，包含了CallAdapter.Factory，用户可以通过Builder的addCallAdapterFactory来添加。Android中默认添加了ExecutorCallAdapterFactory。使用callbackExecutor作为Executor。
- validateEagerly：这个是设置的在创建动态代理对象之前，是否提前解析接口Method，创建MethodHandler并添加到Cache中。

Retrofit重要方法:
- create(final Class<T> service):T
这个是一个public模版方法，用户可以通过这个方法，传入接口Class（T），获得接口Class Http请求的动态代理对象。这是该开源库的主入口，这个函数先验证接口以及其方法，然后创建一个匿名InvocationHandler，在匿名InvocationHandler的invoke中首先去掉Object以及Platform默认的方法，然后调用loadMethodHandler解析对应的方法（接口方法），创建MethodHandler加入到methodHandlerCache中，得到MethodHandler，最后调用MethodHandler的invoke方法得到返回结果（接口方法的返回类型）。动态代理请见[Java动态代理][5]
- loadMethodHandler(Method method):MethodHandler<?> 
解析对应的方法（接口方法），创建MethodHandler加入到methodHandlerCache中，返回得到MethodHandler。
- nextCallAdapter(CallAdapter.Factory skipPast, Type returnType,
      Annotation[] annotations):CallAdapter<?> 
该方法主要是从callAdapterFactories中获取新的CallAdapter，它会跳过skipPast，以及skipPast之前的Factory，然后找到与returnType和annotations都匹配的CallAdapterFactory。如果不匹配CallAdapterFactory的get会返回null，所以搜索Factories的时候，直到搜索到返回非null就找到对应的了。

如果没有找到对应的CallAdapterFactories，得到CallAdapter，则该方法会抛出一个IllegalArgumentException异常，异常里面的message会是"Could not locate call adapter for "，如果遇到这个异常，则去判断对应的方法的返回类型是不是与CallAdapterFactory不匹配。
- requestConverter(Type type, Annotation[] annotations):Converter<T, RequestBody> 
也是模版方法，该方法返回Converter。利用converterFactories创建一个与RequestBody对应的Converter对象。
如果没有找到对应的ConverterFactory，得到Converter，则该方法会抛出一个IllegalArgumentException异常，异常里面的message会是"Could not locate RequestBody converter for  "。同样，如果遇到这个异常，则去判断对应的方法的返回类型是不是与ConverterFactory不匹配。
- responseConverter(Type type, Annotation[] annotations): Converter<ResponseBody, T> 
与requestConverter类似，不过该方法对应的是Response。


#####4.2.2 MethodHandler
MethodHandler是retrofit中连接了解析部分，执行部分，转换部分的一个关键的中间类。不过MethodHandler的代码量很少。它可以说是连接各个部分的桥梁，也是接口方法的描述类。它有包含了retrofit，requestFactory，callAdapter，responseConverter成员变量。主要方法如下

- create(Retrofit retrofit, Method method):MethodHandler<?> 
这是个静态方法。MethodHandler的创建方法，在这个方法里面通过创建CallAdapter，responseConverter，requestFactory，最后创建MethodHandler。
- createCallAdapter(Method method, Retrofit retrofit): CallAdapter<?> 
这是个静态方法。通过retrofit的newCallAdapter创建CallAdapter
-  createResponseConverter(Method method,Retrofit retrofit, Type responseType):Converter<ResponseBody, ?> 
这是个静态方法。通过retrofit的responseConverter方法得到responseConverter
- invoke(Object... args):Object 
通过callAdapter的adapter将OkHttpCall转换成需要返回的Call
```
  Object invoke(Object... args) {
    return callAdapter.adapt(new OkHttpCall<>(retrofit, requestFactory, responseConverter, args));
  }
```

#####4.2.3 Converter 与Converter.Factory
这两个类别都是在Converter文件下。Converter是接口，Factory抽象类，很简短。
```
public interface Converter<F, T> {
  T convert(F value) throws IOException;
  
  abstract class Factory {
    // 返回将ResponseBody转化为Type具体的对象的Converter
    public Converter<ResponseBody, ?> fromResponseBody(Type type, Annotation[] annotations) {
      return null;
    }

    //返回将函数Body参数转化为RequestBody的Converter
    public Converter<?, RequestBody> toRequestBody(Type type, Annotation[] annotations) {
      return null;
    }
  }
}
```
Factory主要是负责生成两种Converter。Retrofit实现了一个简单的BuiltInConverters。

#####4.2.4 Call
这是Retrofit的框架基础接口。它是Retrofit的发送请求给服务器并且返回响应体的调用。每个Call都有自己的HTTP请求和相匹配的响应。
它有如下四个接口：

- execute 同步执行请求
`Response<T> execute() throws IOException; `
- enquene 异步执行请求，并且使用Callback作为请求结束后的回调。
`void enqueue(Callback<T> callback); `
- cancel 取消请求
`void cancel(); `
- clone 复制请求，如果需要很多相同的Call，可以通过clone复制。
`Call<T> clone(); `


#####4.2.5 CallAdapter
这是Retrofit的框架基础接口。CallAdapter是将一个Call适配给另外一个Call的适配器接口。它有以下两个接口：
- responseType 返回请求后，转化的参数Type类型。
`Type responseType(); `
- adapt 适配，将一个Call转换成另外一个Call。
`<R> T adapt(Call<R> call);`

#####4.2.6 Callback
请求结构的回调接口。在Call的enquene接口中使用 有如下两个方法

- onResponse 返回响应体
  `void onResponse(Response<T> response, Retrofit retrofit);`
- onFailure 请求失败的时候，比如网络或者一些难以预料的异常。
  `void onFailure(Throwable t);`

#####4.2.7 OkHttpCall
实现了Call接口，但同样是模版类。首先介绍一下OkHttpCall的主要函数：
- createRawCall

`private com.squareup.okhttp.Call createRawCall()`
根据由requestFactory根据args创建一个Request，然后利用这个Request创建一个okhttp.Call。

- parseResponse
`private Response<T> parseResponse(com.squareup.okhttp.Response rawResponse) throws IOException `

解析okhttp.Response，
1. 首先将body读取出来作为rawBody，然后用OkHttpCall.NoContentResponseBody作为新的Body，创建新的rawResponse。
2. 判断Response.code()，如果不在200范围内，读取rawBody出来，返回一个错误的retrofit的Response。如果code为204或205（没有返回内容），则返回一个body为空的retrofit的Response。
3. 如果code正常，则用OkHttpCall.ExceptionCatchingRequestBody包装一下rawBody，  然后使用responseConverter将包装后的catchingBody转化为具体的返回类型数据。

OkHttpCall是将Request放入到okhttp的Call里面执行，执行完成后，又将okhttp的Call返回的Response转化为retrofit的Response，在此同时将Body里面的内容，通过converter转化为对应的对象。这个OkHttpCall

#####4.2.8 Response
这个类是包含了具体返回对象的响应体。里面包含了模版参数T类型的body对象，以及okhttp的Response。

#####4.2.9 注解类
在Retrofit里面创建了Body注解，Filed注解（Field，FieldMap），请求方法注解（DELETE，GET，PATCH，POST，PUT），请求头注解（HEAD，Header，Headers），multipart注解（Part，Multipart，PartMap），接口加码（FormUrlEncoded），Url，Streaming，查询（Query，QueryMap），参数路径（Path），HTTP

#####4.2.10 RequestBuilderAction
这是一个抽象类，只有一个未实现的perform方法。

`abstract void perform(RequestBuilder builder, Object value); `

但是在RequestBuilderAction类里面有很多RequestBuilderAction的子类，分别对应注解类。Url，Header，Path，Query，QueryMap，Field，FieldMap，Part，PartMap，Body都是在RequestBuilderAction的内部类，并且继承了RequestBuilderAction。RequestBuilder就是将对应注解的值给RequestBuilder。

#####4.2.11 RequestBuilder
这是一个okhttp.Request的创建类。负责设置HTTP请求的相关信息，创建Request。它主要有以下方法：

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

RequestBuilder就是创建请求。

#####4.2.12 RequestFactory
RequestFactory是创建Request，他有个create方法，

`  Request create(Object... args) {`

参数是接口函数对应的参数值，cerate是创建RequestBuilder，遍历RequestFactory的成员变量requestBuilderActions，设置好RequestBuilder，最后创建Request返回。

#####4.2.13 RequestFactoryParser
这个类主要是接口函数Method的每个注解。入口函数是parse。
```
  static RequestFactory parse(Method method, Type responseType, Retrofit retrofit) {
    RequestFactoryParser parser = new RequestFactoryParser(method);
    parser.parseMethodAnnotations(responseType);
    parser.parseParameters(retrofit);
    return parser.toRequestFactory(retrofit.baseUrl());
  }
```
先解析方法注解（应用到方法上的注解），比如说FormUrlEncoded，Headers。得到对应的值。

然后再解析方法参数注解（应用到方法参数上的注解），在解析方法参数注解的时候，会生成一个requestBuilderActions数组，对应到每个参数。每个Action都对应了每个函数参数的处理。等到具体函数调用的时候，跟函数具体的参数值对应。也就是RequestFactory与Builder的工作了，这部分是等到运行的时候才能够确定的。


#####4.2.14 BuiltInConverters，OkHttpResponseBodyConverter，VoidConverter，OkHttpRequestBodyConverter
BuiltInConverters 继承自Converter.Factory，返回的responseConverter是OkHttpResponseBodyConverter或VoidConverter，也就是接口方法返回的职能是OkHttp的ResponseBody，或者Void。
返回的requestConverter是OkHttpRequestBodyConverter，接口方法的参数中如果使用Body，那Body也只能是OkHttp的RequestBody。

VoidConverter： 将OkHttp的ResponseBody转化为Void。
OkHttpResponseBodyConverter：将OkHttp的ResponseBody转化为OkHttp的ResponseBody。如果是Streaming标记的接口的话，利用Utils.readBodyToBytesIfNecessary缓冲整个body。
OkHttpRequestBodyConverter：将OkHttp的RequestBody转化为OkHttp的RequestBody。

#####4.2.15 PlatForm.Android.MainThreadExecutor
一个Executor，通过android Handler将Runnable执行在UI线程中。
```
    static class MainThreadExecutor implements Executor {
      private final Handler handler = new Handler(Looper.getMainLooper());

      @Override public void execute(Runnable r) {
        handler.post(r);
      }
    }
```

####4.3 扩展
Retrofit是很适合扩展的，里面设计的Call，以及Converter就是为了方便扩展使用。

#####4.3.1 Converter
Retrofit提供的默认的Converter只会返回ResponseBody，如果我们想要返回具体的Object，我们可以使用另外的第三方包，并且在创建Retrofit的时候添加对应的ConverterFactory。这里有6个序列化第三方库:

- Gson: com.squareup.retrofit:converter-gson
- Jackson: com.squareup.retrofit:converter-jackson
- Moshi: com.squareup.retrofit:converter-moshi
- Protobuf: com.squareup.retrofit:converter-protobuf
- Wire: com.squareup.retrofit:converter-wire
- Simple XML: com.squareup.retrofit:converter-simplexml

#####4.3.2 Rxjava
retrofit也可以与[Rxjava](https://github.com/ReactiveX/RxJava)联合起来使用，之前的版本使用范例可以参考[http://randomdotnext.com/retrofit-rxjava/](http://randomdotnext.com/retrofit-rxjava/)

- adapter-Rxjava: com.squareup.retrofit:adapter-rxjava

正在开发中，主要是通过扩展CallAdapter，将之前Call，转换为rxjava需要的Observable<?>。

###5 杂谈
  Retrofit整体框架的代码并不多，主要是围绕着converter，CallAdapter设计的整个框架。花了两天时间耐耐心心地把代码也是挺有收获。Retrofit用到的基本技术是动态代理，Java注解。另外如果对设计模式很熟悉的话，读起来感觉就会很简单。整个架构设计的非常好。


  [1]: /tool-lib/network/retrofit/images/model.png
  [2]: /tool-lib/network/retrofit/images/flow-draw.png
  [3]: /tool-lib/network/retrofit/images/retrofit-uml.png
  [4]: /tool-lib/network/retrofit/images/call-adapter-uml.png
  [5]: http://a.codekk.com/detail/Android/Caij/%E5%85%AC%E5%85%B1%E6%8A%80%E6%9C%AF%E7%82%B9%E4%B9%8B%20Java%20%E5%8A%A8%E6%80%81%E4%BB%A3%E7%90%86

