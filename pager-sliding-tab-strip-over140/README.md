Android PagerSlidingTabStrip 源码解析
----------------
> 本文为 [Android 开源项目实现原理解析](https://github.com/android-cn/android-open-project-analysis) 中 PagerSlidingTabStrip 部分  
> 项目地址：[PagerSlidingTabStrip](https://github.com/astuetz/PagerSlidingTabStrip)，分析的版本：，Demo 地址：    
> 分析者：[over140](https://github.com/over140)，校对者：，完成状态：未完成   

# 简介

PagerSlidingTabStrip 是一个搭配ViewPager实现Tab滑动导航效果的控件。

![PagerSlidingTabStrip Sample Screenshot 1](https://lh3.ggpht.com/PXS7EmHhQZdT1Oa379iy91HX3ByWAQnFZAthMAFa_QHAOHNClEaXU5nxDEAj1F2eqbk)![PagerSlidingTabStrip Sample Screenshot 2](https://lh3.ggpht.com/oaksDoUcQlGB4j7VEkBCOjrvSzjtzVHHcKq8pAnGVfm6oxkcJg_w1QS4tyP3fLcqrwcX)

主要特点：
* 简单，就一个类
* 效果好

# 使用方法



# 相关属性

 * `pstsIndicatorColor` 滑动指示器的颜色
 * `pstsUnderlineColor` 设置Tab底部与下面分割的细线的颜色
 * `pstsDividerColor` Tab之前分割线的颜色
 * `pstsIndicatorHeight` 滑动指示器的高度
 * `pstsUnderlineHeight` 设置Tab底部与下面分割的细线的高度
 * `pstsDividerPadding` 设置Tab直接分割线顶部和底部的边距
 * `pstsTabPaddingLeftRight` 设置Tab的左右边距
 * `pstsScrollOffset` 选中Tab的偏移
 * `pstsTabBackground` 设置Tab的背景
 * `pstsShouldExpand` 如果设置为true，所有Tab将使用相同的weight，默认为false。
 * `pstsTextAllCaps` 如果设置为true，所有Tab的标题将使用大写，默认true。
