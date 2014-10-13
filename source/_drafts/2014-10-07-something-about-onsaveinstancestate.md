title: 关于onSaveInstanceState的那点事
date: 2014-10-07 19:52:21
tags: [android,持久化]
---
前言
----------
之前对这个函数功能不是太了解，在[阅读《Android开发精要》][1]这本书的过程中发现关于这个函数的描述存在矛盾。这个在Activity的生命周期中可能涉及到的函数做了什么，又是怎么做的，又有什么需要注意的，不弄懂的话似乎说不过去吖
<!--more-->

正文
-------------
### 做了什么
默认的情况下是怎么做的,先来看下官方的文档：
> 默认实现是调用`View.onSaveInstanceState()`将大部分有指定id的View状态保存下来，如果存在focusedView也会保存对应的id信息。这些设置的信息都会在`onRestoreInstanceState()`的默认实现中恢复。如果你重写了这个方法来保存没被View保存的信息，记得调用`super`来调用默认实现，否则做好自己手动保存所有View状态的准备~(**注：**默认View是什么都不保存的


官方是如此说明的，那么就跟踪一下代码如何？
### 怎么做的 
从Activity开始：
```java
// android/app/Activity.class
protected void onSaveInstanceState(Bundle outState) {
  // 调用了Window保存层次信息，Window是一个抽象类，实现类是PhoneWindow
  outState.putBundle(WINDOW_HIERARCHY_TAG,mWindow.saveHierarchyState());
  ...
}
```
再看看在Window的实现类里面做了什么：
```java
//com/android/internal/policy/impl/PhoneWindow.java
@Override public Bundle saveHierarchyState() {
        Bundle outState = new Bundle();
	// 如果没有设置mContentParent,即它为null，则不需要保存任何信息直接返回
	// mContentParent为空的情况也就是没有调用installDecor的情况
	// 也即没有发生任何获取窗口的View以及设置添加View的行为,如setContentView
        if (mContentParent == null) {
            return outState;
        }
        SparseArray<Parcelable> states = new SparseArray<Parcelable>();
	// 保存整个ViewGroup的层次信息
        mContentParent.saveHierarchyState(states);
        outState.putSparseParcelableArray(VIEWS_TAG, states);
	// 保存当前ViewGroup的focusedView
        // save the focused view id
        View focusedView = mContentParent.findFocus();
        if (focusedView != null) {
            if (focusedView.getId() != View.NO_ID) {
                outState.putInt(FOCUSED_ID_TAG, focusedView.getId());
            } else {
		...// log something
            }
        }
	... // save the panels
	... // save ActionBar state
        return outState;
}
```
这里调用了`mContentParent.saveHierarchyState(states)`来保存整个ViewGroup的状态信息,其中`mContentParent = generateLayout(mDecor);`。如果有了解过Window的界面布局的话，会知道mContentParent其实也就是我们`setContentView()`时添加的布局。  
再看看`mContentParent`做了什么：
```java
// android/view.class
public void saveHierarchyState(SparseArray<Parcelable> container) {
      dispatchSaveInstanceState(container);
}
protected void dispatchSaveInstanceState(SparseArray<Parcelable> container) {
	// 只有存在ID且允许保存状态的View才会被保存状态，否则忽略
        if (mID != NO_ID && (mViewFlags & SAVE_DISABLED_MASK) == 0) {
            mPrivateFlags &= ~PFLAG_SAVE_STATE_CALLED;
            Parcelable state = onSaveInstanceState();
	// 继承类，如果没有调用super来保存父类的信息那么就抛出异常
    	// 这里有点小技巧，通过与操作取得了该设置位的信息
            if ((mPrivateFlags & PFLAG_SAVE_STATE_CALLED) == 0) {
                throw new IllegalStateException(
                        "Derived class did not call super.onSaveInstanceState()");
            }
            if (state != null) {
                container.put(mID, state);
         }
     }
}
```
mContentParent调用的`saveHierarchyState()`实际是View中实现的方法，而在ViewGroup只是重写了`dispatchSaveInstanceState()`这个方法来分发保存状态的事件到ChildView：
```java
	// android/ViewGroup.class
    @Override protected void dispatchSaveInstanceState(SparseArray<Parcelable> container) {
        super.dispatchSaveInstanceState(container);
        final int count = mChildrenCount;
        final View[] children = mChildren;
        for (int i = 0; i < count; i++) {
            View c = children[i];
		   // 如果ViewGroup没有设置不需要保存状态才保存ChildView
            if ((c.mViewFlags & PARENT_SAVE_DISABLED_MASK) != PARENT_SAVE_DISABLED) {
                c.dispatchSaveInstanceState(container);
            }
        }
    }
```
基本过程就是这样子，简单地总结下它怎么做的话就是
*onSaveInstanceState默认将从Window开始调用有id且有需要的View的onSaveInstanceState方法来保存它们自己所需要的信息*（其实还会有保存fragment信息的过程，在这篇博客中暂时忽略~）

### 存在哪里
做完上面那堆事之后，留有所有信息的那个outState又存到哪去了？  
具体存储的过程可以在ActivityThread的源码中看到，它将这个bundle（也就是SavaInstanceState）存储在了ActivityClientRecord这个类的实例中。  
在《Android Programming》中提到的内容**基本**可信，因此这里不再跟踪源码。【注意只是基本，看了一小节就发现有错误了】
> The bundle is then stuffed(被塞进）into your activity's activity record by OS.

那么这个record什么时候会被销毁？  
按照书上的说法就是如果你按下了返回键，那么Activity就被销毁了，对应的Activity record也不复存在。另外在机器重启，或者这个Activity长期没再使用时，这个record也会被销毁。

### 什么时候
那么它又在什么时候调用的咧。
> 为了能够在重新创建的某个时间点上来恢复它的状态，这个方法会在一个Activity可能被销毁前被调用。
> 不要跟Activity的生命周期回调中搞混。考虑下面三个场景
> 1. onPause和onStop都被调用，但这个方法没被调用：从B导航回A，此时没有调用B的onSaveInstanceState方法，因为B不需要被恢复。
> 2. onPause被调用，但是onSaveInstanceState不调用的情况：当B在A之后被启动
> Do not confuse this method with activity lifecycle callbacks such as onPause, which is always called when an activity is being placed in the background or on its way to destruction, or onStop which is called before destruction. 
> An example when onPause is called and not onSaveInstanceState is when activity B is launched in front of activity A: the system may avoid calling onSaveInstanceState on activity A if it isn't killed during the lifetime of B since the state of the user interface of A will stay intact. 
> 如果这个方法真会被调用，那么只能保证它会在onStop之前调用，无法保证它是在onPause之前还是之后被调用。

### 需要注意的地方
#### 关于View的id
其实从mContentParent分发保存状态事件的过程中就可以发现，默认是使用ID来作为SparseArray的key的，那么就会存在无法保存状态或者状态异常的情况。
1. 没有设置ID
2. 设置了同名id

第一种情况基本不是问题，因为对于重要的控件一般都会设置一个id，但第二种情况就会比较麻烦了。设置同名id会导致后面出现的控件覆盖前面同名控件所保存的信息，而且还不会有任何警告。在查资料的过程中就发现了这样子的一个[例子][2]。

#### onRestoreInstanceState
这是跟onSaveInstance对应的方法，对应的实现过程就不再做分析。只谈谈这个方法调用的时机。

补充
----------

勘误
----------------
> 以下勘误都仅限Android Programming这本书

P80, 在*'Saving Data Across Rotation'*这一节中，提到的onSaveInstanceState的默认实现有误，不是所有的views，而是有id的view。另外它还提到能放在SaveInstanceState里面的只能是基本类型和实现了Serializable接口的类，但明显实现Parcelable也是可以的，或者说更推荐后面这种方式。

资料
--------------
《Android Programming: The Big Nerd Ranch Guide》 

[1]: {{root_url}}/2014/10/01/reading-android-kaifa-jingyao/
[2]: http://www.cnblogs.com/xiaoweiz/p/3813914.html
[3]: http://eigo.co.uk/labs/managing-state-in-an-android-activity/
[4]: http://stackoverflow.com/questions/4096169/onsaveinstancestate-and-onrestoreinstancestate