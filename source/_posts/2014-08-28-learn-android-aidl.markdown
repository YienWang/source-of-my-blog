---
title: 初探 Android AIDL
date: 2014-08-28 11:04:39
comments: true
tags: [android,AIDL]
---
前言
-------------------
AIDL, *Android Interface Defination Languge*, Android的接口定义语言，是为了简化为跨进程通信写代理代码所准备的一种语言。  
只要写了aidl文件且满足语法，那么IDE就可以自动生成对应IInterface文件，使用的时候直接继承来实现这个抽象类来进行IPC（跨进程通信），非常方便~
<!--more-->

正文
-------------
### 使用AIDL的基本步骤：
>**在使用AIDL之前，请先确定一下，你是不是真的需要一个能为多个应用服务的，能绑定的，多线程的远程进程来执行你的任务。如果不是，考虑一下直接用LocalService来bind，或者IPC的Messenger来做单线程操作更加适合**  

#### **定义AIDL接口**
每个AIDL文件只能定义一个接口，且只能声明接口和函数签名。
```java
package com.example.android;
interface IRemoteService {
	int getPid();
	void basic(int aInt, long aLong,String aString);
} 
```
#### **实现接口的Stub**  
继承生成的类中Stub，实现之前声明的RPC接口。  
默认情况下，RPC调用是同步的。且不会有任何异常反馈给调用方。
#### **将接口暴露给客户端**  
继承Service，实现`onBind()`函数，将之前实现的Stub的类成员返回。  
在调用 `bindService()` 时传入的 `ServiceConnection` 变量定义的`onServiceConnected()`函数中，通过 `IRemoteService.Stub.asInterface(mService)`将返回的 `iBinder` 转成 `IRemoteService`(其实也就是客户端了，Stub是桩，作为代理对象而存在）。  
#### **通过IPC传递对象**   
对于要传递的对象，你需要先实现Parcelable接口，然后创建一个aidl文件来指明这个对象的所在的包。关于Parcelable，见备注。  
#### **调用IPC方法**
直接通过IRemoteService这个对象调用就好了~它会自动将请求通过代理转发给另外一个进程的对应对象，让它执行函数。  
如果需要执行回调，那么就需要额外的工作。定义一个回调接口 `IRemoteServiceCallback`
```java
oneway interface IRemoteServiceCallback {
 /** Called when the service has a new value for you.*/
    void valueChanged(int value);
}
```
将接口作为参数传给另外一个interface  
```java
import ....IRemoteServiceCallback;
interface IRemoteService {
    void registerCallback(IRemoteServiceCallback cb);
    void unregisterCallback(IRemoteServiceCallback cb);
}
```
在远程服务中实现的时候，使用`RemoteCallBackList<IInterface>`管理回调。在调用时注册事件，在完成事件时处理回调，在服务退出的时候销毁所有事件。
```java
final RemoteCallbackList<IRemoteServiceCallback> mCallbacks = new RemoteCallbackList<IRemoteServiceCallback>();
//需要处理回调的事件的时候调用已经注册的回调
Handler handler = new Handler() {
	public void handleMessage(Message msg){
		//类似于写时拷贝，创建一个拷贝之后放开锁
		final int N = mCallbacks.beginBroadcast();
		for (int i = 0; i < N; i++) {
		try {
			mCallbacks.getBroadcastItem(i).valueChanged(value);
		} catch (RemoteException e) {
			// The RemoteCallbackList will take care of removing
			// the dead object for us.
			}
		}
		mCallbacks.finishBroadcast();
	}
}
@Override public void onDestory() {
	super.onDestory();
	mCallbacks.kill();
}
```

### 定义需要在AIDL中传输的数据
默认情况下只支持基本类型和CharSequence，ArrayList，Map，String，其他数据需要自己定义，并导入。  
导入方式基本按照如下aidl文件定义--必须先添加对应aidl文件才可以在接口中导入需要的包。
```java
// Weather.aidl 
// 这个AIDL接口文件用于说明Weather这个对象所在的位置。
// 如果没有这个文件，那么使用者就无法调用到Parcelable对象
// 如果有了这个文件，那么在gen文件夹下就会出现一个对应的package
// 这个文件要放在对应的package中
package com.demo.contract.model;
parcelable Weather;
```

### 定义接口时需要注意的地方：
- 方法可以带有一个或多个参数，返回或不返回值
- 所有的非元类型参数都需要一个方向标识用来指明数据来源。可以是in，out或者inout。元类型默认是in，且不能为其他标识。~~（对参数进行marshalling代价是昂贵的，所以尽量控制）(大错特错！）~~

>All non-primitive parameters require a directional tag indicating which way the data goes. Either in, out, or inout (see the example below).
Primitives are in by default, and cannot be otherwise.  
**Caution:**   
You should limit the direction to what is truly needed, because marshalling parameters is expensive.  
这段话的意思是，你应该对方向进行一个限制。因为对参数进行marshalling的代价是高昂的。in的意思是只需要读入，out的意思是在操作中会被写入，inout是既需要输入，又需要输出。
另外见补充。  

- 只有方法是暴露的，不能在AIDL中暴露静态域。
- AIDL的实现必须是完全线程安全的（因为要满足多个线程同时调用）

**补充**
-----------
### 关于序列化接口 Parcelable
举个栗子：
```
public class Weather implements Parcelable {
	private WeatherInfo info;
	public Weather(Parcel parcel) {
		info = parcel.readParcelable(this.getClass().getClassLoader());
	}
	@Override public int describeContents() {
		return 0;
	}
	@Override public void writeToParcel(Parcel dest, int flags) {
		dest.writeParcelable(info, flags);
	}
	public static final Parcelable.Creator<Weather> CREATOR = new Creator<Weather>() {
		@Override public Weather createFromParcel(Parcel source) {
			return new Weather(source);
		}
		@Override public Weather[] newArray(int size) {
			return new Weather[size];
		}
	};
}
```
主要是多了个以 `Parcel`为参数的构造函数，静态的Creator成员，`writeToParcel()` 方法。构造函数跟write方法要注意的地方就是**两者的写入顺序必须一致**，不是逆序，请注意。 
### 在Parcelable对象中包含另外的对象
- 如果Parceable对象里面包含另外一个Parcelable对象，那么在read的时候需要指明ClassLoader：
```java
public User(Parcel source) {
	this.name = source.readParcelable(this.getClass().getClassLoader());
	this.pwd = source.readParcelable(this.getClass().getClassLoader());
}
```
否则会出现如下异常（在加载类的时候出错）：
```java
E/Parcel: Class not found when unmarshalling: com.example.parcelablepro.Name, e: java.lang.ClassNotFoundException: xxx 
```
- 如果使用了Bundle来作为传输载体，那么在从bundle中读取Parcelable对象的时候，记得先给它设置当前class的ClassLoader。（很常见，如果数据过多的情况下使用这种方式或者才是正确的，不然要一个一个对齐，容易出错）。
```java
public User(Parcel source) {
	Bundle bundle = source.readBundle();
	bundle.setClassLoader(this.getClass().getClassLoader());
	this.name = bundle.getParcelable("Name");
	this.pwd = bundle.getParcelable("Pwd");
}
```
否则会出现这种异常（在解析数据的时候出错）：
```java
E/AndroidRuntime: Caused by: android.os.BadParcelableException: ClassNotFoundException when unmarshalling: xxx
```
那是为什么呢？  
在Bundle的 `unparcel()` 函数中有如下一句：
```java
mParcelledData.readArrayMapInternal(mMap, N, mClassLoader);
```
使用 `mClassLoader` 来读取Parcel对象，而 `mClassLoader` 在默认的构造函数中传入的是当前类的类加载器：
```java
public Bundle() {
    mMap = new ArrayMap<String, Object>();
    mClassLoader = getClass().getClassLoader();
}
```
检查了一下两个类Bundle和自定义类对应的类加载器：
```java
D/Class: APK classloader: dalvik.system.PathClassLoader[dexPath=/mnt/asec/com.example.parcelablepro-1/pkg.apk,libraryPath=/mnt/asec/com.example.parcelablepro-1/lib]
D/Class: FrameWork classloader: java.lang.BootClassLoader@41af8ed0
//注意不要在log里面用this，不然拿到的类加载器也会是BootClassLoader
```
可以发现，两者使用的类加载器并不一致。前者是PathClassLoader，而后者是BootClassLoader。其中BootClassLoader是PathClassLoader的父代加载器，由于类加载的双亲委派机制，无论BootClassLoader如何委托，都无法将类交到PathClassLoader中。最终抛出异常。  
根据StackOverflow上的解释，框架层和APK层调用的类加载器是不一样的。具体可见另外一篇博文 [关于android的类加载器][android-class-loader]

### 关于方向的tag
>**StackOverFlow**:  
In AIDL, the "out" tag specifies an output-only parameter. In other words, it's a parameter that contains no interesting data on input, but will be filled with data during the method.   
This is important because the contents of every parameter must be marshalled (serialized, transmitted, received, and deserialized). The in/out tags allow the Binder to skip the marshalling step for better performance.

### 特殊的关键字：`oneway` 
这个关键字用于说明在根据是否RPC进行不同的操作
>The oneway keyword modifies the behavior of remote calls. When used, **a remote call does not block**; it simply sends the transaction data and immediately returns. The implementation of the interface eventually receives this as a regular call from the Binder thread pool as a normal remote call. If oneway is used with a local call, there is no impact and the call is still synchronous.

>The oneway keyword means that if that call results in an IPC (i.e. the caller and callee are in different processes) then the calling process will not wait for the called process to handle the IPC. If it does not result in an IPC (i.e. they're both in the same process), the call will be synchronous. It's an unfortunate detail that simplifies the implementation of binder IPC a lot. If they're in the same process, the call is just a regular java method call. --stackoverflow
>这个关键字用于说明如果方法是跨进程方式调用，则调用者不会等待被调用者做完所有事情之后才退出方法，而是直接返回。如果非跨进程的，那么就相当于一个同步方法，调用直到退出。

[android_aidl]: http://developer.android.com/guide/components/aidl.html

[android-class-loader]: {{root_url}}/2014/08/30/learn-android-class-loader/