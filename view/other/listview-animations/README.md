# ListViewAnimationsԴ�����



---
����Ϊ [Android ��Դ��ĿԴ�����](http://codekk.com/open-source-project-analysis) ��ListViewAnimations����
��Ŀ��ַ��[ListViewAnimations](https://github.com/nhaarman/ListViewAnimations)�������İ汾��[25124c5](https://github.com/nhaarman/ListViewAnimations/commit/25124c555c9d6ced81ce3e78b3d8a0e02fc274ce)��Demo ��ַ��ListViewAnimations Demo
�����ߣ�[demonyan](https://github.com/demonyan)������״̬��δ��ɣ�У���ߣ�[Trinea](https://github.com/trinea)��У��״̬��δ��ʼ

## 1. ���ܽ���
### 1.1 ListViewAnimations
`ListViewAnimations`��Դ�������Android�����ߺܷ���Ϊ`ListView`(�ϸ���˵`AbsListView`��׼ȷЩ����Ϊ�˷���ͳ��`ListView`����ͬ)��Ӷ���Ч����
`ListViewAnimations`�ṩ`Alpha`��`Swing-RightIn`��`Swing-LeftIn`�� `Swing-BottomIn`��`Swing-RightIn`��`ScaleIn`�����ö���Ч������֧��`StickyListHeaders`
��`Swipe-to-Dismiss`��`Swipe-Undo`��`Drag-and-Drop`������`ExpandableList`�ȳ��ù��ܡ������߿�����Ͽ��ṩ�Ķ�����Ҳ���Լ̳п�ĳ�����ʵ���Զ���Ķ�����

## 2. �������
����֪����`ListView`��ͼ��չ�ַ�ʽ������ģ�������ԴҲ�Ƕ��ֶ����ģ�ͬʱ���ǻ����Ÿ�ʽ�����Ķ����������Ҫ��`ListView`���кܺõ���չ�ԡ�
![listview-adapter](https://github.com/demonyan/android-open-project-analysis/tree/master/view/other/listview-animations/image/listview-adapter.png)
������`Android Framework`��`ListView`����Ӧ`Adapter`�����ϵͼ��������Android�Ŷӵ�������ƣ�ͨ���Ž�ģʽ�ܺõĽ���˶�`ListView`���Ӷ�������Ҳ����ͨ�������󲿷ֺ�ʵ�ֲ��ַ������Ӷ�ʹ�ø��Կ�����Ӧ��������仯��ͬʱ�������Է���������ģʽ��Ӧ�ã�ͨ��`ListAdapter`�����������Ŀ��ӿڣ����������������Ŀ��ӿڶ�����Դ�������Ρ����磬`SimpleAdapter`�ṩ��򵥵����䣬`ArrayAdapter`������Դ�����������������䣬`CursorAdapter`������Դ�����ݿ������������䡣`ListViewAnimations`���ǶԴ���Ƶļ̳кͷ�����߱Ƚ����������װ��ģʽ�����ı�ԭ���������£����������ǿ��Ҳ������`getView`�����������Ӹ��ֶ���Ч����װ��ģʽ���ܶ�̬����¹��ܣ�����Ч���⹦�����ʱ���౬ը��`BaseAdapterDecorator`�Ǽ̳���`BaseAdapter`�ĳ���װ���࣬���е�`mDecoratorBaseAdapter`�������ڱ���װ�ν�ɫ��������`ListViewAnimations`�ĺ��ģ���������װ�������̳��ڸ��ಢʵ��`item`�Ķ������ܡ�

## 3. ����ͼ

## 4. ��ϸ���
`ListViewAnimations`���ģ����������֣�
4.1 `lib-core`�����Ŀ⣬��Ҫ�Ǹ���չʾ��������
>* `BaseAdapterDecorator`
`BaseAdapterDecorator`�̳���`BaseAdapter`�࣬��ʵ����`SectionIndexer`��`Swappable`��`Insertable`��`ListViewWrapperSetter`�ӿڡ�`ListViewAnimations`��������`Adapter`���̳��ڸ��࣬�����ݾ�������ʵ����Ӧ�ӿ��еķ��������ϵͼ������ʾ��
![listviewanimations-BaseAdapterDecorator.png](https://github.com/demonyan/android-open-project-analysis/blob/master/view/other/listview-animations/image/listviewanimations-BaseAdapterDecorator.png)
`SectionIndexer`�ӿ�������`ListView`�е�`sections`����ٹ����ķ�����`Swappable`�ӿ�����ʵ��`ListView`������`item`�໥�����ķ�����`Insertable`�ӿ�������`ListView`���`item`�ķ�����`ListViewWrapperSetter`�ӿ�������`AbsListView`���а�װ�ķ�����
���������Ա����`mDecoratedBaseAdapter`�ͳ�Ա����`mListViewWrapper`���ֱ����ڱ���װ�ν�ɫ�Ͷ�Ӧ������`ListView`��װ��ʵ�������У�`setListViewWrapper`�������ڱ���`ListView`�İ�װ��ʵ����
```java
public void setListViewWrapper(@NonNull final ListViewWrapper listViewWrapper) {
    mListViewWrapper = listViewWrapper;
    /** �ݹ��ʼ��ÿ��װ�ν�ɫ��mListViewWrapper */
    if (mDecoratedBaseAdapter instanceof ListViewWrapperSetter) {
        ((ListViewWrapperSetter) mDecoratedBaseAdapter).setListViewWrapper(listViewWrapper);
    }
}
```
>* `ArrayAdapter`
�̳�BaseAdapter�࣬����`ArrayList`���䣬��ʵ��`Swappable`��`Insertable`�ӿڣ�Ҳ����˵ʹ�ø�����Ϊ`Adapter`������ʵ�ֻ����Ļ���`item`������ɾ��`item`��
```java
/** ʵ��Swappable�ӿڡ�*/
public void swapItems(final int positionOne, final int positionTwo) {
    T firstItem = mItems.set(positionOne, getItem(positionTwo));
    notifyDataSetChanged();
    mItems.set(positionTwo, firstItem);
}
/** ʵ��Insertable�ӿڡ�*/
public void add(final int index, @NonNull final T item) {
    mItems.add(index, item);
    notifyDataSetChanged();
}
```
>* `ViewAnimator`
����λ���ж��Ƿ�Ӧ�ø���View��Ӷ����������㶯����ʱ��������Ա����`mAnimators`������Ϊ`SparseArray<Animator>`�����ڱ���`item`�Ķ�����
>* `AnimationAdatper`
`BaseAdaperDecorator`�����࣬�����û�������`getView`������Ϊ`item`���`AnimatorSet`��������Ա����`mViewAnimator`��ͨ����������`animateViewIfNecessary`��������View��Ӷ����������������ֹ��ɣ���һ���֣�ͨ��`mDecoratorBaseAdapter`��`getAnimators`�������װ����ʵ���Ķ������ڶ����֣���õ�ǰʵ���Ķ������������֣�`Alpha`��ʾ����������`item`ִ���������ֵĶ�����ϡ�
```java
private void animateViewIfNecessary(final int position, @NonNull final View view, @NonNull final ViewGroup parent) {
    ...
    Animator[] childAnimators;
    if (getDecoratedBaseAdapter() instanceof AnimationAdapter) {
        childAnimators = ((AnimationAdapter) getDecoratedBaseAdapter()).getAnimators(parent, view);
    } else {
        childAnimators = new Animator[0];
    }
    Animator[] animators = getAnimators(parent, view);
    Animator alphaAnimator = ObjectAnimator.ofFloat(view, ALPHA, 0, 1);
    /** ������������� */
    Animator[] concatAnimators = AnimatorUtil.concatAnimators(childAnimators, animators, alphaAnimator);
    /** ����ViewAnimator�ķ�������View��Ӷ��� */
    mViewAnimator.animateViewIfNecessary(position, view, concatAnimators);
}
```
>* `ResourceAnimationAdapter`
`AnimationAdatper`�����࣬`getAnimators`�����п���ͨ��`getAnimationResourceId`����Զ��嶯���������߿��Լ̳���������ಢʵ��`getAnimationResourceId`�������Ӷ�ʵ���Զ��嶯��Ч����
```java
public Animator[] getAnimators(@NonNull final ViewGroup parent, @NonNull final View view) {
    return new Animator[]{AnimatorInflater.loadAnimator(mContext, getAnimationResourceId())};
}
```
>* `SingleAnimationAdapter` `AnimationAdatper`�����࣬�ṩ�ĳ��󷽷�`getAnimator`������ʵ�֡�ͨ���̳���������ಢʵ��`getAnimator`�������Ӷ�ʵ���Զ��嶯��Ч����
>* `AlphaInAnimationAdapter`
`AnimationAdatper`�����࣬`getAnimator`������δ�����κζ�����
```java
protected Animator getAnimator(@NonNull final ViewGroup parent, @NonNull final View view) {
    return new Animator[0];
}
```
>* `ScaleInAnimationAdapter`
`AnimationAdatper`�����࣬`getAnimator`��������scale������
```java
protected Animator getAnimator(@NonNull final ViewGroup parent, @NonNull final View view) {
    ObjectAnimator scaleX = ObjectAnimator.ofFloat(view, SCALE_X, mScaleFrom, 1f);
    ObjectAnimator scaleY = ObjectAnimator.ofFloat(view, SCALE_Y, mScaleFrom, 1f);
    return new ObjectAnimator[]{scaleX, scaleY};
}
```
>* `SwingBottomInAnimationAdapter`
`AnimationAdatper`�����࣬`getAnimator`��������Y�᷽��������ϵĶ�����
```java
protected Animator getAnimator(@NonNull final ViewGroup parent, @NonNull final View view) {
    return ObjectAnimator.ofFloat(view, TRANSLATION_Y, parent.getMeasuredHeight() >> 1, 0);
}
```
>* `SwingLeftInAnimationAdapter`
`AnimationAdatper`�����࣬`getAnimator`�������ش������ҵĶ�����
```java
protected Animator getAnimator(@NonNull final ViewGroup parent, @NonNull final View view) {
    return ObjectAnimator.ofFloat(view, TRANSLATION_X, 0 - parent.getWidth(), 0);
}
```
>* `SwingRightInAnimationAdapter`
`AnimationAdatper`�����࣬`getAnimator`�������ش�������Ķ�����
```java
protected Animator getAnimator(@NonNull final ViewGroup parent, @NonNull final View view) {
    return ObjectAnimator.ofFloat(view, TRANSLATION_X, parent.getWidth(), 0);
}
```
---
4.2 `lib-manipulation`����Ҫ�ǲ�����View�Ĺ��ܣ�����`Swipe-to-Dismiss`��`Drag-and-Drop`��`Swipe-Undo`�ȡ�
>* `DynamicListView`
`ListViewAnimations`��������Ҫ���Զ���������̳���`ListView`���������¹��ܣ�`Drag-and-Drop`���ܣ���`DragAndDropHandler`��ʵ�֣�`Swipe-to-Dismiss`���ܣ���`SwipeTouchListener`�༰������ʵ�֣�`Swipe-Undo`���ܣ���`SwipeUndoAdapter`��ʵ�֣�`Animate addition`���ܣ���`AnimateAdditionAdapter`��ʵ�֡�
`DynamicListView`��д�˸���`ListView`��`setAdapter`������`dispatchTouchEvent`�������Ӷ���չ�˸���Ĺ��ܡ�����ͼ������ʾ��
![listviewanimations-DynamicListview-event.png](https://github.com/demonyan/android-open-project-analysis/blob/master/view/other/listview-animations/image/listviewanimations-DynamicListview-event.png)
`setAdapter`�������ڸ�`ListView`��`Adapter`������ͨ��`Adapter`�����ϻ��`rootAdapter`��������ʵ��`getView`�ӿڵ�`Adapter`��һ�����`Adapter`��Ҫ������ʵ�֡������`Adapter`���з���`SwipeUndoAdatper`��ʵ����˵����Ҫ`Swipe-Undo`���ܣ����ʼ��`mSwipeUndoAdapter`Ϊ��`SwipeUndoAdatper`��ʵ�������`rootAdapter`ʵ����`Insertable`�ӿڣ����ʼ��`mAnimationAdditionAdapter`�����`mDragAndDropHandler`��Ϊ�գ���Ϊ`mDragAndDropHandler`��`Adapter`��
```java
public void setAdapter(final ListAdapter adapter) {
    ListAdapter wrappedAdapter = adapter;
    mSwipeUndoAdapter = null;
    if (adapter instanceof BaseAdapter) {
        BaseAdapter rootAdapter = (BaseAdapter) wrappedAdapter;
        /** ����Adapter�����rootAdapter */
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
> `dispatchTouchEvent`�������ڴ���ַ������¼��������û��`TouchEventHandler`����`onTouch`�¼������Ƚ���`DragAndDropHandler`��`onTouchEvent`�����Դ�����¼������`onTouch`�¼�����������ק����������ô`TouchEventHandler`��Ϊ`DragAndDropHandler`��ͬʱ���`SwipeTouchListener`����`ACTION_CANCEL`�¼����Ժ��`ACTION_MOVE`�¼����ύ��`DragAndDropHandler`������ֱ��`ACTION_UP`�¼�����`ACTION_CANCEL`�¼����֡�����ύ��`SwipeTouchListener`�����Դ����¼������`onTouch`�¼�����Ử����������ô`TouchEventHandler`��Ϊ`SwipeTouchListener`��ͬʱ����`ACTION_CANCEL`�¼���`DragAndDropHandler`���Ժ��`ACTION_MOVE`�¼����ύ��`SwipeTouchListener`������ֱ��`ACTION_UP`�¼�����`ACTION_CANCEL`�¼����֡�
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
��������`onTouch`�¼���`onTouchEvent`�ӿڷ�����`item`���ڱ������`isInteracting`�ӿڷ�����`DragAndDropHandler`��`SwipeTouchListener`��ʵ���˸ýӿڡ�
>* `DragAndDropHandler`
ʵ��`TouchEventHandler`�ӿڣ�ʵ��`ListView`��`Drag-and-Drop`���ܡ������`onTouchEvent`��������`dispatchTouchEvent`�ַ�����`onTouch`�¼��������յ�`ACTION_DOWN`ʱ����¼��ָ��ѹ��λ�á�
�����յ�`ACTION_MOVE`ʱ�������ǰ��δ��ʼ��ק`item`��������ָ���ƾ���`deltaY`�����ٽ�ֵ`mSlop`����ͨ��`DraggableManage`�����ж��Ƿ�������ק`item`������������Ѿ���ʼ��ק`item`����ͨ����ָ��ק�����ж��Ƿ����������`item`����λ�ã�������`SDK`�汾ִ����Ӧ������`Kitkat`����ǰ�İ汾ִ��`KitKatSwitchViewAnimator`������`Kitkat`֮��İ汾ִ��`LSwitchViewAnimator`������
```java
private boolean handleMoveEvent(@NonNull final MotionEvent event) {
    ...
    if (mHoverDrawable == null && Math.abs(deltaY) > mSlop && Math.abs(deltaY) > Math.abs(deltaX)) {
        int position = mWrapper.pointToPosition((int) event.getX(), (int) event.getY());
        if (position != AdapterView.INVALID_POSITION) {
            View downView = mWrapper.getChildAt(position - mWrapper.getFirstVisiblePosition());
            assert downView != null;
            if (mDraggableManager.isDraggable(downView, position - mWrapper.getHeaderViewsCount(), event.getX() - downView.getX(), event.getY() - downView.getY())) {
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
> �����յ�`ACTION_UP`ʱ��������ָ�ɿ�ʱ�ĵ�ǰλ����ȷ��`item`������λ��.

>* `DraggableManager`
�ж��û��Ƿ������ק`item`�Ľӿڡ�ͨ��ʵ�ָýӿڵ�isDraggable�����������Զ���������ק��������
>* `TouchViewDraggableManager` ʵ����`DraggableManager`�ӿڣ�ͨ���Ƿ����ض���`View`�Ӵ����ж��û�������ק`item`��
>* `SwipeTouchListener`
ʵ��`OnTouchListener`�ӿں�`TouchEventHandler`�ӿڣ�ʹ��`ListView`��`item`�ܹ�`swipeable`��������Ҫʵ��`afterViewFling`���󷽷�������ȷ��ָ�Ử��Ĳ����������`onTouchEvent`��������`dispatchTouchEvent`�ַ�����`onTouch`�¼��������յ�`ACTION_DOWN`ʱ��ͨ��`DismissableManager`�����ж�`item`�Ƿ����ɾ�������`DynamicListView`�ĸ�����Ҳ����ˮƽ�������������Ҫͨ��`requestDisallowInterceptTouchEvent`������Ҫ����`onTouch`�¼���
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
> �����յ�`ACTION_MOVE`ʱ���ж���ָ�Ử����`deltaX`�Ƿ�����ٽ�ֵ`mSlop`����������`ListView`����һ��`ACTION_CANCEL`�¼���ʹ��`ListView`���ٴ���`ACTION_MOVE`�¼����Ӷ����ᴥ��`item`�ı���������Ȼ��ͨ����ָ�Ử�ľ��뿪ʼ`item`���򻬶��Ķ���Ч����
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
> �����յ�`ACTION_CANCEL`�¼�ʱ��`item`��ص���ʼ״̬��
> �����յ�`ACTION_UP`�¼�ʱ��ͨ����ָ�Ử�ľ�����ٶ����ж��Ƿ���Ҫɾ����ǰ`item`�������Ҫ����ʾ`UndoView`�����û������վ����Ƿ�ɾ����ǰ`item`��

>* `SwipeDismissTouchListener`
`SwipeTouchListener`�����࣬ʵ����ָ�Ử��`item`��ɾ��������ɾ���ɹ������`OnDismissCallback`���ص�������ʵ�ֵ��߼���
>* `SwipeUndoTouchListener`
`SwipeDismissTouchListener`�����࣬����ָ�Ử��������û�ѡ���Ƿ���ɾ����������ֱ��ɾ����View��������ʾ��������`UndoView`�����û�ȷ�ϳ�������ʱ��ʾ����������
>* `AnimateAdditionAdaptor`
`BaseAdaperDecorator`�����࣬ʵ�ֲ���`item`ʱ�Ķ���Ч��������װ�ε�`rootAdapter`����ʵ��`Insertable`�ӿڣ������췽�����׳��쳣��
>* `SwipeDismissAdapter`
`BaseAdapterDecorator`�����࣬������Ա����`mDismissTouchListener`�ͳ�Ա����`mOnDismissCallback`��ͨ����װ���࣬��ͨ��`ListView`(����ʹ��`DynamicListView`)Ҳ����ʵ��`Swipe-to-Dismiss`���ܡ���д����`setListViewWrapper`������ʼ����Ա������`mDismissTouchListener`������`ListView`����`onTouch`��������`mOnDismissCallback`��ɾ��`item`ʱ�Ļص��ӿڡ�
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
����`getUndoView`�ӿڷ�����`getUndoClickView`�ӿڷ����������Ҫ`Undo`���ܣ�����ʵ�ָýӿڷ�����
>* `SwipeUndoAdapter`
`BaseAdaperDecorator`�����࣬Ϊ`ListView`���`Swipe-Undo`��Ϊ�����ó�Ա����`mSwipeUndoTouchListener`��`undo`����ִ�г����������ָ�`item`�ĳ�ʼ״̬�����ߵ��ó�Ա����`mSwipeUndoTouchListener`��`dismiss`����ɾ������λ�õ�`item`��
>* `SimpleSwipeUndoAdapter` �̳�`SwipeUndoAdapter`�ಢʵ��`UndoCallback`�ӿڣ�`getView`�����лὫ`primaryView`����`undoView`��ʾ��`item`����װ�ε�`Adapter`����ʵ��`UndoAdapter`�ӿڣ����������쳣��
```java
public View getView(final int position, @Nullable final View convertView, @NonNull final ViewGroup parent) {
    SwipeUndoView view = (SwipeUndoView) convertView;
    if (view == null) {
        view = new SwipeUndoView(mContext);
    }
    View primaryView = super.getView(position, view.getPrimaryView(), view);
    view.setPrimaryView(primaryView);
    View undoView = mUndoAdapter.getUndoView(position, view.getUndoView(), view);
    view.setUndoView(undoView);
    mUndoAdapter.getUndoClickView(undoView).setOnClickListener(new UndoClickListener(view, position));
    /** ��ʾprimaryView����undoView */
    boolean isInUndoState = mUndoPositions.contains(position);
    primaryView.setVisibility(isInUndoState ? View.GONE : View.VISIBLE);
    undoView.setVisibility(isInUndoState ? View.VISIBLE : View.GONE);
    return view;
}
```
>* `ExpandableListItemAdapter`
`ArrayAdapter`�����࣬ʵ�ֵ��`item`��`Expandable`��Ϊ����������Ҫ�̳иó������ಢʵ��`getTitleView`������`getContentView`������`getTitleView`����������ʾ`Title`����ͼ��`getContentView`����������ʾ`Content`����ͼ��

4.3 `lib-core-slh`���Ժ��Ŀ����չ������֧��`StickListHeaders`����
`StickyListHeaders`ʹ����`Android`Ҳ������`iOS`һ������`ListView`���������`Header`�Ŀ�Դ�⣬���Բο����ĺ���������ӡ�
>* `StickyListHeadersAdapterDecorator`
�̳���`BaseAdapterDecorator`�ಢʵ��`StickyListHeadersAdapter`�ӿڣ���`getHeaderView`�����и�`headView`��Ӷ�������װ�ε�`Adapter`����ʵ��`StickyListHeadersAdapter`�ӿڣ����������쳣��
```java
public View getHeaderView(final int position, final View convertView, final ViewGroup parent) {
    ...
    View itemView = mStickyListHeadersAdapter.getHeaderView(position, convertView, parent);
    animateViewIfNecessary(position, itemView, parent);
    return itemView;
}
```
>* `StickyListHeadersListViewWrapper`
`StickyListHeadersListView`�İ�װ�࣬ʵ��`ListViewWrapper`�Ľӿڷ�����

## 5. ��̸
����`ListViewAnimations`��ʵ���г�������װ���Լ��ص��ӿڣ�����ɶ��Բ��ߡ�ͨ�����Դ�룬ͬʱ���`Demo`ѧϰ�ÿ⣬���˽�`ListView`ʵ��˼·��`View`���¼��ַ����ƣ�Android�������������ģʽ������������ĿǰRecyclerView�������е����Ƶõ����������ʹ�ã�`ListViewAnimations`���`feature_recyclervier`��֧��ʵ���˶�`RecyclerView`��֧�֡����߿���Ϊ�˱���`master`̫���Ӷ�������ʹ�ò��㣬���߾���`RecyclerView`��`item`������֧���Ѿ��㹻�������㣬`mater`��֧��δ������`RecyclerView`��֧�֡�

### �ο�����
1. [Android���ģʽԴ�����֮�Ž�ģʽ](https://github.com/simple-android-framework/android_design_patterns_analysis/tree/master/bridge/shen0834)
1. [����������֮ View �¼�����](http://a.codekk.com/detail/Android/Trinea/%E5%85%AC%E5%85%B1%E6%8A%80%E6%9C%AF%E7%82%B9%E4%B9%8B%20View%20%E4%BA%8B%E4%BB%B6%E4%BC%A0%E9%80%92)
1. [����������֮ Android ��������](http://a.codekk.com/detail/Android/lightSky/%E5%85%AC%E5%85%B1%E6%8A%80%E6%9C%AF%E7%82%B9%E4%B9%8B%20Android%20%E5%8A%A8%E7%94%BB%E5%9F%BA%E7%A1%80)
1. [StickyListHeaders](https://github.com/emilsjolander/StickyListHeaders)

