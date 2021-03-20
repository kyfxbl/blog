---
title: QQ音乐网页版同步播放列表的方案
date: 2021-03-20 21:48:52
categories: web
---
![qqmusic](http://pic.kyfxbl.com/music.jpg)
最近用QQ音乐网页版听歌，发现一个不错的体验，当已经打开一个播放列表的时候，如果再播放另一首新歌，这首新歌会自动加到播放列表中。觉得有些好奇就研究了一下实现的原理
<!--more-->

# 效果

效果是类似下面这样的，已经打开了播放列表，是chrome中一个单独的页签
![1](http://pic.kyfxbl.com/qqmusic1.png)

然后在另一个单独页签中，播放另一首新歌，会发现自动加入了播放列表
![2](http://pic.kyfxbl.com/qqmusic2.png)

![3](http://pic.kyfxbl.com/qqmusic3.png)

# 原理

原本猜想是不是跟服务端建立了长连接，播放新歌的时候服务端给播放列表页面推送刷新请求。但是在chrome里查看network和application，既没有发现有长连接，也没有ajax请求。看来不是从服务端触发的刷新，确实这样效率也不高。

其实本质就是chrome不同的页签之间如何通信的问题，于是想到local storage，查看application发现
![4](http://pic.kyfxbl.com/qqmusic4.png)

看key非常靠谱，看来很可能是基于local storage实现的，那么肯定是播放页面写了local storage，在列表页面读取local storage，于是进一步检查js

果然找到一段相关的代码，打断点试试
![5](http://pic.kyfxbl.com/qqmusic5.png)

# 总结

对于同域的多个页面，通过local storage来做跨页签通信是一个不错的办法，而且比通过服务端来交互更加高效