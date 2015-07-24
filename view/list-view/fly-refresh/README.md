FlyRefresh 源码解析
====================================

> 本文为 [FlyRefresh](https://github.com/android-cn/android-open-project-analysis) 中 FlyRefresh 部分  
> 项目地址：[FlyRefresh](https://github.com/race604/FlyRefresh)，分析的版本：[5299e8b](https://github.com/race604/FlyRefresh/commit/5299e8b969aab63fc1fde0a5423b19a61cded53b "Commit id is 5299e8b969aab63fc1fde0a5423b19a61cded53b]")，Demo 地址：[fly-refresh-demo]()  

> 分析者：[skyacer](http://github.com/skyacer)，分析状态：未完成，校对者：[Trinea](https://github.com/trinea)，校对状态：未开始  

##1. 功能介绍  

FlyRefresh 是一个非常漂亮的下拉刷新的框架，下拉后会有纸飞机在顶部转一圈然后如果有新的item则会增加一条。添加方法只需要在你的布局中添加 FlyRefreshLayout 即可，使用的RecyclerView实现。

###1.1 **完成时间**  

- `2015-2-12`完成  



###1.2 **集成指南**  


在 gradle 中

``` xml

dependencies {

    compile 'com.race604.flyrefresh:library:1.0.1''

}

```

### 1.3 **使用指南**

>####1.在你的布局 XML 声明一个`FlyRefreshLayout`  



>``` xml

  		<com.race604.flyrefresh.FlyRefreshLayout
  		android:id="@+id/fly_layout"
  		android:layout_width="match_parent"
  		android:layout_height="match_parent">

   		 <android.support.v7.widget.RecyclerView
    		  android:id="@+id/list"
    		  android:layout_width="match_parent"
    		  android:layout_height="match_parent"
    		  android:paddingTop="24dp"
   		   android:background="#FFFFFF"/>
		</com.race604.flyrefresh.FlyRefreshLayout>

>```





---
