---
layout: post
title: "用Path画个小太极"
date: 2014-08-24 16:37:39
comments: true
tags: [Android,Path]
---

最近两天一直在折腾博客，想着弄好之后就来写这一篇东西。我想要用Canvas跟Path画出一个太极，并使其旋转。  
最后要实现的效果是这个样子的：
<!-- more -->
<img alt="figure 1" src={{root_url}}/img/android-path/taiji_04.jpg width=300px height=300px/>
  
1. 一开始的想法很简单，就是外面一个大圆，左右两个怀玉一样的部分，先画左边一半，再对左边的Path进行一个180度旋转，再填充另外一种颜色就可以实现。	
但是在尝试的过程中发现，Path中添加弧线的函数会无法连接不同Rect的两段弧线，即使第一段弧线的终点跟第二段弧线的起点一致。  
两天前的实现结果是这样子的：  
![figure2][taiji00]![figure3][taiji01]
```java
public void initPath() {
	mPath.addOval(mRectF, Direction.CCW);
	mBlackPath.addArc(mTopRectF, -90, 180);
	mBlackPath.addArc(mBottomRectF, 90, 180);
	mBlackPath.addArc(mRectF, 90, 180);
}
protected void onDraw(Canvas canvas) {
	canvas.drawPath(mPath, mBorderPaint);
	canvas.drawPath(mBlackPath, mPaint);
	canvas.drawPath(mWhitePath, mWhitePaint);
}

```
可以从图中看到了吧，这里原本的打算是从上方开始一直画两个相反方向的半圆，再从底部的半圆绕大圆左边回到顶部，路径似乎是对的，但是实际上addArc还是重新创建了一条路径出来，对API的解释也有些疑问：
>Add the specified arc to the path as a new contour (轮廓).

~~作为新的轮廓，其实也就是添加了新的路径的意思了吧。而Path的FillType是只对闭合曲线有效的。新创建的轮廓其实也就相当于新开了一条路径了(存疑)，这时的 `FillType.EVEN_ODD` 也就对Path无效了。~~（其实并不是，只是ADT有病而已！见下面更新）  

