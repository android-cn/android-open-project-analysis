ButterKnife
=======================

# 简介

ButterKnife 是一个 View 注入库， 使用注解配置， 可以简化开发， 使代码更加优雅和易读。  

主要有两个功能

* 各种 `View` 的注入
* 各种 `事件` 的注入

带来了两个好处

* 不用再写 `findViewById()`
* 不用再写 `setOnXXListener()`

# 使用方法

## 入门实例

### 使用 `ButterKnife` 之前， 在一个 `Activity` 中你可能会写类似代码

```java
	public class SimpleActivity extends Activity implements OnClickListener,
			OnItemClickListener {

		TextView mTextView;
		Button mButton;
		ListView mListView;

		@Override
		protected void onCreate(Bundle savedInstanceState) {
			super.onCreate(savedInstanceState);
			setContentView(R.layout.activity_simple);

			// findView
			mTextView = (TextView) this.findViewById(R.id.textview);
			mButton = (Button) this.findViewById(R.id.button);
			mListView = (ListView) this.findViewById(R.id.listview);

			mButton.setOnClickListener(this);

			mListView.setOnItemClickListener(this);
		}

		@Override
		public void onClick(View v) {
			// Button click
		}

		@Override
		public void onItemClick(AdapterView<?> parent, View view, int position,
				long id) {
			// ListView item click
		}

	}
```

### 使用 `ButterKnife` 之后，你可以这么写

```java
	public class SimpleActivity extends Activity {

		@InjectView(R.id.textview)
		TextView mTextView;
		@InjectView(R.id.button)
		Button mButton;
		@InjectView(R.id.listview)
		ListView mListView;

		@Override
		protected void onCreate(Bundle savedInstanceState) {
			super.onCreate(savedInstanceState);
			setContentView(R.layout.activity_simple);
			// 开始注入
			ButterKnife.inject(this);
		}

		@OnClick(R.id.button)
		public void onClick(View v) {
			// Button click
		}

		@OnItemClick(R.id.listview)
		public void onItemClick(AdapterView<?> parent, View view, int position,
				long id) {
			// ListView item click
		}

	}
```

我们使用 **注解** 代替了繁琐的 **findView** 和 **设置事件** 的代码。

## 使用详解

### 注入类型

`View` 注入相关的注解，修饰在 **属性** 上

* @InjectView(int)
	* 修饰在一个 View **Field** 上， 参数是一个 View 的 ID
* @InjectViews(int[])
	* 修饰在一个 View数组 **Field** 上， 参数是一组 View ID

`事件` 注入相关的注解，修饰在 **方法** 上，注解参数都是 View ID 的数组。

* @OnCheckedChanged
	* View 类型：`android.widget.CompoundButton`
	* 对应的方法：`setOnCheckedChangeListener`
	* 方法参数：`(android.widget.CompoundButton, boolean)`
* @OnClick
	* View 类型：`android.view.View`
	* 对应的方法：`setOnClickListener`
	* 方法参数：`(android.view.View)`
* @OnEditorAction
	* View 类型：`android.widget.TextView`
	* 对应的方法：`setOnEditorActionListener`
	* 方法参数：`(android.widget.TextView, int, android.view.KeyEvent)`
	* 返回类型：`boolean`
* @OnFocusChange
	* View 类型：`android.view.View`
	* 对应的方法：`setOnFocusChangeListener`
	* 方法参数：`(android.view.View, boolean)`
* @OnItemClick
	* View 类型：`android.widget.AdapterView<?>`
	* 对应的方法：`setOnItemClickListener`
	* 方法参数：`(android.widget.AdapterView<?>, android.view.View, int, long)`
* @OnItemLongClick
	* View 类型：`android.widget.AdapterView<?>`
	* 对应的方法：`setOnItemLongClickListener`
	* 方法参数：`(android.widget.AdapterView<?>, android.view.View, int, long)`
	* 返回类型：`boolean`
* @OnItemSelected
	* View 类型：`android.widget.AdapterView<?>`
	* 对应的方法：`setOnItemSelectedListener`
	* `OnItemSelectedListener` 接口有两个方法，该注解有第二个参数，是枚举类型，可以指定监听哪个方法
		* Callback.ITEM_SELECTED (默认)
			* 对应方法：`onItemSelected`
			* 方法参数：`(android.widget.AdapterView<?>, android.view.View, int, long)`
		* Callback.NOTHING_SELECTED
			* 对应方法：`onNothingSelected`
			* 方法参数：`(android.widget.AdapterView<?>)`
* @OnLongClick
	* View 类型：`android.view.View`
	* 对应的方法：`setOnLongClickListener`
	* 方法参数：`(android.view.View)`
	* 返回类型：`boolean`
* @OnPageChange
	* View 类型：`android.support.v4.view.ViewPager`
	* 对应的方法：`setOnPageChangeListener`
	* `OnPageChangeListener` 接口有三个方法，该注解有第二个参数，是枚举类型，可以指定监听哪个方法
		* Callback.PAGE_SELECTED (默认)
			* 对应方法：`onPageSelected`
			* 方法参数：`(int)`
		* Callback.PAGE_SCROLLED
			* 对应方法：`onPageScrolled`
			* 方法参数：`(int, float, int)`
		* Callback.PAGE_SCROLL_STATE_CHANGED
			* 对应方法：`onPageScrollStateChanged`
			* 方法参数：`(int)`
* @OnTextChanged
	* View 类型：`android.widget.TextView`
	* 对应的方法：`addTextChangedListener`
	* `TextWatcher` 接口有三个方法，该注解有第二个参数，是枚举类型，可以指定监听哪个方法
		* Callback.TEXT_CHANGED (默认)
			* 对应方法：`onTextChanged`
			* 方法参数：`(java.lang.CharSequence, int, int, int)`
		* Callback.BEFORE_TEXT_CHANGED
			* 对应方法：`beforeTextChanged`
			* 方法参数：`(java.lang.CharSequence, int, int, int)`
		* Callback.AFTER_TEXT_CHANGED
			* 对应方法：`afterTextChanged`
			* 方法参数：`(android.text.Editable)`
* @OnTouch
	* View 类型：`android.view.View`
	* 对应的方法：`setOnTouchListener`
	* 方法参数：`(android.view.View, android.view.MotionEvent)`
	* 返回类型：`boolean`

可选类型，修饰在 Field 和 Method 上，如果不加可选类型，View ID 不存在会抛出异常。

* @Optional

### 使用场景


## 其他功能

# 原理

## 原理简析

## 流程分析