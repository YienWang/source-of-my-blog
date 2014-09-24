---
layout: post
title: "常用开源库的使用"
date: 2014-09-07 11:49:51 +0800
comments: true
tags: [开源,Android]
---
常用开源库的介绍，自己的备忘录兼使用之后自己的小总结。不定期更新。  

1. ActiveAndroid
2. Gson

<!--more-->

ActiveAndroid
-----------------
一个简单的ORM库。  
个人不是太喜欢，但是如果只是随手做一些小东西的话还是蛮方便的，适合于快速原型开发。
使用步骤：

1. 初始化ActiveAndroid
```java
	@Override public void onCreate() {
		super.onCreate();
		ActiveAndroid.initialize(this);
	}
```
2. 声明模型类
表名由 `@Table` 声明, 行则由 `@Column`声明。注意表名中可显式声明id，通常建议为`_id`，可供CursorAdapter使用。
**一定要继承Model模型，且提供空构造器，且调用super方法**
```java
@Table(name = "task", id = "_id")
public class Task extends Model {

	public static final SimpleDateFormat SDF_DATE_FORMAT = new SimpleDateFormat(
			"yyyy/MM/dd", Locale.CHINA);

	public Task() {
		this("NULL");
	}
	@Column(name = "content") private String content;
}
```
3. 具体操作
使用起来比较简单。操作均在Model中定义了。
```java
new Task(content).save(); //插入操作
task.delete();	//任务的删除
mTasks = new Select().from(Task.class).execute(); //获取列表
//order
new Select().from(Task.class).orderBy("tt ASC").execute(); //DESC
```
4. （可选）定义数据库的名字和版本
正如上面所说，由于只想用来当快速原型开发工具。名字什么的就随意啦~(^ ^)~。如果真的要定义，那么就在AndroidManifest里面声明两个元数据。
**注意是在Application结点。**
```xml
 <meta-data android:name="AA_DB_NAME" android:value="Pickrand.db" />
 <meta-data android:name="AA_DB_VERSION" android:value="5" />
```

5. 升级数据库版本
创建一个文件夹 `assets/migrations`，修改 `AA_DB_VERSION`的值（大于当前版本），然后创建一个文件 `x.sql`，其中x为对应的版本号。
如果是更改表的结构，可以像这样子：
```sql
ALTER TABLE focus_time ADD COLUMN remark TEXT DEFAULT 'NO REASON';
```
在修改表结构的同时设置默认值。    
其他资料：[update databae][update_database]

ButterKnife
-----------------
好像ButterKnife在整个根节点就是所要id的时候就会出异常


[update_database]: http://stackoverflow.com/questions/4253804/insert-new-column-into-table-in-sqlite