2. 折腾了两天博客之后，再开始想如何实现太极要怎么实现。就发觉我忽略了某种最方便的方法--下面的那个半圆我只需要用白色画笔来填充就好了，这样子就可以产生某种‘正确的错觉’了。如下面的左图，这里因为背景是灰色的，所以可以很明显地看到右边的半圆是透明的（我也是在手机上才发觉的- -)。因此需要先把底色设置为白色，如右图：  
![figure3][taiji02]![figure4][taiji03]
```java
public void initPath() {
	mPath.addOval(mRectF, Direction.CCW);
	mBlackPath.addArc(mTopRectF, -90, 180);
	mBlackPath.addArc(mRectF, 90, 180);
	mWhitePath.addArc(mBottomRectF, 90, 180);
}
protected void onDraw(Canvas canvas) {
	canvas.drawPath(mPath, mBorderPaint); //绘制黑色边界
	canvas.drawPath(mPath, mWhitePaint); //填充白色背景
	canvas.drawPath(mBlackPath, mPaint); //绘制黑色部分
	canvas.drawPath(mWhitePath, mWhitePaint); //覆盖部分黑色
}
```  
3. 接下来就是绘制上下两个点了。上面那个白色的点绘制起来还是比较简单的，直接在白色的路径 `mWhitePath` 里面加一个半圆即可。但是下面那个黑色的点就没法这么做了，因为黑色路径在白色路径之前被绘制，所以黑色路径画出来小黑点有一半会被白色覆盖掉。没办法，只能再加一条路径了。最后画出来的效果是这样子的：  
![figure5][taiji04]
```java
private void initPath() {
	// mWhitePath.reset();
	// mBlackPointPath.reset();
	mPath.addOval(mRectF, Direction.CCW);
	mBlackPath.addArc(mTopRectF, -90, 180);
	mBlackPath.addArc(mRectF, 90, 180);
	mWhitePath.addCircle(mTopRectF.centerX(), mTopRectF.centerY(),mTopRectF.height() / 10, Direction.CCW);
	mWhitePath.addArc(mBottomRectF, 90, 180);
	mBlackPointPath.addCircle(mBottomRectF.centerX(),mBottomRectF.centerY(), mBottomRectF.height() / 10,Direction.CCW);
}
protected void onDraw(Canvas canvas) {
	canvas.drawPath(mPath, mBorderPaint);
	canvas.drawPath(mPath, mWhitePaint);
	canvas.drawPath(mBlackPath, mPaint);
	canvas.drawPath(mWhitePath, mWhitePaint);
	canvas.drawPath(mBlackPointPath, mPaint);//绘制下方黑点
}
```
关于上面代码中被注释的两行： 这是个很奇怪的现象- -一开始没加上这两句，导致画面中图形的左上角会出现一个小半圆。  
突然发觉了原因~实际上这个时候还是有在左上角画出一个1/4圆的，只是这个时候半径为0了---原因就是我把小圆点的半径设置成了 `m*Rect.height()/10` ,而 `m*Rect` 初始高度为0。  
一开始会有小圆点，也是因为RectF的初始值为(0,0)点，导致小圆的路径在一开始就被添加进去了，没有重置的话就会一直绘制下去，因此在再次调用initPath的时候就要重置一下。
4. 接下来就是添加动画了~
动画的效果，之前一直想着用 `Matrix` 来对Path进行操作。代码差不多应该是这样子的：
```java
mMatrix.postRotate(degrees);
mWhitePath.transform(mMatrix);
...
degrees++;
```
但是这样子就需要对多个路径同时transform，好麻烦啊。  
突然间想起了canvas也是可以旋转的嘛！于是就直接操作Canvas啦~
```java
@Override protected void onDraw(Canvas canvas) {
	//按照中心点来旋转特定角度，注意如果不指定的话是按照canvas的原点来旋转
	canvas.rotate(degrees, mCenterPoint.x, mCenterPoint.y); 
	canvas.drawPath(mPath, mBorderPaint);
	canvas.drawPath(mPath, mWhitePaint);
	canvas.drawPath(mBlackPath, mPaint);
	canvas.drawPath(mWhitePath, mWhitePaint);
	canvas.drawPath(mBlackPointPath, mPaint);
	increate();
}
private void increate() {
	degrees += 4;
	invalidate(); //一定要记得调用invalide啊！不然会停住不动的
}

```
####重要的更新 8-25 19:47
我被eclipse骗了= =eclipse的xml里面是无法展示Path的FillType的，AndroidStudio也是。也就是说一开始我是想对了，现在只要像下面这么做就可以了。
```java
	private void initPath() {
		mBlackPath.setFillType(FillType.EVEN_ODD);
		mBlackPath.reset();
		mPath.addOval(mRectF, Direction.CCW);
		mBlackPath.addArc(mTopRectF, -90, 180);
		mBlackPath.addArc(mRectF, 90, 180);
		mBlackPath.addArc(mBottomRectF, 90, 180);
		mBlackPath.addCircle(mBottomRectF.centerX(), mBottomRectF.centerY(),
				mBottomRectF.width() * 0.16f, Direction.CCW); // 绘制上面的黑色小圆点
		mBlackPath.addCircle(mTopRectF.centerX(), mTopRectF.centerY(),
				mTopRectF.width() * 0.16f, Direction.CCW); // 再画一遍，以此来使其透出小白点
	}

	@Override
	protected void onDraw(Canvas canvas) {
		canvas.rotate(degrees, mCenterPoint.x, mCenterPoint.y);
		canvas.drawPath(mPath, mBorderPaint);
		canvas.drawPath(mPath, mWhitePaint);
		canvas.drawPath(mBlackPath, mPaint);
		increate();
	}
```


###小总结
感觉我有把问题想象得比原来难的习惯，比如一开始的半圆，我总想着各种麻烦的方案，然后不动手。其实稍微取巧一下也是应该的吧，如果找不出答案或者懒得去做，就找个更懒的方法啊，不要总想着**“虽然知道怎么做，但是看起来好麻烦啊，不想做”**，这样的话就不会成长了。


------------
**补充：**
===============
**Path中有如下函数**：
```java
public void close(); 				//关闭曲线，如果终点跟起点不同，则用lineTo连接到起点
public void quadTo(x1, y1, x2, y2); //使用贝塞尔曲线连接两个点
public void arcTo(...); 			//绘制特定矩形内的弧线，注意0度是3点钟方向
public void reset(...); 			//重置Path中的路径，但是不重置FillType
```
**关于Path的FillType**  
[一个关于FillType的网页][filltype]  
**WARNING** Eclipse的实时预览功能无法展现Path的FillType（坑我呢，难怪一直没弄出来！）

[taiji00]: {{root_url}}/img/android-path/taiji_00.jpg  
[taiji01]: {{root_url}}/img/android-path/taiji_01.jpg
[taiji02]: {{root_url}}/img/android-path/taiji_02.jpg  
[taiji03]: {{root_url}}/img/android-path/taiji_03.jpg
[taiji04]: {{root_url}}/img/android-path/taiji_04.jpg
[filltype]: http://www.imobilebbs.com/wordpress/archives/1589

