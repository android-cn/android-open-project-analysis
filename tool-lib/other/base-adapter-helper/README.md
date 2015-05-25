base-adapter-helper 源码分析
====================================
> 本文为 [Android 开源项目源码解析](https://github.com/android-cn/android-open-project-analysis) 中 base-adapter-helper 部分  
> 项目地址：[base-adapter-helper](https://github.com/JoanZapata/base-adapter-helper)，分析的版本：[e65d7d8](https://github.com/JoanZapata/base-adapter-helper/commit/e65d7d83c5f5181feb189e5ff4f5cc5835eaadfe "Commit id is e65d7d83c5f5181feb189e5ff4f5cc5835eaadfe")，Demo 地址：[base-adapter-helper Demo](https://github.com/aosp-exchange-group/android-open-project-demo/tree/master/base-adapter-helper-demo)     
> 分析者：[hongyangAndroid](https://github.com/hongyangAndroid)

###1. 功能介绍  
####1.1. base-adapter-helper  
base-adapter-helper 是对我们传统的BaseAdapter的ViewHolder的模式的一个封装。主要功能就是简化我们在书写AbsListView，例如ListView,GridView的Adapter的代码。

####1.2 基本使用
```java
 mListView.setAdapter(mAdapter = new QuickAdapter<Bean>(
			MainActivity.this, R.layout.item_list, mDatas)
	{

		@Override
		protected void convert(BaseAdapterHelper helper, Bean item)
		{
			helper.setText(R.id.tv_title, item.getTitle())
                 .setImageUrl(R.id.id_icon, item.getUrl())
			     .setText(R.id.tv_describe, item.getDesc())
			     .setText(R.id.tv_phone, item.getPhone())
			     .setText(R.id.tv_time, item.getTime());
		}
	});
```

####1.3 特点
1. 提供QucikAdapter，省去写类似getCount()等样板代码，只需关注Model到View的显示。
2. BaseAdapterHelper中封装了大量用于为View操作的辅助方法，例如从网络加载图片：
`helper.setImageUrl(R.id.iv_photo, item.getPhotoUrl());`

###2. 总体设计
由于base-adapter-helper本质上仍然是ViewHolder Pattern，下面贴出base-adapter-helper的总体设计图和ViewHolder Pattern的设计图，通过两图的比较，可以看出base-adapter-helper对传统的`BaseAdapter`进行了初步的实现（`QuickAdapter`），并且仅公布出`convert()`方法，在`convert()`中可以拿到`BaseAdapterHelper`,`BaseAdapterHelper`就相当于`ViewHolder`，但其内部提供了大量的辅助方法，用于设置View上的数据，甚至是事件等。

##### base-adapter-helpr
![base-adapter-helpr设计图](image/base-adapter-helpr.png)  
##### ViewHolder Pattern
![ViewHolder Pattern](image/view_holder_pattern.png)  



###3. 详细设计
####3.1 类关系图
![类关系图](image/base-adapter-helper-ClassDiagram.jpg)  
这是 base-adapter-helper 框架的主要类关系图    

1. 在BaseQucikAdapter中实现了BaseAdapter中通用的抽象方法
2. BaseQuickAdapter中两个泛型，一个T是针对数据，一个H是针对BaseAdapterHelper
3. QucikAdapter继承自BaseQuickAdapter，并且传入BaseAdapterHelper作为H泛型
4. EnhancedQuickAdapter 主要为convert方法，添加一个itemChanged参数，用于区分 dataset changed / dataset invalidated
5. BaseAdapterHelper中封装了常用View的赋值，以及事件监听的方法，方便操作。并且赋值方法都有采用链式编程，更加方便书写。
6. 扩展BaseAdapterHelper可以继承BaseAdapterHelper，编写Adapter时继承BaseQuickAdapter,传入自定义的类作为H泛型。 

###3.2 核心类源码分析
####3.2.1 BaseQucikAdapter.java 
该类继承自BaseAdapter，完成BaseAdapter中部分通用抽象方法的编写，类似`ArrayAdapter`.
该类声明了两个泛型，一个是我们的Bean（T），一个是`BaseAdapterHelper(H)`主要用于扩展`BaseAdapterHelper`时使用。

#####(1).构造方法 
```java

    public BaseQuickAdapter(Context context, int layoutResId) {
        this(context, layoutResId, null);
    }
    public BaseQuickAdapter(Context context, int layoutResId, List<T> data) {
        this.data = data == null ? new ArrayList<T>() : new ArrayList<T>(data);
        this.context = context;
        this.layoutResId = layoutResId;
    }
```
Adapter的必须元素ItemView通过layoutResId指定，展示数据通过data指定。

#####(2).BaseAdapter中需要实现的方法
```java
    @Override
    public int getCount() {
        int extra = displayIndeterminateProgress ? 1 : 0;
        return data.size() + extra;
    }

    @Override
    public T getItem(int position) {
        if (position >= data.size()) return null;
        return data.get(position);
    }

    @Override
    public long getItemId(int position) {
        return position;
    }

    @Override
    public int getViewTypeCount() {
        return 2;
    }

    @Override
    public int getItemViewType(int position) {
        return position >= data.size() ? 1 : 0;
    }

 	@Override
    public boolean isEnabled(int position) {
        return position < data.size();
    }

    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
        if (getItemViewType(position) == 0) {
            final H helper = getAdapterHelper(position, convertView, parent);
            T item = getItem(position);
            helper.setAssociatedObject(item);
            convert(helper, item);
            return helper.getView();
        }

        return createIndeterminateProgressView(convertView, parent);
    }

    protected abstract void convert(H helper, T item);

    protected abstract H getAdapterHelper(int position, View convertView, ViewGroup parent);

    private View createIndeterminateProgressView(View convertView, ViewGroup parent) {
        if (convertView == null) {
            FrameLayout container = new FrameLayout(context);
            container.setForegroundGravity(Gravity.CENTER);
            ProgressBar progress = new ProgressBar(context);
            container.addView(progress);
            convertView = container;
        }
        return convertView;
    }

 	public void showIndeterminateProgress(boolean display) {
        if (display == displayIndeterminateProgress) return;
        displayIndeterminateProgress = display;
        notifyDataSetChanged();
    }
    public void add(T elem) {
        data.add(elem);
        notifyDataSetChanged();
    }

    public void addAll(List<T> elem) {
        data.addAll(elem);
        notifyDataSetChanged();
    }

    public void set(T oldElem, T newElem) {
        set(data.indexOf(oldElem), newElem);
    }

    public void set(int index, T elem) {
        data.set(index, elem);
        notifyDataSetChanged();
    }

    public void remove(T elem) {
        data.remove(elem);
        notifyDataSetChanged();
    }

    public void remove(int index) {
        data.remove(index);
        notifyDataSetChanged();
    }

    public void replaceAll(List<T> elem) {
        data.clear();
        data.addAll(elem);
        notifyDataSetChanged();
    }

    public boolean contains(T elem) {
        return data.contains(elem);
    }

    /** Clear data list */
    public void clear() {
        data.clear();
        notifyDataSetChanged();
    }
```
方法基本分为两类，一类是BaseAdapter中需要实现的方法；另一类用于操作我们的data。
重点看以下几个点：

1.	重写了`getViewTypeCount`和`getItemViewType`，这里type为2，主要是为了在AbsListView最后显示一个进度条。通过`getCount`，`getItemViewType`，以及getView就可以明确的看出。这里也暴露了一个弊端，无法支持多种Item样式的布局。
2.	实现了getView方法，而对外公布了`convert(helper, item)`。convert的参数是`BaseAdapterHelper`和`Bean`，通过`BaseAdapterHelper`封装的View赋值方法，将`Bean`中的数据赋值给ItemView，所以公布这个方法还是极其方便的。
3.	`convert(helper, item)`这个helper为`BaseAdapterHelper`类型，通过`getAdapterHelper`提供，子类可以通过该方法提供扩展的BaseAdapterHelper。关于`getAdapterHelper`的实现见`QuickAdapter`。

####3.2.2 QucikAdapter.java 
这个类中没什么代码，主要用于提供一个快速使用的Adapter。一般情况下直接用此类作为Adapter即可，但是如果你扩展了`BaseAdapterHelper`，可能就需要自己去继承`BaseAdapterHelper`实现自己的Adapter。所以该类，对于`getAdapterHelper`直接返回了`BaseAdapterHelper`。
```java
 protected BaseAdapterHelper getAdapterHelper(int position, View convertView, ViewGroup parent) {
        return BaseAdapterHelper.get(context, convertView, parent, layoutResId, position);
    }
```
####3.2.3 EnhancedQuickAdapter.java 
这个类仅仅是为convert方法添加了一个参数`itemChanged`用于区分dataset changed / dataset invalidated。
```java
 @Override
    protected final void convert(BaseAdapterHelper helper, T item) {
        boolean itemChanged = helper.associatedObject == null || !helper.associatedObject.equals(item);
        helper.associatedObject = item;
        convert(helper, item, itemChanged);
    }

    protected abstract void convert(BaseAdapterHelper helper, T item, boolean itemChanged);
```

可以看到它的实现是通过helper.associatedObject的`equals()`方法，associatedObject的即我们的bean。在`BaseQuickAdapter`可以看到其赋值的代码。

####3.2.4 BaseAdapterHelper.java 
该类的功能：

1. 充当了我们的ViewHolder角色，保存convertView中子View的引用，和convertView通过tag关联；
2. 提供了一堆辅助方法，用于为View赋值和设置事件。

#####(1).构造相关方法 
```java
protected BaseAdapterHelper(Context context, ViewGroup parent, int layoutId, int position) {
        this.context = context;
        this.position = position;
        this.views = new SparseArray<View>();
        convertView = LayoutInflater.from(context) //
                .inflate(layoutId, parent, false);
        convertView.setTag(this);
    }

    /**
     * This method is the only entry point to get a BaseAdapterHelper.
     * @param context     The current context.
     * @param convertView The convertView arg passed to the getView() method.
     * @param parent      The parent arg passed to the getView() method.
     * @return A BaseAdapterHelper instance.
     */
    public static BaseAdapterHelper get(Context context, View convertView, ViewGroup parent, int layoutId) {
        return get(context, convertView, parent, layoutId, -1);
    }

    /** This method is package private and should only be used by QuickAdapter. */
    static BaseAdapterHelper get(Context context, View convertView, ViewGroup parent, int layoutId, int position) {
        if (convertView == null) {
            return new BaseAdapterHelper(context, parent, layoutId, position);
        }

        // Retrieve the existing helper and update its position
        BaseAdapterHelper existingHelper = (BaseAdapterHelper) convertView.getTag();
        existingHelper.position = position;
        return existingHelper;
    }
```
在`QuickAdapter`中，通过上面的5个参数的`get`得到`BaseAdapterHelper`的实例（4个参数的 get方法，只是将position默认传入了-1，即不关注postion方法）。这里可以回想下，我们平时在`getView`中编写的ViewHolder模式的代码。

1.	首先如果`convertView==null`，我们需要去通过`LayoutInflater`去inflate一个布局文件，返回我们的`convertView`。看上面的构造方法，的确是inflate了一个布局作为我们的`convertView`，并且完成对context,postion的赋值，由于我们这里并不会为每个Item的布局去编写ViewHolder，该类充当了一个万能的ViewHolder的角色，所以存储`convertView`子View的引用，使用了`SparseArray<View>`，最后将`convertView`与`BaseAdapterHelper`通过`tag`关联。
2.	如果`convertView!=null`，直接通过`tag`获取到我们关联的`BaseAdapterHelper`，更新position后返回。

#####(2).几个重要的方法 
一般情况下，我们在`Adapter`的`convert`方法中拿到`BaseAdapterHelper`是通过`getView(int viewId)`拿到该`View`，然后进行赋值，使用如下代码：
```java
 public <T extends View> T getView(int viewId) {
        return retrieveView(viewId);
    }

    @SuppressWarnings("unchecked")
    protected <T extends View> T retrieveView(int viewId) {
        View view = views.get(viewId);
        if (view == null) {
            view = convertView.findViewById(viewId);
            views.put(viewId, view);
        }
        return (T) view;
    }
```
通过viewId去我们的views中进行寻找，找到则返回，找不到则添加并返回。每个`convertView`对应于一个`BaseAdapterHelper`，每个`BaseAdapterHelper`中包含一个`views`，`views`中保持`convertView`的子View的引用。

#####(3).辅助方法
一般情况下，通过`getView(int viewId)`拿到该`View`，然后进行赋值就可以了。但是此框架考虑：既然是拿到View然后赋值，不如我提供一些赋值的辅助方法。于是产生了一堆类似`setText(int viewId, String value)`的代码，内部首先通过viewId找到该View，转为`TextView`然后调用`setText(value)`。具体代码如下：

```java
public BaseAdapterHelper setText(int viewId, String value) {
        TextView view = retrieveView(viewId);
        view.setText(value);
        return this;
    }

    public BaseAdapterHelper setImageResource(int viewId, int imageResId) {
        ImageView view = retrieveView(viewId);
        view.setImageResource(imageResId);
        return this;
    }

    public BaseAdapterHelper setBackgroundColor(int viewId, int color) {
        View view = retrieveView(viewId);
        view.setBackgroundColor(color);
        return this;
    }

    public BaseAdapterHelper setBackgroundRes(int viewId, int backgroundRes) {
        View view = retrieveView(viewId);
        view.setBackgroundResource(backgroundRes);
        return this;
    }

    public BaseAdapterHelper setTextColor(int viewId, int textColor) {
        TextView view = retrieveView(viewId);
        view.setTextColor(textColor);
        return this;
    }

    public BaseAdapterHelper setTextColorRes(int viewId, int textColorRes) {
        TextView view = retrieveView(viewId);
        view.setTextColor(context.getResources().getColor(textColorRes));
        return this;
    }

    public BaseAdapterHelper setImageDrawable(int viewId, Drawable drawable) {
        ImageView view = retrieveView(viewId);
        view.setImageDrawable(drawable);
        return this;
    }

    public BaseAdapterHelper setImageUrl(int viewId, String imageUrl) {
        ImageView view = retrieveView(viewId);
        Picasso.with(context).load(imageUrl).into(view);
        return this;
    }

    public BaseAdapterHelper setImageBuilder(int viewId, RequestCreator requestBuilder) {
        ImageView view = retrieveView(viewId);
        requestBuilder.into(view);
        return this;
    }

    public BaseAdapterHelper setImageBitmap(int viewId, Bitmap bitmap) {
        ImageView view = retrieveView(viewId);
        view.setImageBitmap(bitmap);
        return this;
    }

    @SuppressLint("NewApi")
	public BaseAdapterHelper setAlpha(int viewId, float value) {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.HONEYCOMB) {
            retrieveView(viewId).setAlpha(value);
        } else {
            // Pre-honeycomb hack to set Alpha value
            AlphaAnimation alpha = new AlphaAnimation(value, value);
            alpha.setDuration(0);
            alpha.setFillAfter(true);
            retrieveView(viewId).startAnimation(alpha);
        }
        return this;
    }

    public BaseAdapterHelper setVisible(int viewId, boolean visible) {
        View view = retrieveView(viewId);
        view.setVisibility(visible ? View.VISIBLE : View.GONE);
        return this;
    }

    public BaseAdapterHelper linkify(int viewId) {
        TextView view = retrieveView(viewId);
        Linkify.addLinks(view, Linkify.ALL);
        return this;
    }

    public BaseAdapterHelper setTypeface(Typeface typeface, int... viewIds) {
        for (int viewId : viewIds) {
            TextView view = retrieveView(viewId);
            view.setTypeface(typeface);
            view.setPaintFlags(view.getPaintFlags() | Paint.SUBPIXEL_TEXT_FLAG);
        }
        return this;
    }

    public BaseAdapterHelper setProgress(int viewId, int progress) {
        ProgressBar view = retrieveView(viewId);
        view.setProgress(progress);
        return this;
    }

    public BaseAdapterHelper setProgress(int viewId, int progress, int max) {
        ProgressBar view = retrieveView(viewId);
        view.setMax(max);
        view.setProgress(progress);
        return this;
    }

    public BaseAdapterHelper setMax(int viewId, int max) {
        ProgressBar view = retrieveView(viewId);
        view.setMax(max);
        return this;
    }

    public BaseAdapterHelper setRating(int viewId, float rating) {
        RatingBar view = retrieveView(viewId);
        view.setRating(rating);
        return this;
    }
   
    public BaseAdapterHelper setRating(int viewId, float rating, int max) {
        RatingBar view = retrieveView(viewId);
        view.setMax(max);
        view.setRating(rating);
        return this;
    }
 
    public BaseAdapterHelper setTag(int viewId, Object tag) {
        View view = retrieveView(viewId);
        view.setTag(tag);
        return this;
    }
 
    public BaseAdapterHelper setTag(int viewId, int key, Object tag) {
        View view = retrieveView(viewId);
        view.setTag(key, tag);
        return this;
    }

    public BaseAdapterHelper setChecked(int viewId, boolean checked) {
        Checkable view = (Checkable) retrieveView(viewId);
        view.setChecked(checked);
        return this;
    }

    
    public BaseAdapterHelper setAdapter(int viewId, Adapter adapter) {
        AdapterView view = retrieveView(viewId);
        view.setAdapter(adapter);
        return this;
    }
```
都是根据viewId找到View，然后为View赋值的代码。这里只要注意下：`setImageUrl(int viewId, String imageUrl)` 这个方法，默认是通过`Picasso`去加载图片的，当然你可以更改成你项目中使用的图片加载框架Volley，UIL等。
上述代码基本都是为View赋值的代码，有时候我们需要在`getView`中为子View去设置一个事件监听，于是有了下面几个方法：
```java
    public BaseAdapterHelper setOnClickListener(int viewId, View.OnClickListener listener) {
        View view = retrieveView(viewId);
        view.setOnClickListener(listener);
        return this;
    }

    public BaseAdapterHelper setOnTouchListener(int viewId, View.OnTouchListener listener) {
        View view = retrieveView(viewId);
        view.setOnTouchListener(listener);
        return this;
    }

    public BaseAdapterHelper setOnLongClickListener(int viewId, View.OnLongClickListener listener) {
        View view = retrieveView(viewId);
        view.setOnLongClickListener(listener);
        return this;
    }

```
当然了，这里仅仅几个常用的方法，如果有些控件的方法这里没有封装，你就需要通过`BaseAdapterHelper.getView(viewId)`拿到控件，然后去设置事件。


###4. 扩展多种Item布局
通过上面的分析，可以看出base-adapter-helper并不支持多种布局Item的情况，虽然大多数情况下一个种样式即可，但是要是让我用着这么简单的方式写Adapter，忽然来个多种布局Item的ListView又要 按传统的方式去写，这反差就太大了。下面我们介绍，在本框架的基础上添加多布局Item的支持。

#####(1).分析
对于多种布局的Item，大家都清楚，需要去复写`BaseAdapter`的`getViewTypeCount()`和`getItemViewType()`。并且需要在`getView()`里面进行判断以及选取布局文件，不同的布局也需要采用不同的`ViewHolder`。
那么，我们在构造`QucikAdapter`的时候，想办法去设置`getViewTypeCount()`和`getItemViewType()`的值，那么我们可以抽象出一个接口，提供几个方法，如果需要使用多种Item布局的时候，将其传入。

#####(2).扩展

* `MultiItemTypeSupport`
```java
public interface MultiItemTypeSupport<T>
{
	int getLayoutId(int position , T t);
	
	int getViewTypeCount();
	
	int getItemViewType(int postion,T t );
}
```

* 分别在`QuickAdapter`和`BaseQuickAdapter`中添加新的构造方法

`BaseQuickAdapter`

```java
	protected MultiItemTypeSupport<T> mMultiItemSupport;

	public BaseQuickAdapter(Context context, ArrayList<T> data,
			MultiItemTypeSupport<T> multiItemSupport)
	{
		this.mMultiItemSupport = multiItemSupport;
		this.data = data == null ? new ArrayList<T>() : new ArrayList<T>(data);
		this.context = context;
	}

```

`QuickAdapter`

```java
public QuickAdapter(Context context, ArrayList<T> data,
			MultiItemTypeSupport<T> multiItemSupport)
	{
		super(context, data, multiItemSupport);
	}
```
同时肯定需要改写`BaseQuickAdapter`的`getViewTypeCount()`和`getItemViewType()`以及`getView()`。
```java
@Override
	public int getViewTypeCount()
	{
		if (mMultiItemSupport != null)
			return mMultiItemSupport.getViewTypeCount() + 1;
		return 2;
	}

	@Override
	public int getItemViewType(int position)
	{
		if (displayIndeterminateProgress)
		{
			if (mMultiItemSupport != null)
				return position >= data.size() ? 0 : mMultiItemSupport
						.getItemViewType(position, data.get(position));
		} else
		{
			if (mMultiItemSupport != null)
				return mMultiItemSupport.getItemViewType(position,
						data.get(position));
		}

		return position >= data.size() ? 0 : 1;

	}

	@Override
	public View getView(int position, View convertView, ViewGroup parent)
	{
		if (getItemViewType(position) == 0)
		{
			return createIndeterminateProgressView(convertView, parent);
		}
		final H helper = getAdapterHelper(position, convertView, parent);
		T item = getItem(position);
		helper.setAssociatedObject(item);
		convert(helper, item);
		return helper.getView();

	}
```
为了保留其原本提供的添加滚动条的功能，我们在其基础上进行修改。

* 改写`BaseAdapterHelper`的构造方法，因为我们不同的布局，肯定要对于不同的`ViewHolder`，这里`BaseAdapterHelper`其实就扮演了`ViewHolder`的角色。我们的`BaseAdapterHelper`是在`QuickAdapter`的`getAdapterHelper`中构造的，修改后代码：

`QuickAdapter`

```java
protected BaseAdapterHelper getAdapterHelper(int position,
			View convertView, ViewGroup parent)
	{

		if (mMultiItemSupport != null)
		{
			return get(
					context,
					convertView,
					parent,
					mMultiItemSupport.getLayoutId(position, data.get(position)),
					position);
		} else
		{
			return get(context, convertView, parent, layoutResId, position);
		}
	}
``` 

`BaseAdapterHelper`的`get`方法也需要修改。
```java
/** This method is package private and should only be used by QuickAdapter. */
	static BaseAdapterHelper get(Context context, View convertView,
			ViewGroup parent, int layoutId, int position)
	{
		if (convertView == null)
		{
			return new BaseAdapterHelper(context, parent, layoutId, position);
		}

		// Retrieve the existing helper and update its position
		BaseAdapterHelper existingHelper = (BaseAdapterHelper) convertView
				.getTag();

		if (existingHelper.layoutId != layoutId)
		{
			return new BaseAdapterHelper(context, parent, layoutId, position);
		}

		existingHelper.position = position;
		return existingHelper;
	}
```
我们在helper中存储了当前的layoutId，如果layoutId不一致，则重新创建。

#####(3).测试
下面展示核心代码
```java
mListView = (ListView) findViewById(R.id.id_lv_main);
		
		MultiItemTypeSupport<ChatMessage> multiItemTypeSupport = new MultiItemTypeSupport<ChatMessage>()
		{
			@Override
			public int getLayoutId(int position, ChatMessage msg)
			{
				if (msg.isComMeg())
				{
					return R.layout.main_chat_from_msg;
				}
				return R.layout.main_chat_send_msg;
			}

			@Override
			public int getViewTypeCount()
			{
				return 2;
			}

			@Override
			public int getItemViewType(int postion, ChatMessage msg)
			{
				if (msg.isComMeg())
				{
					return ChatMessage.RECIEVE_MSG;
				}
				return ChatMessage.SEND_MSG;
			}
		};

		initDatas();

		
		mAdapter = new QuickAdapter<ChatMessage>(ChatActivity.this, mDatas,
				multiItemTypeSupport)
		{
			@Override
			protected void convert(BaseAdapterHelper helper, ChatMessage item)
			{
				switch (helper.layoutId)
				{
				case R.layout.main_chat_from_msg:
					helper.setText(R.id.chat_from_content, item.getContent());
					helper.setText(R.id.chat_from_name, item.getName());
					helper.setImageResource(R.id.chat_from_icon, item.getIcon());
					break;
				case R.layout.main_chat_send_msg:
					helper.setText(R.id.chat_send_content, item.getContent());
					helper.setText(R.id.chat_send_name, item.getName());
					helper.setImageResource(R.id.chat_send_icon, item.getIcon());
					break;
				}

			}
		};
//		mAdapter.showIndeterminateProgress(true);
		
		mListView.setAdapter(mAdapter);
```
当遇到多种布局Item的时候，首先构造一个`MultiItemTypeSupport`接口对象，然后记得在`convert`中根据layoutId，分别进行赋值，因为不同的布局，控件可能不同，id也可能不同。

贴张效果图

<img src="image/snapshot.png" width = "320px"  />

添加多Item布局后的地址：[github](https://github.com/hongyangAndroid/base-adapter-helper)
