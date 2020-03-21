title: 一些厉害的CSS技巧
date: 2014-11-06 21:21
categories: web 
---
![css](http://pic.kyfxbl.com/css.jpeg)
本文介绍一些厉害的CSS小技巧
<!--more-->

# 纯CSS实现高度与宽度成比例的效果

一个两列比例布局，比如左边栏80%，右边栏20%，所以右边栏的width是不知道的，但是无论右边栏的width是多少，都要求height和width保持1：1的比例。不需要js，用纯css也可以实现这个效果

原理主要是2点：

1、当设置padding为百分比的时候，参照是父元素的width，这点和设置子元素的width行为是一致的。比如这个DOM：

```
<div style="width: 100px">
    <div id="son">abc</div>
</div>
```

```
#son {
    width: 20%;
    padding-bottom: 20%;
}
```

计算出son的width和padding-bottom都是相对于parent的20%，即都是20px，这样就可以实现高宽比的任意比例

2、overflow的计算包括了content和padding，所以只要padding够大，即使content是0，内容一样可以显示出来

所以，结合以上2点，就可以用纯CSS实现高度宽度固定比例的效果：

```
#son {
    width: 20%;
    padding-bottom: 20%;
    height: 0;
}
```

# 子元素垂直方向居中

在父元素高度不确定的情况下，令子元素在垂直方向上自适应居中，通过简单的CSS就能做到

效果图：

![vertical](http://pic.kyfxbl.com/vertical.jpg)

效果图里卡片形状的区域，height是不固定的。因为width是根据屏幕的宽度自适应，并且height和width保持固定比例。所以在PC上，height可能是300px，但是在手机上就只有100px

希望中间的剩余金额保持垂直居中显示，方法有很多，我用的是绝对定位 + 百分比的方法。关键CSS如下：

```
<div id="parent">
    <div>季度卡 vip86900006</div>
    <div id="son">3次</div>
</div>
```

```
#parent {
    position: absolute;
    width: 50%;
}

#son {
    position: absolute;
    top: 40%;
    width: 100%;
    text-align: center;
}
```

因为父容器的高度不确定，所以top不能写成固定的值，可以写成百分比，这样就会根据父容器的宽度自适应

# 实现平铺滚动效果

还是上面那张效果图，实现原生应用平铺滚动的效果，利用简单的CSS也可以做到。思路是在父容器上设置水平方向滚动，并禁止换行。然后子元素用inline-block实现平铺。关键CSS如下：

```
<ul>
    <li></li>
    <li></li>
    <li></li>
</ul>
```

```
ul {
    width: 100%;
    overflow: scroll;
    white-space: nowrap;
}

li {
    display: inline-block;
    width: 50%;
    vertical-align: top;
}
```

# 兼容移动端的两列布局

用inline-block实现两列布局是一种常见做法，不过需要处理手机浏览器的兼容性问题

下面这个DOM结构

```
<div>
    <div>div1</div>
    <div>div2</div>
</div>
```

使用inline-block的方式实现2列布局：

```
div {
    font-size: 0;
}

div > div {
    display: inline-block;
    width: 50%;
    font-size: 14px;
}
```

虽然在PC上可以解决1px间隙的问题，但是在很多手机浏览器上（android 4.2以下），会有兼容性问题。右边的div会掉到下面

所以更好的办法是：

```
div {
    overflow: hidden;
}

div > div {
    float: left;
    width: 50%;
}
```

# 修复浏览器上臭名昭著的1px blank问题

下面这个dom结构：
```
<div>
    <div id="div1"></div>
    <div id="div2"></div>
</div>
```
通过inline-block的方法让2个子div水平排列：
```
div > div {
    display: inline-block;
    width: 50%;
}
```

实践发现，在chrome下，div2会掉到下面去。调试以后发现，多了1px神秘的间隙，然后2个div的width都是50%，所以总的width变成100% + 1，超过了父div的width，结果第二个div就掉下去了

最简单的改法，可以把html写到同一行：

```
<div>
    <div id="div1"></div><div id="div2"></div>
</div>
```
但是这种方法显然不好，不仅html的格式很糟，而且代码格式化，平台差异，自己忘记了等等因素都会使bug重现。比较好的方法，是设置font-size为0，然后在具体的div里再覆盖font-size

```
#parent {
    font-size: 0;
}

.son {
    display: inline-block;
    width: 50%;
    font-size: 1rem;
}
```
