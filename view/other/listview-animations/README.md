ListViewAnimations 源码解析
----------------
> 本文为 [Android 开源项目源码解析](http://a.codekk.com) 中 ListViewAnimations 部分   
项目地址：[ListViewAnimations](https://github.com/nhaarman/ListViewAnimations)，分析的版本：[25124c5](https://github.com/nhaarman/ListViewAnimations/commit/25124c555c9d6ced81ce3e78b3d8a0e02fc274ce)，Demo 地址：[ListViewAnimations Demo](https://github.com/aosp-exchange-group/android-open-project-demo/tree/master/listview-animations-demo)  
分析者：[demonyan](https://github.com/demonyan)，分析状态：已完成，校对者：[Trinea](https://github.com/trinea)，校对状态：未开始

## 1. 功能介绍
### 1.1 ListViewAnimations
`ListViewAnimations`开源库可以让 Android 开发者很方便为`ListView`(严格来说`AbsListView`更准确些，因为还包含`GridView`。但为了方便统称`ListView`，下同)添加动画效果。
`ListViewAnimations`提供`Alpha`，`Swing-RightIn`，`Swing-LeftIn`， `Swing-BottomIn`，`Swing-RightIn`和`ScaleIn`等内置动画效果。并支持`StickyListHeaders`
，`Swipe-to-Dismiss`和`Swipe-Undo`，`Drag-and-Drop`重排序，`ExpandableList`等常用功能。开发者可以组合库提供的动画，也可以继承库的抽象类实现自定义的动画。

## 2. 总体设计
我们知道，`ListView`视图的展现方式是无穷尽的，数据来源也是多种多样的，同时人们还有着各式各样的动画需求。这就要求`ListView`具有很好的扩展性。  
![listview-adapter.png](https://github.com/aosp-exchange-group/android-open-project-analysis/blob/master/view/other/listview-animations/image/listview-adapter.png)
以上是`Android Framework`中`ListView`及相应`Adapter`的类关系图。得益于`Android`团队的良好设计，通过桥接模式很好的解决了对`ListView`复杂多变的需求，也就是通过将抽象部分和实现部分分离解耦，从而使得各自可以适应各种需求变化。同时，还可以发现适配器模式的应用，通过`ListAdapter`定义最基本的目标接口，其他适配器类针对目标接口对数据源进行修饰。例如，`SimpleAdapter`提供最简单的适配，`ArrayAdapter`对数据源是数组的情况进行适配，`CursorAdapter`对数据源是数据库的情况进行适配。`ListViewAnimations`正是对此设计的继承和发扬，作者比较巧妙地运用装饰模式，不改变原有类的情况下，对其进行增强，也就是在`getView`方法体内对`item`增加各种动画效果。装饰模式既能动态添加新功能，又有效避免功能组合时的类爆炸。`BaseAdapterDecorator`是继承于`BaseAdapter`的抽象装饰类，类中的`mDecoratorBaseAdapter`对象用于保存装饰角色。该类是`ListViewAnimations`的核心，其他具体装饰器都继承于该类并实现`item`的动画功能。

## 3. 流程图
`ListViewAnimations`动画库的使用流程图如下所示。   
![listview-animations-use-flow.png](https://github.com/aosp-exchange-group/android-open-project-analysis/blob/master/view/other/listview-animations/image/listview-animations-use-flow.png)  
与使用`ListView`的流程基本一致，主要不同的地方在于可以使用一个或多个`BaseAdapterDecorator`类来装饰`BaseAdapter`对象。  

## 4. 详细设计
`ListViewAnimations`组成模块包含三部分：  
4.1 `lib-core`：核心库，主要是各种展示动画部分
>* `BaseAdapterDecorator`  
`BaseAdapterDecorator`继承于`BaseAdapter`类，并实现了`SectionIndexer`, `Swappable`, `Insertable`, `ListViewWrapperSetter`接口。`ListViewAnimations`库中其他`Adapter`都继承于该类，并根据具体需求实现相应接口中的方法。类关系图如下所示。
![listviewanimations-BaseAdapterDecorator.png](https://github.com/aosp-exchange-group/android-open-project-analysis/blob/master/view/other/listview-animations/image/listviewanimations-BaseAdapterDecorator.png)
`SectionIndexer`接口声明在`ListView`中的`sections`间快速滚动的方法，`Swappable`接口声明实现`ListView`中两个`item`相互交换的方法，`Insertable`接口声明给`ListView`添加`item`的方法，`ListViewWrapperSetter`接口声明对`AbsListView`进行包装的方法。
该类包含成员变量`mDecoratedBaseAdapter`和成员变量`mListViewWrapper`，分别用于保存装饰角色和对应的任意`ListView`包装类实例。其中，`setListViewWrapper`方法用于保存`ListView`的包装类实例。
```java
public void setListViewWrapper(@NonNull final ListViewWrapper listViewWrapper) {
    mListViewWrapper = listViewWrapper;
    /** 递归初始化每个装饰角色的 mListViewWrapper */
    if (mDecoratedBaseAdapter instanceof ListViewWrapperSetter) {
        ((ListViewWrapperSetter) mDecoratedBaseAdapter).setListViewWrapper(listViewWrapper);
    }
}
```
>* `ArrayAdapter`  
继承自 BaseAdapter 类，用于`ArrayList`适配，并实现`Swappable`和`Insertable`接口，也就是说使用该类作为`Adapter`，可以实现基本的`item`互换和增加删除`item`。
```java
/** 实现 Swappable 接口　*/
public void swapItems(final int positionOne, final int positionTwo) {
    T firstItem = mItems.set(positionOne, getItem(positionTwo));
    notifyDataSetChanged();
    mItems.set(positionTwo, firstItem);
}
/** 实现 Insertable 接口　*/
public void add(final int index, @NonNull final T item) {
    mItems.add(index, item);
    notifyDataSetChanged();
}
```
>* `ViewAnimator`  
根据位置判断是否应该给`item`添加动画，并计算动画延时。包含成员变量`mAnimators`，类型为`SparseArray<Animator>`，用于保存`item`的动画。
>* `AnimationAdatper`  
`BaseAdaperDecorator`的子类，根据用户需求在`getView`方法中为`item`添加`AnimatorSet`。包含成员变量`mViewAnimator`，通过调用它的`animateViewIfNecessary`方法给`item`添加动画。动画由三部分构成，第一部分，通过`mDecoratorBaseAdapter`的`getAnimators`方法获得装饰器实例的动画；第二部分，获得当前实例的动画；第三部分，`Alpha`显示动画。最终`item`执行这三部分的动画组合。
```java
private void animateViewIfNecessary(final int position, @NonNull final View view, 
        @NonNull final ViewGroup parent) {
    ...
    Animator[] childAnimators;
    if (getDecoratedBaseAdapter() instanceof AnimationAdapter) {
        childAnimators = ((AnimationAdapter) getDecoratedBaseAdapter()).getAnimators(parent, view);
    } else {
        childAnimators = new Animator[0];
    }
    Animator[] animators = getAnimators(parent, view);
    Animator alphaAnimator = ObjectAnimator.ofFloat(view, ALPHA, 0, 1);
    /** 将动画组合起来 */
    Animator[] concatAnimators = AnimatorUtil.concatAnimators(childAnimators, animators, alphaAnimator);
    /** 调用 ViewAnimator 的方法给 item 添加动画 */
    mViewAnimator.animateViewIfNecessary(position, view, concatAnimators);
}
```
>* `ResourceAnimationAdapter`  
`AnimationAdatper`的子类，`getAnimators`方法中可以通过`getAnimationResourceId`添加自定义动画。开发者可以继承这个抽象类并实现`getAnimationResourceId`方法，从而实现自定义动画效果。
```java
public Animator[] getAnimators(@NonNull final ViewGroup parent, @NonNull final View view) {
    return new Animator[]{AnimatorInflater.loadAnimator(mContext, getAnimationResourceId())};
}
```
>* `SingleAnimationAdapter` `AnimationAdatper`的子类，提供的抽象方法`getAnimator`由子类实现。通过继承这个抽象类并实现`getAnimator`方法，从而实现自定义动画效果。  
>* `AlphaInAnimationAdapter`
`AnimationAdatper`的子类，`getAnimator`方法并未返回任何动画。
```java
protected Animator getAnimator(@NonNull final ViewGroup parent, @NonNull final View view) {
    return new Animator[0];
}
```
>* `ScaleInAnimationAdapter`  
`AnimationAdatper`的子类，`getAnimator`方法返回 scale 动画。
```java
protected Animator getAnimator(@NonNull final ViewGroup parent, @NonNull final View view) {
    ObjectAnimator scaleX = ObjectAnimator.ofFloat(view, SCALE_X, mScaleFrom, 1f);
    ObjectAnimator scaleY = ObjectAnimator.ofFloat(view, SCALE_Y, mScaleFrom, 1f);
    return new ObjectAnimator[]{scaleX, scaleY};
}
```
>* `SwingBottomInAnimationAdapter`  
`AnimationAdatper`的子类，`getAnimator`方法返回 Y 轴方向从下往上的动画。
```java
protected Animator getAnimator(@NonNull final ViewGroup parent, @NonNull final View view) {
    return ObjectAnimator.ofFloat(view, TRANSLATION_Y, parent.getMeasuredHeight() >> 1, 0);
}
```
>* `SwingLeftInAnimationAdapter`  
`AnimationAdatper`的子类，`getAnimator`方法返回从左往右的动画。
```java
protected Animator getAnimator(@NonNull final ViewGroup parent, @NonNull final View view) {
    return ObjectAnimator.ofFloat(view, TRANSLATION_X, 0 - parent.getWidth(), 0);
}
```
>* `SwingRightInAnimationAdapter`  
`AnimationAdatper`的子类，`getAnimator`方法返回从右往左的动画。
```java
protected Animator getAnimator(@NonNull final ViewGroup parent, @NonNull final View view) {
    return ObjectAnimator.ofFloat(view, TRANSLATION_X, parent.getWidth(), 0);
}
```
  
4.2 `lib-manipulation`：主要是操作`item`的功能，比如`Swipe-to-Dismiss`，`Drag-and-Drop`，`Swipe-Undo`等。  
>* `DynamicListView`  
`ListViewAnimations`库中最重要的自定义组件，继承于`ListView`。包含以下功能：`Drag-and-Drop`功能，由`DragAndDropHandler`类实现；`Swipe-to-Dismiss`功能，由`SwipeTouchListener`类及其子类实现；`Swipe-Undo`功能，由`SwipeUndoAdapter`类实现；`Animate addition`功能，由`AnimateAdditionAdapter`类实现。
`DynamicListView`重写了父类`ListView`的`setAdapter`方法和`dispatchTouchEvent`方法，从而扩展了父类的功能。流程图如下所示。
![listviewanimations-DynamicListview-event.png](https://github.com/aosp-exchange-group/android-open-project-analysis/blob/master/view/other/listview-animations/image/listviewanimations-dynamicListview-event.png)
`setAdapter`方法用于给`ListView`绑定`Adapter`，首先通过`Adapter`链向上获得`rootAdapter`，即最终实现`getView`接口的`Adapter`，一般这个`Adapter`需要开发者实现。如果在`Adapter`链中发现`SwipeUndoAdatper`的实例，说明需要`Swipe-Undo`功能，则初始化`mSwipeUndoAdapter`为该`SwipeUndoAdatper`的实例。如果`rootAdapter`实现了`Insertable`接口，则初始化`mAnimationAdditionAdapter`。如果`mDragAndDropHandler`不为空，则为`mDragAndDropHandler`绑定`Adapter`。
```java
public void setAdapter(final ListAdapter adapter) {
    ListAdapter wrappedAdapter = adapter;
    mSwipeUndoAdapter = null;
    if (adapter instanceof BaseAdapter) {
        BaseAdapter rootAdapter = (BaseAdapter) wrappedAdapter;
        /** 遍历 Adapter 链获得 rootAdapter */
        while (rootAdapter instanceof BaseAdapterDecorator) {
            if (rootAdapter instanceof SwipeUndoAdapter) {
                mSwipeUndoAdapter = (SwipeUndoAdapter) rootAdapter;
            }
            rootAdapter = ((BaseAdapterDecorator) rootAdapter).getDecoratedBaseAdapter();
        }
        if (rootAdapter instanceof Insertable) {
            mAnimateAdditionAdapter = new AnimateAdditionAdapter((BaseAdapter) wrappedAdapter);
            mAnimateAdditionAdapter.setListView(this);
            wrappedAdapter = mAnimateAdditionAdapter;
        }
    }
    super.setAdapter(wrappedAdapter);
    if (mDragAndDropHandler != null) {
        mDragAndDropHandler.setAdapter(adapter);
    }
}
```
> `dispatchTouchEvent`方法用于处理分发来的事件。如果还没有`TouchEventHandler`消费`onTouch`事件，则先交由`DragAndDropHandler`的`onTouchEvent`来尝试处理该事件，如果`onTouch`事件可以满足拖拽的条件，那么`TouchEventHandler`则为`DragAndDropHandler`，同时会给`SwipeTouchListener`发送`ACTION_CANCEL`事件，以后的`ACTION_MOVE`事件都会交由`DragAndDropHandler`来处理，直到`ACTION_UP`事件或者`ACTION_CANCEL`事件出现。否则会交由`SwipeTouchListener`来尝试处理事件，如果`onTouch`事件满足横滑的条件，那么`TouchEventHandler`则为`SwipeTouchListener`，同时发送`ACTION_CANCEL`事件给`DragAndDropHandler`，以后的`ACTION_MOVE`事件都会交由`SwipeTouchListener`来处理，直到`ACTION_UP`事件或者`ACTION_CANCEL`事件出现。
```java
public boolean dispatchTouchEvent(@NonNull final MotionEvent ev) {
    if (mCurrentHandlingTouchEventHandler == null) {
        /* None of the TouchEventHandlers are actively consuming events yet. */
        boolean firstTimeInteracting = false;
        /* We don't support dragging items when there are items in the undo state. */
        if (!(mSwipeTouchListener instanceof SwipeUndoTouchListener)
                || !((SwipeUndoTouchListener) mSwipeTouchListener).hasPendingItems()) {
            /* Offer the event to the DragAndDropHandler */
            if (mDragAndDropHandler != null) {
                mDragAndDropHandler.onTouchEvent(ev);
                firstTimeInteracting = mDragAndDropHandler.isInteracting();
                if (firstTimeInteracting) {
                    mCurrentHandlingTouchEventHandler = mDragAndDropHandler;
                    sendCancelEvent(mSwipeTouchListener, ev);
                }
            }
        }
        /* If not handled, offer the event to the SwipeDismissTouchListener */
        if (mCurrentHandlingTouchEventHandler == null && mSwipeTouchListener != null) {
            mSwipeTouchListener.onTouchEvent(ev);
            firstTimeInteracting = mSwipeTouchListener.isInteracting();
            if (firstTimeInteracting) {
                mCurrentHandlingTouchEventHandler = mSwipeTouchListener;
                sendCancelEvent(mDragAndDropHandler, ev);
            }
        }
        if (firstTimeInteracting) {
            /* One of the TouchEventHandlers is now taking over control.
                Cancel touch event handling on this DynamicListView */
            MotionEvent cancelEvent = MotionEvent.obtain(ev);
            cancelEvent.setAction(MotionEvent.ACTION_CANCEL);
            super.onTouchEvent(cancelEvent);
        }
        return firstTimeInteracting || super.dispatchTouchEvent(ev);
    } else {
        return onTouchEvent(ev);
    }
}
```
>* `TouchEventHandler`  
声明处理`onTouch`事件的`onTouchEvent`接口方法和`item`正在被处理的`isInteracting`接口方法。`DragAndDropHandler`和`SwipeTouchListener`都实现了该接口。  
>* `DragAndDropHandler`  
实现`TouchEventHandler`接口，实现`ListView`的`Drag-and-Drop`功能。该类的`onTouchEvent`方法接收`dispatchTouchEvent`分发来的`onTouch`事件，当接收到`ACTION_DOWN`时，记录手指按压的位置。
当接收到`ACTION_MOVE`时，如果当前还未开始拖拽`item`，并且手指竖移距离`deltaY`大于临界值`mSlop`，则通过`DraggableManage`类来判断是否满足拖拽`item`的条件；如果已经开始拖拽`item`，则通过手指拖拽距离判断是否可以与相邻`item`互换位置，并根据`SDK`版本执行相应动画。`Kitkat`及以前的版本执行`KitKatSwitchViewAnimator`动画，`Kitkat`之后的版本执行`LSwitchViewAnimator`动画。
```java
private boolean handleMoveEvent(@NonNull final MotionEvent event) {
    ...
    if (mHoverDrawable == null && Math.abs(deltaY) > mSlop && Math.abs(deltaY) > Math.abs(deltaX)) {
        int position = mWrapper.pointToPosition((int) event.getX(), (int) event.getY());
        if (position != AdapterView.INVALID_POSITION) {
            View downView = mWrapper.getChildAt(position - mWrapper.getFirstVisiblePosition());
            assert downView != null;
            if (mDraggableManager.isDraggable(downView, position - mWrapper.getHeaderViewsCount(), 
                    event.getX() - downView.getX(), event.getY() - downView.getY())) {
                startDragging(position - mWrapper.getHeaderViewsCount());
                handled = true;
            }
        }
    } else if (mHoverDrawable != null) {
        mHoverDrawable.handleMoveEvent(event);
        switchIfNecessary();
        mWrapper.getListView().invalidate();
        handled = true;
    }
    return handled;
}
```
> 当接收到`ACTION_UP`时，根据手指松开时的当前位置来确定`item`的最终位置.

>* `DraggableManager`  
判断用户是否可以拖拽`item`的接口。通过实现该接口的 isDraggable 方法，可以自定义满足拖拽的条件。
>* `TouchViewDraggableManager`  
实现了`DraggableManager`接口，通过是否与特定的`View`接触来判断用户可以拖拽`item`。
>* `SwipeTouchListener`  
实现`OnTouchListener`接口和`TouchEventHandler`接口，使得`ListView`的`item`能够`swipeable`。子类需要实现`afterViewFling`抽象方法，以明确手指横滑后的操作。该类的`onTouchEvent`方法接收`dispatchTouchEvent`分发来的`onTouch`事件，当接收到`ACTION_DOWN`时，通过`DismissableManager`类来判断`item`是否可以删除。如果`DynamicListView`的父窗口也可以水平方向滚动，则需要通过`requestDisallowInterceptTouchEvent`来请求不要拦截`onTouch`事件。
```java
private boolean handleDownEvent(@Nullable final View view, @NonNull final MotionEvent motionEvent) {
    ...
    int downPosition = AdapterViewUtil.getPositionForView(mListViewWrapper, downView);
    mCanDismissCurrent = isDismissable(downPosition);
    /* Check if we are processing the item at this position */
    if (mCurrentPosition == downPosition || downPosition >= mVirtualListCount) {
        return false;
    }
    if (view != null) {
        view.onTouchEvent(motionEvent);
    }
    disableHorizontalScrollContainerIfNecessary(motionEvent, downView);
    ...
}
```
> 当接收到`ACTION_MOVE`时，判断手指横滑距离`deltaX`是否大于临界值`mSlop`，如果是则给`ListView`发送一个`ACTION_CANCEL`事件，使得`ListView`不再处理`ACTION_MOVE`事件，从而不会触发`item`的背景高亮，然后通过手指横滑的距离开始`item`横向滑动的动画效果。
```java
private boolean handleMoveEvent(@Nullable final View view, @NonNull final MotionEvent motionEvent) {
    ... 
    if (Math.abs(deltaX) > mSlop && Math.abs(deltaX) > Math.abs(deltaY)) {
        if (!mSwiping) {
            mActiveSwipeCount++;
            onStartSwipe(mCurrentView, mCurrentPosition);
        }
        mSwiping = true;
        mListViewWrapper.getListView().requestDisallowInterceptTouchEvent(true);
        /* Cancel ListView's touch (un-highlighting the item) */
        if (view != null) {
            MotionEvent cancelEvent = MotionEvent.obtain(motionEvent);
            cancelEvent.setAction(MotionEvent.ACTION_CANCEL | motionEvent.getActionIndex() << MotionEvent.ACTION_POINTER_INDEX_SHIFT);
            view.onTouchEvent(cancelEvent);
            cancelEvent.recycle();
        }
    }
    if (mSwiping) {
        if (mCanDismissCurrent) {
            ViewHelper.setTranslationX(mSwipingView, deltaX);
            ViewHelper.setAlpha(mSwipingView, Math.max(mMinimumAlpha,
                        Math.min(1, 1 - 2 * Math.abs(deltaX) / mViewWidth)));
        } else {
            ViewHelper.setTranslationX(mSwipingView, deltaX * 0.1f);
        }
        return true;
    }
    return false;
}
```
> 当接收到`ACTION_CANCEL`事件时，`item`会回到初始状态。
> 当接收到`ACTION_UP`事件时，通过手指横滑的距离和速度来判断是否需要删除当前`item`，如果需要则显示`UndoView`，由用户来最终决定是否删除当前`item`。

>* `SwipeDismissTouchListener`  
`SwipeTouchListener`的子类，实现手指横滑后`item`的删除操作，删除成功后调用`OnDismissCallback`来回调开发者实现的逻辑。
>* `SwipeUndoTouchListener`  
`SwipeDismissTouchListener`的子类，在手指横滑后可以让用户选择是否撤销删除，而不是直接删除`item`。可以显示或者隐藏`UndoView`，当用户确认撤销操作时显示撤销动画。
>* `AnimateAdditionAdaptor`  
`BaseAdaperDecorator`的子类，实现插入`item`时的动画效果。该类装饰的`rootAdapter`必须实现`Insertable`接口，否则构造方法会抛出异常。
>* `SwipeDismissAdapter`  
`BaseAdapterDecorator`的子类，包含成员变量`mDismissTouchListener`和成员变量`mOnDismissCallback`。通过该装饰类，普通的`ListView`(不需使用`DynamicListView`)也可以实现`Swipe-to-Dismiss`功能。重写父类`setListViewWrapper`方法初始化成员变量，`mDismissTouchListener`用来给`ListView`设置`onTouch`监听器，`mOnDismissCallback`是删除`item`时的回调接口。
```java
public void setListViewWrapper(@NonNull final ListViewWrapper listViewWrapper) {
    super.setListViewWrapper(listViewWrapper);
    if (getDecoratedBaseAdapter() instanceof ArrayAdapter<?>) {
        ((ArrayAdapter<?>) getDecoratedBaseAdapter()).propagateNotifyDataSetChanged(this);
    }
    mDismissTouchListener = new SwipeDismissTouchListener(listViewWrapper, mOnDismissCallback);
    if (mParentIsHorizontalScrollContainer) {
        mDismissTouchListener.setParentIsHorizontalScrollContainer();
    }
    if (mSwipeTouchChildResId != 0) {
        mDismissTouchListener.setTouchChild(mSwipeTouchChildResId);
    }
    listViewWrapper.getListView().setOnTouchListener(mDismissTouchListener);
}
```
>* `UndoAdapter`  
声明`getUndoView`接口方法和`getUndoClickView`接口方法。如果需要`Undo`功能，必须实现该接口方法。
>* `SwipeUndoAdapter`  
`BaseAdaperDecorator`的子类，为`ListView`添加`Swipe-Undo`行为。调用成员变量`mSwipeUndoTouchListener`的`undo`方法执行撤销动画并恢复`item`的初始状态，或者调用成员变量`mSwipeUndoTouchListener`的`dismiss`方法删除给定位置的`item`。
>* `SimpleSwipeUndoAdapter`  
继承`SwipeUndoAdapter`类并实现`UndoCallback`接口，`getView`方法中会将`primaryView`或者`undoView`显示于`item`。所装饰的`Adapter`必须实现`UndoAdapter`接口，否则会产生异常。
```java
public View getView(final int position, @Nullable final View convertView, 
        @NonNull final ViewGroup parent) {
    SwipeUndoView view = (SwipeUndoView) convertView;
    if (view == null) {
        view = new SwipeUndoView(mContext);
    }
    View primaryView = super.getView(position, view.getPrimaryView(), view);
    view.setPrimaryView(primaryView);
    View undoView = mUndoAdapter.getUndoView(position, view.getUndoView(), view);
    view.setUndoView(undoView);
    mUndoAdapter.getUndoClickView(undoView).setOnClickListener(new UndoClickListener(view, position));
    /** 显示 primaryView 或者 undoView */
    boolean isInUndoState = mUndoPositions.contains(position);
    primaryView.setVisibility(isInUndoState ? View.GONE : View.VISIBLE);
    undoView.setVisibility(isInUndoState ? View.VISIBLE : View.GONE);
    return view;
}
```
>* `ExpandableListItemAdapter`  
`ArrayAdapter`的子类，实现点击`item`的`Expandable`行为。开发者需要继承该抽象类类并实现`getTitleView`方法和`getContentView`方法，`getTitleView`方法返回显示`Title`的视图，`getContentView`方法返回显示`Content`的视图。

4.3 `lib-core-slh`：对动画核心库的扩展，用于支持`StickListHeaders`功能。  
`StickyListHeaders`使得在`Android`也可以像`iOS`一样，给`ListView`中内容添加`Header`的开源库，可以参考本文后的资料链接。
>* `StickyListHeadersAdapterDecorator`  
继承于`BaseAdapterDecorator`类并实现`StickyListHeadersAdapter`接口，在`getHeaderView`方法中给`headView`添加动画。所装饰的`Adapter`必须实现`StickyListHeadersAdapter`接口，否则会产生异常。
```java
public View getHeaderView(final int position, final View convertView, final ViewGroup parent) {
    ...
    View itemView = mStickyListHeadersAdapter.getHeaderView(position, convertView, parent);
    animateViewIfNecessary(position, itemView, parent);
    return itemView;
}
```
>* `StickyListHeadersListViewWrapper`  
`StickyListHeadersListView`的包装类，实现`ListViewWrapper`的接口方法。

## 5. 杂谈
由于`ListViewAnimations`库实现中出现许多包装类以及回调接口，代码可读性不高。通过多读源码，同时结合`Demo`学习该库，对了解`ListView`实现思路，`View`的事件分发机制，`Android`动画基础和设计模式会有所帮助。目前`RecyclerView`以它独有的优势得到愈来愈多的使用，`ListViewAnimations`库的`feature_recyclervier`分支中实现了对`RecyclerView`的支持。作者可能为了避免`master`太复杂而带来的使用不便，或者觉得`RecyclerView`对`item`动画(`RecycleView`有`ItemAnimator`来实现各种动画)的支持已经足够灵活和优秀，`mater`分支并未看到对`RecyclerView`的支持。

### 参考文献
1. [Android 设计模式源码解析之桥接模式](https://github.com/simple-android-framework/android_design_patterns_analysis/tree/master/bridge/shen0834)
1. [公共技术点之 View 事件传递](http://a.codekk.com/detail/Android/Trinea/%E5%85%AC%E5%85%B1%E6%8A%80%E6%9C%AF%E7%82%B9%E4%B9%8B%20View%20%E4%BA%8B%E4%BB%B6%E4%BC%A0%E9%80%92)
1. [公共技术点之 Android 动画基础](http://a.codekk.com/detail/Android/lightSky/%E5%85%AC%E5%85%B1%E6%8A%80%E6%9C%AF%E7%82%B9%E4%B9%8B%20Android%20%E5%8A%A8%E7%94%BB%E5%9F%BA%E7%A1%80)
1. [StickyListHeaders](https://github.com/emilsjolander/StickyListHeaders)

