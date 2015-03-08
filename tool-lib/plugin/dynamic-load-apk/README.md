DynamicLoadApk 源码解析
====================================
> 本文为 [Android 开源项目源码解析](https://github.com/android-cn/android-open-project-analysis) 中 DynamicLoadApk 部分  
> 项目地址：[DynamicLoadApk](https://github.com/singwhatiwanna/dynamic-load-apk)，分析的版本：[354fc6c](https://github.com/singwhatiwanna/dynamic-load-apk/commit/354fc6c3d9ab2f55945096d81621f936f49a18e3 "Commit id is 354fc6c3d9ab2f55945096d81621f936f49a18e3")，Demo 地址：[DynamicLoadApk Demo](https://github.com/android-cn/android-open-project-demo/tree/master/dynamic-load-apk-demo)    
> 分析者：[FFish](https://github.com/FFish)，分析状态：未完成，校对者：[Trinea](https://github.com/trinea)，校对状态：未开始   

###1. 功能介绍  
DynamicLoadApk是实现Android App插件化开发的一个开源框架。它提供了3种开发方式，让开发者在无需理解其工作原理的情况下快速的集成插件化功能。

1. 宿主程序与插件完全独立 
2. 宿主程序开放部分接口供插件与之通信 
3. 宿主程序耦合插件的部分业务逻辑 

###2. 总体设计
![总体设计图](image/design.png)   

###3. 流程图
![流程图](image/flow.png)

###4. 详细设计 
####4.1 类关系图
![类关系图](image/classes.jpg)
####4.2 类功能介绍
#####4.2.1 DLPluginManager.java
该类为DynamicLoadApk的核心类，负责插件Apk的加载与维护，同时还兼具调起插件Apk中的`Activity`的任务。  
**主要方法**  
```java
public DLPluginPackage loadApk(String dexPath);

private DexClassLoader createDexClassLoader(String dexPath);

private AssetManager createAssetManager(String dexPath);

private Resources createResources(AssetManager assetManager);

public int startPluginActivity(Context context, DLIntent dlIntent);

public int startPluginActivityForResult(Context context, DLIntent dlIntent, int requestCode);
```
第一个方法为插件化功能的起点，用于将特定路径下的插件Apk加载到内存中。在实际使用时，一般是将插件Apk集中存放到某一文件夹下面，通过循环调用该方法将若干个Apk加载进内存。这里要注意的是，**该方法只能被宿主Apk调用**。
每当加载进来一个插件Apk，都会调用第2,3,4三个方法来生成相应的`DexClassLoader`和资源管理类，从而实现对插件Apk代码和资源的访问。这里有一点技巧的地方在于`AssetManager`的创建。
```java
    private AssetManager createAssetManager(String dexPath) {
        try {
            AssetManager assetManager = AssetManager.class.newInstance();
            Method addAssetPath = assetManager.getClass().getMethod("addAssetPath", String.class);
            addAssetPath.invoke(assetManager, dexPath);
            return assetManager;
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }

    }
```
在Android中，大多数的资源是通过R文件来访问的。但是实现插件化之后，宿主Apk是无法通过R文件访问插件Apk中的资源的。所以这里使用反射来生成属于插件Apk的`AssetManager`。而使用反射的原因在于`addAssetPath`不是一个公开的api
```java
    /** 
     * Add an additional set of assets to the asset manager.  This can be 
     * either a directory or ZIP file.  Not for use by applications.  Returns 
     * the cookie of the added asset, or 0 on failure. 
     * {@hide} 
     */  
    public final int addAssetPath(String path) {  
        int res = addAssetPathNative(path);  
        return res;  
    }
```
除了`loadApk()`之外，这个类对外提供的主要api还有`startPluginActivity()`和`startPluginActivityForResult()`，主要用于调起插件Apk中的`Activity`。使用方法与理解并无难点，这里不再赘述。
#####4.2.2 DLPluginPackage
该类为插件Apk对应的实体类，主要封装了下面一些信息
```java
public DexClassLoader classLoader;
    
public AssetManager assetManager;
    
public Resources resources;
    
public PackageInfo packageInfo;

public String packageName;
    
private String mDefaultActivity;
    
public String path;
```
其中前4个成员变量封装了插件Apk的主要信息，并且会在Apk被加载进来的时候(通过`DLPluginManager`的`loadApk()`)完成初始化。每当一个`DLPluginPackage`生成，`DLPluginManager`就会将其存入自己的一个`HashMap`成员变量中。
#####4.2.3 DLPlugin.java
这是一个接口，插件中的`Activity`通过实现这个接口来模拟`Activity`生命周期。接口中包含的方法签名与`Activity`生命周期方法类似。
```java
public interface DLPlugin {
    public void onCreate(Bundle savedInstanceState);
    public void onStart();
    public void onRestart();
    public void onActivityResult(int requestCode, int resultCode, Intent data);
    public void onResume();
    public void onPause();
    public void onStop();
    public void onDestroy();
    public void attach(Activity proxyActivity, DLPluginPackage pluginPackage);
    public void onSaveInstanceState(Bundle outState);
    public void onNewIntent(Intent intent);
    public void onRestoreInstanceState(Bundle savedInstanceState);
    public boolean onTouchEvent(MotionEvent event);
    public boolean onKeyUp(int keyCode, KeyEvent event);
    public void onWindowAttributesChanged(LayoutParams params);
    public void onWindowFocusChanged(boolean hasFocus);
    public void onBackPressed();
    public boolean onCreateOptionsMenu(Menu menu);
    public boolean onOptionsItemSelected(MenuItem item);
}
```
#####4.2.4 DLProxyActivity.java/DLProxyFragmentActivity.java
这两个类大同小异，所以这里只分析`DLBaseProxyActivity`。首先来看下它的成员变量。
**(1). 成员变量**
```java
protected DLPlugin mRemoteActivity;
    
private DLProxyImpl impl = new DLProxyImpl(this);

private DLPluginManager mPluginManager;
```
这个类总共有3个成员变量，其中`DLProxyImpl`是不同类型的代理`Activity`的公共代码。下一节会详细介绍。而`DLPluginManager`上面已经说过，是一个插件Apk的加载和维护类。所以这节主要来分析一下`DLPlugin`。
因为插件中的`Activity`并没有在宿主的`AndroidManifest.xml`中注册，所以在运行时，程序不会管理这些`Activity`的生命周期。所以需要规范一个接口，通过接口来模拟`Activity`的生命周期。具体做法如下。
```java
    @Override
    protected void onResume() {
        mRemoteActivity.onResume();
        super.onResume();
    }
    
    @Override
    protected void onDestroy() {
        mRemoteActivity.onDestroy();
        super.onDestroy();
    }
```
可以看出，通过在特定的生命周期中调用`DLPlugin`中的方法，来实现`Activity`生命周期的模拟功能。
**(2). DLProxyActivity.java/DLProxyFragmentActivity.java在DynamicLoadApk框架中的角色**
其实通过名字就可以知道，`DLProxyActivity`就是一个代理`Activity`。具体做法是，先将`DLProxyActivity`注册进宿主的`AndroidManifest.xml`，然后将每个生命周期对应的业务代码写在插件中的`Activity`(实现了`Plugin`接口)。

#####4.2.5 DLBasePluginActivity.java/DLBasePluginFragmentActivity.java


###5. 杂谈
