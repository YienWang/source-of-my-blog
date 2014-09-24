---
layout: post
title: "ListView 优化"
date: 2014-08-25 09:24:56
comments: true
tags: [Android,ListView,调优]
---
关于ListView的优化，想必都是老生长谈了。什么“面试必问”的问题之类的，感觉这个问题也差不多该自己总计下了。
<!-- more -->
**复用convertView及使用ViewHolder**
------
这是最最最简单的方式，也是一提到优化就会想起来的，但也是非常具有效率的方式。（还有一个是尽量使布局flat-这个就不另外提出来了）  
在Adapter中getView的时候，先判断convertView是否为空，再进行分别操作。这样子inflate的操作其实也就是前面几次而已（根据第一屏的能容纳的项目数来定）。  
而使用ViewHolder则节省了findView的时间。
```java
	@Override public View getView(int position, View convertView, ViewGroup parent) {
		ViewHolder mHolder = null;
		if (convertView == null) {
			// 实战中千万不用这么做，每次都创建一个inflater实例也太蛋疼了，可以在外面创建一个，然后第一次创建的时候保存起来
			convertView = LayoutInflater.from(parent.getContext()).inflate(
					android.R.layout.simple_expandable_list_item_1, parent,
					false); 
			mHolder = new ViewHolder(convertView);
			convertView.setTag(mHolder);
		} else {
			mHolder = convertView.getTag();		
		}

		return convertView;
	}
	// 注意static
	static class ViewHolder {
		TextView mTextView;
		public ViewHolder(View convertView) {
			mTextView = (TextView) convertView.findViewById(android.R.id.text1);
		}
	}
```
**使用恰当大小的Bitmap及缓存Adapter中使用的图片。**
------------
读取图片时缩放成适当的大小：
```java
	public Bitmap getScaleBitmap(Context context, int height, int width, int resId) {
		BitmapFactory.Options options = new BitmapFactory.Options();
		//设置该属性时，BitmapFactory的decord返回的Bitmap会是null，但是options中的属性会被设置。
		options.inJustDecodeBounds = true;
		BitmapFactory.decodeResource(context.getResources(), resId, options);
		options.inSampleSize = calInSampleSize(height, width, options);
		//真正需要获取图片的时候记得把标志设置为false，表示真的要decode了
		options.inJustDecodeBounds = false;
		return BitmapFactory.decodeResource(context.getResources(), resId,
				options);
	}
	//计算inSampleSize【始终觉得哪里有些冗余
	private int calInSampleSize(int height, int width, Options options) {
		int sampleSize = 1;
		int outHeight = options.outHeight;
		int outWidth = options.outWidth;

		if (outHeight > height || outWidth > width) {
			final int halfHeight = outHeight >> 1;
			final int halfWidth = outWidth >> 1;
			while (halfHeight / sampleSize > height
					|| halfWidth / sampleSize > width) {
				sampleSize <<= 1;
			}
		}
		return sampleSize;
	}
```
使用LruCache来缓存图片：
```java
class BitmapCache {
	final int MAX_SIZE = (int) Runtime.getRuntime().maxMemory() / 8;//详情看补充
	LruCache<Integer, Bitmap> mLruCache = new LruCache<Integer, Bitmap>(MAX_SIZE) {
		@Override protected int sizeOf(Integer key, Bitmap value) {
			return value.getByteCount();
		}
	};

	public Bitmap getBitmap(Context context, int height, int width, int resId) {
		Bitmap mBitmap = mLruCache.get(resId);
		if (mBitmap == null) {
			mBitmap = getScaleBitmap(context, height, width, resId);
			mLruCache.put(resId, mBitmap);
		}
		return mBitmap;
	}
}
```
**不要在adapter的getView中做过多操作（包括创建对象）**
---------------
因为adapter的getView是在ListView每次获取对象的时候都会调用的，因此里面创建对象跟一些逻辑操作都需要尽量地少。另外，可以结合上面的convertView，在判断convertView为空的情况中，给ViewHolder中的控件设置统一的属性（当然我也不觉得这有必要就是了）
**滑动时暂停加载，滑动结束再继续加载图片**
------------------
这个适用于异步加载图片的情况（或者说如果需要图片，使用异步加载才是正确的选择）。  
异步加载，其实也就是将获取图片（从缓存中获取，从资源中获取，从网络中获取等等），操作图片（缩放图片）这样的费时操作放到另外的线程中执行。  
需要注意的是，这里需要传入ImageView这个引用，同时需要对应的位置的信息---可以在ImageView里面设置tag）。传入引用的时候需要使用过WeakReference包裹住ImageView这个引用（因为如果是强引用，可能会导致adapter无法被GC，导致内存泄露；使用WeakReference时记得先检查ImageView是否为null，同时位置信息是否匹配------因为ListView复用机制，有可能导致设置的时候错位）。  
**待仔细研究一下Volley的源码怎么实现正确的下载~**
```java
listView.setOnScrollListener(new OnScrollListener() {
     @Override public void onScrollStateChanged(AbsListView listView, int scrollState) {
    	 // Pause disk cache access to ensure smoother scrolling
         if (scrollState == AbsListView.OnScrollListener.SCROLL_STATE_FLING)
			imageLoader.stopProcessingQueue();
   	  else 
     		imageLoader.startProcessingQueue();
	}
	@Override public void onScroll(AbsListView view, int firstVisibleItem, int visibleItemCount, int totalItemCount) {}
	}
);
```
**使用Curadapter获取字符串时使用copyStringToBuffer**
---------------
这个优化仅限于CurorAdapter，因为需要频繁getString的adapter只有从数据库获取数据的时候。优化的原理也很简单，跟StringBuilder拼接字符串类似，减少了创建对象的次数。举个例子：
```java
private static classViewHolder {
	CharArrayBuffer titleBuffer= newCharArrayBuffer(128);
	CharArrayBuffer subtitleBuffer= newCharArrayBuffer(128);
}
@Override public voidbindView(View view, Context context, Cursor cursor) {
	final TwoLineListItem item = (TwoLineListItem) view;
	final ViewHolder holder = (ViewHolder) view.getTag();
	final CharArrayBuffer titleBuffer = holder.titleBuffer;
	cursor.copyStringToBuffer(DataQuery.TITLE, titleBuffer);
	item.getText1().setText(titleBuffer.data, 0, titleBuffer.sizeCopied);
	final CharArrayBuffer subtitleBuffer = holder.titleBuffer;
	cursor.copyStringToBuffer(DataQuery.SUBTITLE, subtitleBuffer);
	item.getText2().setText(subtitleBuffer.data, 0, subtitleBuffer.sizeCopied);
}
@Override publicView newView(Context context, Cursor cursor, ViewGroup parent) {
	final View v = LayoutInflater.from(context).inflate(
	android.R.layout.two_line_list_item, parent, false);
	v.setTag(newViewHolder());
	returnv;
}
```

