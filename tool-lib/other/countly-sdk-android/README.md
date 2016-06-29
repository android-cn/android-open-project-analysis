>  项目地址：[countly-sdk-android](https://github.com/Countly/countly-sdk-android)，分析的版本：[16.02.01 Release](https://github.com/Countly/countly-sdk-android/commit/c105d06e573703d9e29d5c92068a41befa36f43f)，Demo 地址：[countly-sdk-android-demo](https://github.com/Labmem003/zhenzhi-open-project-analysis/tree/master/countly-sdk-android-demo)    
 分析者：[振之](https://github.com/Labmem003)，分析状态：完成

### 1.功能介绍  
#### Countly
Countly 是一款类似于友盟的移动&Web应用通用的实时统计分析系统，专注于易用性、扩展性和功能丰富程度。不同之处是 Countly 是开源的，任何人都可以将 Countly 客户端部署在自己的服务器并将开发工具包整合到他们的应用程序中。比友盟要简洁干净，关键是数据和程序都完全处于自己掌控之下，不愿被第三方掌握数据，或者有什么特殊需求的，可以自己满足自己了。

这里分析的是其 Android 端的 sdk, 以了解和学习移动应用统计类的工具收集 App 的使用情况和终端用户的行为的机制。主要功能包括 App 基本数据收集、自定义事件记录、崩溃报告。
#### 基本使用
（1）将 Countly SDK 添加到您的项目
Android Studio
添加二进制 Maven 存储库：

```
buildscript {
    repositories {
        maven {
            url  "http://dl.bintray.com/countly/maven"
        }
    }
}
```
添加 Countly SDK 依赖：

```
dependencies {
    compile 'ly.count.android:sdk:15.06'
}
```
当然也可以直接下载 Jar 使用, 这里分析的是 [sdk-16.02.01.jar](https://github.com/Countly/countly-sdk-android/releases/download/16.02.01/sdk-16.02.01.jar)。

（2）服务器端程序的安装和使用
篇幅和重点所限，不详细介绍服务端安装使用，在[ Countly 体验](https://cloud.count.ly)创建一个应用试用即可。

（3）具体使用

设置 SDK，AppKey 在（2）的“管理－应用”中查看: 

```
  Countly.sharedInstance().init(this, "https://YOUR_SERVER", "YOUR_APP_KEY");
```
记录事件：

```
  Countly.sharedInstance().recordEvent("purchase", 1);
```

设置崩溃报告：

```
  Countly.sharedInstance().enableCrashReporting()
```
更详细的使用，可参照我写的小 [Demo](https://github.com/Labmem003/zhenzhi-open-project-analysis/tree/master/countly-sdk-android-demo)。
### 2.总体设计

[![总体设计](http://blog.qiji.tech/wp-content/uploads/2016/06/CountlyDesign.png)](http://blog.qiji.tech/wp-content/uploads/2016/06/CountlyDesign.png)

上面是 Countly SDK 的总体设计图。

SDK 主要处理 Event、Crash 和会话流（Session）3种数据记录请求。其中 Crash 和 Session 自动记录，并作为 Connection 持久存储到ConnectionQueue, 等待提交到服务器；Event 则由开发者调用，并配有一个 EventQueue 存储，但是在上报给服务器的时候依然是通过加入到 ConnectionQueue。也就是说，所有请求，最后都是 Connection。
ConnectionQueue 和 EventQueue 不是平常意义的 FIFO 队列，而是本地存储队列。包装了基于 SharePreference 实现的持久层 Store，每个请求会被字符串化，加上分隔符，添加到对应的SP键值后面。

最终存储在SP的 ConnectionQueue，大概长这样：

```
  "app_key=appKey_×tamp=3482759874&hour=6&dow=2&session_duration=24&location=3，8:::app_key=appKey_×tamp=345567773&hour=8&dow=3&session_duration=12&location=3，8"
```
OK, 接口地址知道，数据在手, 取出来按接口要求拼装好，fire the hole。

### 3.流程图

[![流程图](http://blog.qiji.tech/wp-content/uploads/2016/06/CountlyFlowchartDiagram.png)](http://blog.qiji.tech/wp-content/uploads/2016/06/CountlyFlowchartDiagram.png)

### 4. 详细设计
#### 4.1 类关系图
[![classship](http://blog.qiji.tech/wp-content/uploads/2016/06/classship.png)](http://blog.qiji.tech/wp-content/uploads/2016/06/classship.png)  
#### 4.2 类详细介绍
结构很简单，一共就两个包，countly 核心包和 openudid 包。

[![structure](http://blog.qiji.tech/wp-content/uploads/2016/06/structure.png)](http://blog.qiji.tech/wp-content/uploads/2016/06/structure.png)

countly 包解决统计什么，怎么实施统计；而 openudid 包解决如何标记统计的数据来自何方。

##### 4.2.1 openudid 包
先来看看比较简单的 openudid 包，她是一个设备标识方案，能提供一个设备通用统一标识符（Unique Device IDentifier/UDID）。如果同一台设备上有多个 App 都用了这个包来生成 UDID，他们获取的 UDID 是一致的（即所谓设备标识）。当我们将统计数据发送给服务端时，会将 UDID 附上。不难想到，之后服务端算日活、描述用户特征、事件追踪等等各种后续的数据分析肯定都离不开 UDID ，算是很必要的基础设施。实际上她也是一个开源包[OpenUDID](https://github.com/vieux/OpenUDID)。
###### 4.2.1.1 OpenUDID_service.java
这个类很简单，就是一个只重写了 onTransact 方法的 Service，支持跨进程调用。

```
  public class OpenUDID_service extends Service {
    @Override public IBinder onBind(Intent arg0) {
      return new Binder() {
        @Override
        public boolean onTransact(int code, android.os.Parcel data, android.os.Parcel reply,
            int flags) {
          final SharedPreferences preferences =
              getSharedPreferences(OpenUDID_manager.PREFS_NAME, Context.MODE_PRIVATE);

          reply.writeInt(data.readInt()); //Return to the sender the input random number
          reply.writeString(preferences.getString(OpenUDID_manager.PREF_KEY, null));
          return true;
        }
      };
    }
  }
```
它给调用者返回了两个值：

（1）把调用者传过来的 data 中的 int 参数又传回去了，起的是标识发起者的作用，类似于 startActivityForResult() 中的 requestCode。

```
reply.writeInt(data.readInt()); //Return to the sender the input random number
```

（2）返回本进程获取到的 OpenUDID。还可以看到 OpenUDID 是用 SharePreferences 保存的。

```
  reply.writeString(preferences.getString(OpenUDID_manager.PREF_KEY, null));

```
###### 4.2.1.2 OpenUDID_manager.java
调用 sync(Context context) 来初始化一个 udid, 策略是先看自己有木有，木有的话就从好基友那里拿，好基友也没有就只能自己撸一个出来。

一切都撸完之后，调用 getOpenUDID 就可以得到一枚 UDID 了。

（1）public static void sync(Context context)
 
```
  //先尝试从本地 SharePreference 中获取
  OpenUDID=manager.mPreferences.getString(PREF_KEY,null);
```

```
    if (OpenUDID == null) //本地没有
    {
      //获取设备上所有的使用了该包的 OpenUDID_service 的 services 列表, intent 形式保存在 mMatchingIntents
      manager.mMatchingIntents =
          context.getPackageManager().queryIntentServices(new Intent("org.OpenUDID.GETUDID"), 0);

      if (manager.mMatchingIntents != null)
      //尝试从别的使用了 OpenUDID_service 的进程获取
      {
        manager.startService();
      }
    } else {//本地存在 UDID, 初始化完成，即可以直接调用 getOpenUDID() 来获取了
      mInitialized = true;
    }
```



(2)startService()
启动 mMatchingIntents 的首个 intent 并移除；


```
   private void startService() {
    if (mMatchingIntents.size() > 0) { //There are some Intents untested
      if (LOG) {
        Log.d(TAG,
            "Trying service " + mMatchingIntents.get(0).loadLabel(mContext.getPackageManager()));
      }

      final ServiceInfo servInfo = mMatchingIntents.get(0).serviceInfo;
      final Intent i = new Intent();
      i.setComponent(new ComponentName(servInfo.applicationInfo.packageName, servInfo.name));
      mMatchingIntents.remove(0);
      try {  // try added by Lionscribe
        mContext.bindService(i, this, Context.BIND_AUTO_CREATE);
      } catch (SecurityException e) {
        startService();  // ignore this one, and start next one
      }
    } else { //No more service to test

      getMostFrequentOpenUDID(); //Choose the most frequent

      if (OpenUDID == null) //No OpenUDID was chosen, generate one			
      {
        generateOpenUDID();
      }
      if (LOG) Log.d(TAG, "OpenUDID: " + OpenUDID);

      storeOpenUDID();//Store it locally
      mInitialized = true;
    }
  }
```

若启动成功则可以拿到被启动进程的 UDID, 以 UDID 为键，次数为值存入 HashMap 中；然后再次 startService(); 结果是递归地启动了 mMatchingIntents 的所有 intent，得到一张记录着各个进程的不相同的 udid 及其次数的 Map。
	

```
  public void onServiceConnected(ComponentName className, IBinder service) {
    //Get the OpenUDID from the remote service
    ...
    final String _openUDID = reply.readString();
    ...

    if (mReceivedOpenUDIDs.containsKey(_openUDID)) {
      mReceivedOpenUDIDs.put(_openUDID, mReceivedOpenUDIDs.get(_openUDID) + 1);
    } else {
      mReceivedOpenUDIDs.put(_openUDID, 1);
    }
    ...
    startService(); //Try the next one
  }
```

（3）private void getMostFrequentOpenUDID() ，返回 Map 中次数最多的 UDID

（4）private void generateOpenUDID() ，没有从其他进程领到 UDID，就生成一个

 
```
  /*
  * Generate a new OpenUDID
  */
  private void generateOpenUDID() {
    if (LOG) Log.d(TAG, "Generating openUDID");
    //Try to get the ANDROID_ID
    OpenUDID = Secure.getString(mContext.getContentResolver(), Secure.ANDROID_ID);
    if (OpenUDID == null || OpenUDID.equals("9774d56d682e549c") || OpenUDID.length() < 15) {
      //if ANDROID_ID is null, or it's equals to the GalaxyTab generic ANDROID_ID or bad, generates a new one
      final SecureRandom random = new SecureRandom();
      OpenUDID = new BigInteger(64, random).toString(16);
    }
  }
```  

直接使用了 Android ID 来做 UDID，其实有点不靠谱。因为如果你恢复了出厂设置，那他就会改变的。而且如果你 root 了手机，你也可以改变这个 ID。不过如果不需要太精确的统计，也够用了。看你的需求吧。

可以参考[这篇文章](http://www.bkjia.com/Androidjc/1036506.html)来修改适合你的 UDID。

(5)public static String getOpenUDID(),就是getOpenUDID。

##### 4.2.2 countly 包
概念解释

Event，事件，以键值方式记录，键为事件名，值记录次数。

Session，会话，定时更新；形成的会话流，代表应用的一次使用过程。

Crash，崩溃。

Connection，连接；以上请求（事件、会话、崩溃）都会转换为 Connection 提交给服务器。
###### 4.2.2.1 OpenUDIDAdapter.java
包装了 UDID 包，提供 sync（），getOpenUDID（）。但是是用动态反射的方法封装的，不明白为什么。官方的 commit message 说了一句：call OpenUDID dynamically so that including the OpenUDID source is not necessary to get the Countly Android SDK to work when an app provides it's own deviceID。
看懂的请告诉我。
###### 4.2.2.2DeviceId.java
代表设备 ID 的类。 

（1）主要属性是

private String id;//设备标识ID

private Type type;//ID类型

（2）ID类型分为3种

```
  public static enum Type {
    DEVELOPER_SUPPLIED,//开发者自行定义和提供
    OPEN_UDID,//openUDID
    ADVERTISING_ID,//谷歌广告平台设备标识符
  }
```
openUDID 前面已介绍过，其他两种也是类似，不累述。
###### 4.2.2.3 DeviceInfo.java
一个纯 POJO 类，用来存放设备信息，如设备名称、设备分辨率、版本号等。

主要方法：
static String getMetrics(final Context context)，返回 url-encoded 的属性 json 字符串。
###### 4.2.2.4 CrashDetails.java
提供了一些静态方法来获取运行时环境信息，结合 DeviceInfo 类, 为 Crash 时提供详细的参考信息。

主要方法：

```
  static String getCrashData(final Context context, String error, Boolean nonfatal) {
    final JSONObject json = new JSONObject();

    fillJSONIfValuesNotEmpty(json, "_error", error, "_nonfatal", Boolean.toString(nonfatal),
        "_logs", getLogs(), "_device", DeviceInfo.getDevice(), "_os", DeviceInfo.getOS(),
        "_os_version", DeviceInfo.getOSVersion(), "_resolution", DeviceInfo.getResolution(context),
        "_app_version", DeviceInfo.getAppVersion(context), "_manufacture", getManufacturer(),
        "_cpu", getCpu(), "_opengl", getOpenGL(context), "_ram_current", getRamCurrent(context),
        "_ram_total", getRamTotal(context), "_disk_current", getDiskCurrent(), "_disk_total",
        getDiskTotal(), "_bat", getBatteryLevel(context), "_run", getRunningTime(), "_orientation",
        getOrientation(context), "_root", isRooted(), "_online", isOnline(context), "_muted",
        isMuted(context), "_background", isInBackground());

    ...
    json.put("_custom", getCustomSegments());
    ...
    String result = json.toString();

    ...
    result = java.net.URLEncoder.encode(result, "UTF-8");
    ...

    return result;
  }
```
###### 4.2.2.5 UserData.java
类似 CrashDetail。

```
  /*
   *Send provided values to server
   */
  public void save() {
    connectionQueue_.sendUserData();
    UserData.clear();
  }
```
###### 4.2.2.6 Event.java
定义了一个事件的数据结构

```
  public String key;//键，识别事件
  public int count;//发生此事件的次数
  public double sum;//事件的全部数值数据，比如一次支付事件的支付金额，可选
  public Map<String, String> segmentation;//分段键值对，用来扩展自定义数据，数量不受限制
```

由于多个事件可结合在单一请求中，为了正确报告和处理数据（特别是排队数据），还有下面3个属性用来提供数据记录时间：

```
  public int timestamp;//时间戳
  public int hour;//本地时间，0-23
  public int dow;//星期几

  JSONObject toJSON()
  static Event fromJSON(final JSONObject json)
```

还有 fromJson 和 toJson 函数来在事件对象和 json 表示之间转换。
###### 4.2.2.7 EventQueue.java
这个类用来队列化 event，并且可以将 event 转化为 json，方便提交到服务器。

（1）主要属性：

```
  private final CountlyStore countlyStore_;
```
CountlyStore 是一个持久化存储类，EventQueue 类其实就是对 CountlyStore 的一个封装，每次有 Event 添加进 queue，就会通过 CountlyStore 直接持久化存储到本地（本地队列的末尾）；出 queue 的时候也是直接从本地队列中移除。CountlyStore 在后面还会详细介绍。

```
  /**
   * Constructs an EventQueue.
   *
   * @param countlyStore backing store to be used for local event queue persistence
   */
  EventQueue(final CountlyStore countlyStore) {
    countlyStore_ = countlyStore;
  }
```

（2）主要方法：
并不是真正意义上的 queue，使用 recordEvent 入队一个 Event 到本地，使用 events() 直接把整个队列提取出来并且转换为 urlEncoded的json串，可以直接用于提交给服务器。好处是合并请求，方便一次提交多个 event，当然也是因为配合 api。

```
  String events() {
    String result;
    //从CountlyStore取回EventQueue
    final List<Event> events = countlyStore_.eventsList();

    //Json化
    final JSONArray eventArray = new JSONArray();
    for (Event e : events) {
      eventArray.put(e.toJSON());
    }

    result = eventArray.toString();

    countlyStore_.removeEvents(events);

    //UrlEncode
    try {
      result = java.net.URLEncoder.encode(result, "UTF-8");
    } catch (UnsupportedEncodingException e) {
      // should never happen because Android guarantees UTF-8 support
    }

    return result;
  }
```

```
  void recordEvent(final String key, final Map<String, String> segmentation, final int count,
      final double sum) {
    ...
    //直接持久化存入本地 Event 队列
    countlyStore_.addEvent(key, segmentation, timestamp, hour, dow, count, sum);
  }
```
###### 4.2.2.8 CountlyStore.java
该类使用 SharePreference 为 Event, Connection, Location 3类数据提供持久化服务.
介绍存储方式。

以 Event 为例，每次有新来的 Event，会先读取本地的 events, 把新来的 event 加到末尾，然后整个队列重新被 json 化存储到 Shareference。

```
  void addEvent(final Event event) {
    final List<Event> events = eventsList();
    events.add(event);
    preferences_.edit().putString(EVENTS_PREFERENCE, joinEvents(events, DELIMITER)).commit();
  }
```

直接从 SharePreference 取出所有 json 表示的 events。

```
  public String[] events()
```

将 json 表示的 events 反序列化，并排序，返回。

```
  public List<Event> eventsList() {
    final String[] array = events();
    final List<Event> events = new ArrayList<>(array.length);
    for (String s : array) {
      try {
        final Event event = Event.fromJSON(new JSONObject(s));
        if (event != null) {
          events.add(event);
        }
      } catch (JSONException ignored) {
        // should not happen since JSONObject is being constructed from previously stringified JSONObject
        // events -> json objects -> json strings -> storage -> json strings -> here
      }
    }
    // order the events from least to most recent
    Collections.sort(events, new Comparator<Event>() {
      @Override public int compare(final Event e1, final Event e2) {
        return e1.timestamp - e2.timestamp;
      }
    });
    return events;
  }
```

```
   public String[] connections()
   public String[] events()
   public List<Event> eventsList()
```

###### 4.2.2.9 Countly.java
暴露接口和驱动各个类工作的入口类。主要的属性是
EventQueue，ConnectionQueue，和ScheduledExecutorService。
（1）构造和 init
调用其他接口前需先调用 init() 来初始化，主要是参数检验、初始化和配置 EventQueue，ConnectionQueue。构造的时候就开启了一个 1 分钟 1 次的定时任务。

```
  Countly() {
    connectionQueue_ = new ConnectionQueue();
    Countly.userData = new UserData(connectionQueue_);
    timerService_ = Executors.newSingleThreadScheduledExecutor();
    timerService_.scheduleWithFixedDelay(new Runnable() {
      @Override public void run() {
        onTimer();
      }
    }, TIMER_DELAY_IN_SECONDS, TIMER_DELAY_IN_SECONDS, TimeUnit.SECONDS);
  }

  ...
      deviceIdInstance.init(context,countlyStore,true);

  connectionQueue_.setServerURL(serverURL);
  connectionQueue_.setAppKey(appKey);
  connectionQueue_.setCountlyStore(countlyStore);
  connectionQueue_.setDeviceId(deviceIdInstance);

  eventQueue_=new

  EventQueue(countlyStore);

  ...
```

（2）定时任务
记录 session 和 event 到 connectionqueue，并触发发送数据行为，所有的数据都是从 connectionqueue 的 store 中取得。

```
  synchronized void onTimer() {
    final boolean hasActiveSession = activityCount_ > 0;
    if (hasActiveSession) {
      if (!disableUpdateSessionRequests_) {
        connectionQueue_.updateSession(roundedSecondsSinceLastSessionDurationUpdate());
      }
      if (eventQueue_.size() > 0) {
        connectionQueue_.recordEvents(eventQueue_.events());
      }
    }
  }
```

(3) Session 流开始和退出，利用 activityCount
Android 没有很好的办法监听应用开始和结束，所以每个 Activity 都需要调用 Countly.shareInstance().onStart() 和 onStop 方法，方法内部用一个 int 变量 activityCount_ 来记录当前 Activity 数量。onStart时activityCount_＋1, onStop 时 －1。

onStart 中，当 activityCount_＝＝1 时，应用启动， 开始 Session 流；

onStop 中，当 activityCount_＝＝0 时，应用退出前台， 结束 Session 流。

(4)异常捕获,使用 UncaughtExceptionHandler

```
   /**
   * Enable crash reporting to send unhandled crash reports to server
   */
  public synchronized Countly enableCrashReporting() {
    //get default handler
    final Thread.UncaughtExceptionHandler oldHandler = Thread.getDefaultUncaughtExceptionHandler();

    Thread.UncaughtExceptionHandler handler = new Thread.UncaughtExceptionHandler() {

      @Override public void uncaughtException(Thread t, Throwable e) {
        StringWriter sw = new StringWriter();
        PrintWriter pw = new PrintWriter(sw);
        e.printStackTrace(pw);
        Countly.sharedInstance().connectionQueue_.sendCrashReport(sw.toString(), false);

        //if there was another handler before
        if (oldHandler != null) {
          //notify it also
          oldHandler.uncaughtException(t, e);
        }
      }
    };

    Thread.setDefaultUncaughtExceptionHandler(handler);
    return this;
  }
```
（5）记录事件

事件先被记录在 eventQueue, 到了需要被发送的时候会被全部取出放入 connectionQueue。connectionQueue 有发送数据的能力，系统定时把 connectionQueue 中的多条数据合并发送到服务器。

```
  public synchronized void recordEvent(final String key, final Map<String, String> segmentation,
      final int count, final double sum) {
    ...
    eventQueue_.recordEvent(key, segmentation, count, sum);
    sendEventsIfNeeded();
  }

  void sendEventsIfNeeded() {
    if (eventQueue_.size() >= EVENT_QUEUE_SIZE_THRESHOLD) {
      connectionQueue_.recordEvents(eventQueue_.events());
    }
  }
```

###### 4.2.2.10 ConnectionQueue.java
需要被发送的各种数据，包括前面说过的 event 和 crash 等，都在此提供发送接口，实际发送的时候都会被转换为 connection,持久化添加到connectionQueue 中，Tick 的时候由 ConnectionProcessor 从 Store 中取出发送到服务端。

```
  //该方法会被定时触发
  void recordEvents(final String events) {
    checkInternalState();
    final String data = "app_key="
        + appKey_
        + "×tamp="
        + Countly.currentTimestamp()
        + "&hour="
        + Countly.currentHour()
        + "&dow="
        + Countly.currentDayOfWeek()
        + "&events="
        + events;

    store_.addConnection(data);

    tick();
  }
```
tick，持久层有待发送连接且当前没有正在提交数据，则启动一个 ConnectionProcessor 来提交数据。

```
  void tick() {
    if (!store_.isEmptyConnections() && (connectionProcessorFuture_ == null
        || connectionProcessorFuture_.isDone())) {
      ensureExecutor();
      connectionProcessorFuture_ =
          executor_.submit(new ConnectionProcessor(serverURL_, store_, deviceId_, sslContext_));
    }
  }
```
其他数据也跟 Events 类似，包括 session, location, userData, CrashReport

```
  void beginSession()
  void updateSession(final int duration)
  void endSession(final int duration)

  void sendCrashReport(String error, boolean nonfatal)
  void sendUserData()
```
###### 4.2.2.11 ConnectionProcessor.java
是个 Runnable，每次被 Run 的时候，从 Store 中取出当前所有的 Connections，用 http 发送。

```
URLConnection urlConnectionForEventData(final String eventData)
```
拼装 url 并打开 Conn

```
  public void run() {
    while (true) {
      ...
      conn = urlConnectionForEventData(eventData);
      conn.connect();
    }
    // consume response stream
    responseStream = new BufferedInputStream(conn.getInputStream());
    final ByteArrayOutputStream responseData = new ByteArrayOutputStream(
        256); // big enough to handle success response without reallocating
    int c;
    while ((c = responseStream.read()) != -1) {
      responseData.write(c);
    }
    ...
  }
```
从 Store 中取出数据，调用 urlConnectionForEventData（）生成 conn, 发起请求。
