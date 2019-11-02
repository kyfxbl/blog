title: 借助chrome developer tool开发移动设备web应用
date: 2014-01-16 21:38
categories: 其它
---
我们的终端应用，是native + cordova方式。html和javascript跑在终端设备里，调试比较困难，只能通过alert和console等手段。用chrome developer tool，可以让html和js跑在浏览器里，解决这个问题
<!--more-->

详细的文档在：[chrome developer tool doc](https://developers.google.com/chrome-developer-tools/docs/mobile-emulation?hl=zh-cn)

# 启动

```
sudo /Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --disable-enforced-throttling --enable-vertical-tabs --allow-file-access-from-files
```
这是因为html和javascript都是从本地加载的，浏览器默认是禁止的，上面的参数就是改变这个默认行为

# 模拟tap事件

然后还有一个问题，移动终端的点击事件不是click，而是tap，所以需要模拟这个动作

先在配置里开启Emulation，然后在Sensors里勾中emulate touch screen，就可以模拟tap事件了