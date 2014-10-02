---
layout: post
title: "android media player"
date: 2014-09-08 10:46:45
tags: [android]
comments: true
---
关于Android的MediaPlayer。
很常见的类，不过稍微有点忘了。所以小总结一下，以防自己忘记。
<!--more-->
MediaPlayer也就是播放音乐时需要的工具类（跟AudioManager不一样，后者是负责控制手机的音量，情景模式之类的）  
MediaPlayer是一个典型的状态机，也就是说只能从一个状态跳转到另外一个合法的状态。比如说，在start之前，你必须要先调用prepare()或者prepareAsync()。
#### 初始化
最简单粗暴的初始化：
```java
MediaPlayer mp = MediaPlayer.create(context, R.raw.xxx);
mp.start();
```

按照官方的说明，这种方式有个限制：
> the content of this resource should not be raw audio. It should be a properly encoded and formatted media file in one of the supported formats.


设置文件路径的初始化：
```java
String url = "http://........"; // your URL here
MediaPlayer mediaPlayer = new MediaPlayer();
mediaPlayer.setAudioStreamType(AudioManager.STREAM_MUSIC);
mediaPlayer.setDataSource(url);
mediaPlayer.prepare(); // might take long! (for buffering, etc)
mediaPlayer.start();
```
注意 `prepare()` 默认是在调用的同一线程上执行的，且会占用较长时间，因此会建议使用异步方式，并实现回调 `onPrepared()` 。举个官方的栗子：
```java
public class MyService extends Service implements MediaPlayer.OnPreparedListener {
    private static final String ACTION_PLAY = "com.example.action.PLAY";
    MediaPlayer mMediaPlayer = null;

    public int onStartCommand(Intent intent, int flags, int startId) {
        ...
        if (intent.getAction().equals(ACTION_PLAY)) {
            mMediaPlayer = ... // initialize it here
            mMediaPlayer.setOnPreparedListener(this);
            mMediaPlayer.prepareAsync(); // prepare async to not block main thread
        }
    }

    /** Called when MediaPlayer is ready */
    public void onPrepared(MediaPlayer player) {
        player.start();
    }
}
```
#### 销毁
在不需要播放的时候，记得调用释放MediaPlayer占用的资源，并置引用为null。比如在Activity中：
```java
protected void onStop() {
	super.onStop();
	mp.release();
	mp = null；
}
```

#### 后台播放
后台播放大概只能用Service来做--因为它是能够在Activity不可见之后仍然能够继续执行的。
关于后台播放，那又有两种方式：  
1. 使用后台服务播放：这个时候音乐播放器是用户不可见的，只能听到声音
2. 使用前台服务播放：这个时候在通知栏或者其他地方会有可见的提示，用户也可以操作。

### 资料
官方对MediaPlayer的说明： [Mediaplayer][Media_Player]

[Media_Player]: http://developer.android.com/guide/topics/media/mediaplayer.html