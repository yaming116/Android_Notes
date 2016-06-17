此处记录网易新闻的广告业的实现思路
=======

最近看到越来越多的app内置的欢迎广告，就想着如何实现它，最后发现是挺简单的，但是自己又有点懒惰了，先记录下来。
欢迎页内置有两个好处：

* 一个不影响软件的加载
* 增加一些信息的曝光量，不论是添加广告还是自己应用内置活动的推广

页面布局结构如下：

![](){NetEase-Welcome}{https://raw.githubusercontent.com/yaming116/Android_Notes/post/2016/static/NetEase-Welcome.png}

# 实现这个需要做的事情

* 自定义一个等比的ImageView
* ImageView点击的时候增加一层遮罩
* 右上角添加一个倒计时的跳过
* 底部是一个应用信息的展示


# 可以参考实现的资料

![ForegroundLinearLayout](https://gist.github.com/chrisbanes/9091754)
![DownloadProgressBar](https://github.com/panwrona/DownloadProgressBar)