# ListViewAnimations分析
---
#功能介绍
项目主页[https://github.com/nhaarman/ListViewAnimations/](https://github.com/nhaarman/ListViewAnimations/)     
	    
正如名字所言，这是一个针对listview的动画库。通过它我们可以很方便的给listview（或grdview）增加item动画。  
该库依赖于NineOldAndroids来支持3.0以下的版本。就算不支持3.0以下版本，也需要添加。否则使用时可能会出现问题。  
可通过简单的API调用实现以下**功能**：  
1. **item展示动画**  
    * 内置动画（底部滑入动画、左边滑入动画、右边滑入动画、item渐变显示、item放大动画）  
	* 其他自定义动画等等  
2. **item操作**  
	* 滑动删除（以及删除撤销）  
	* item拖动  
	* item（可以是多个item）删除动画  
	* item展开以及动画  
	
#详细设计  
	