**补充：**
==================
**关于Android中使用的内存大小** [AndroidHeap][AndroidHeap]
------------
简单点说，获取的方式有两种：
```java
//1 获取到的单位是Byte
Runtime.getRuntime().maxMemory();
//2 获取到的单位是MB
ActivityManager am = (ActivityManager) getSystemService(ACTIVITY_SERVICE);
int memoryClass = am.getMemoryClass();
```
在大多数情况下，Runtime获取到的会跟am获取到的一致（如果你没在CM系统上设置HeadSize的话。如果设置得比第二种方式小，那么恭喜你，你程序不知道什么时候就会崩了~
**关于缓存时设置的引用**
----------
最好不要使用WeakReference<Bitmap>来做缓存的Value(而且感觉SoftReference也不适合用来当value，因为SoftReference被回收的时候，LruCache是不清楚的，因而无法改变size大小。(可以自己包装一个方法，但是也无法保证引用在被回收之前调用并且改变LruCache的size)):  
>Weak values will be garbage collected once they are weakly reachable. This makes them a poor candidate for caching; consider softValues() instead. -from [guava][guava]

> Soft references are for implementing **memory-sensitive** caches, weak references are for implementing **canonicalizing mappings that do not prevent their keys (or values) from being reclaimed**, and phantom references are for **scheduling pre-mortem cleanup actions in a more flexible way** than is possible with the Java finalization mechanism.  --from [JavaDoc][JavaDoc]

##参考的资料
[Tricks to boost performance of list view][1] (墙外）


[guava]: http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/cache/CacheBuilder.html#weakValues()
[JavaDoc]: http://docs.oracle.com/javase/1.5.0/docs/api/java/lang/ref/package-summary.html
[AndroidHeap]: http://stackoverflow.com/questions/2630158/detect-application-heap-size-in-android/9428660#9428660
[1]: http://optimizationtricks.blogspot.com/2014/01/tricks-to-boost-performance-of-list-view.html