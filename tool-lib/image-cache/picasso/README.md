Picasso 源码分析
====================================
> 本文为 [Android 开源项目源码解析](https://github.com/android-cn/android-open-project-analysis) 中 Picasso 部分  
> 项目地址：[Picasso](https://github.com/square/picasso)，分析的版本：[10bac5b](https://github.com/square/picasso/commit/10bac5ba59c7379eb4be4a2e7af074edc4bd9200)，Demo 地址：[Picasso Demo](https://github.com/aosp-exchange-group/android-open-project-demo/tree/master/picasso-demo)  
> 分析者：[Trinea](https://github.com/Trinea)，分析状态：进行中。校对者：，校对状态：未开始

###1. 功能介绍
Picasso 是 Square 开源的图片缓存库，主要特点有：  
* 包含内存缓存和磁盘缓存两级缓存。  
* 在 Adapter 中自动处理 ImageView 的缓存并且取消之前的图片下载任务。  
* 方便进行图片转换处理。  
