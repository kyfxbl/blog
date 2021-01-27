title: 前端的松耦合
date: 2013-10-14 00:04
categories: web 
---
![decouple](http://pic.kyfxbl.com/decouple.png)
今天没事又翻了翻Nicholas Zakas的《可维护的javascript》，总结一点感想
<!--more-->

书里有句话说得很好：
___在一起工作的组件无法达到无耦合（no coupling），在所有系统中，组件之间总要共享一些信息来完成各自的工作___

这里组件的概念是很宽泛的，2个类，2个js文件，2个系统，都可以认为是不同的组件。如果2个组件不需要协同完成工作，那么其实没必要放在一个上下文里讨论，只有这种情况下才是无耦合的

但是只要组件需要配合，耦合就是不可避免的，区别在于是松耦合还是紧耦合。应该追求的是松耦合，我总结了主要有2点：

1、组件之间通过某种方式来交换信息，这种方式称为接口，接口不能变

2、除了接口之外，组件在内部可以随意修改，不会对其他组件产生影响，也不需要假设其他组件作出修改

比如在典型的三层架构中，把service层和dal层看作2个组件，dal层暴露的CRUD就是接口（在java平台就是interface），这个接口确定后要避免修改，否则2个模块都需要改动

但是，在接口稳定的前提下，dal层内部实现的变化，即使是底层从oracle换成了mongodb，也不会影响service层。这样就认为service和dal做到了松耦合。类似的例子可以举出很多，相信有开发经验的人都有体会。这里主要想说一下这个思想在前端的体现

在前端页面里，css和javascript绝对是需要协同工作的，所以不可能无耦合，问题是怎么做到松耦合

有一种互动的方式，是直接在javascript里修改css属性，比如

```
element.style.color = "red"
```
这就是在javascript里直接修改了DOM元素的css属性，是一种紧耦合，不好的写法，比较好的写法应该是：

```
element.className += " reveal"
```

```
.reveal {
    color:red;
}
```

这里className就是CSS和js交换信息的方式，除此以外，CSS可以随意修改reveal的具体样式，而不需要修改js；反之js也一样。做到松耦合

另外还有js文件协同工作，也可以举个例子：

传统的做法是a.js定义一些global函数或者变量，b.js直接用。这种方式也是紧耦合的方式，如果a.js修改了全局变量，b也就要改动，而且还会引进潜在的BUG

比较好的做法是a.js只定义唯一的全局变量作为接口，b只依赖这个接口。如jQuery提供的$就是这种做法

现在javascript关于模块和依赖管理这块也很有进步了，有了AMD，CommonJS等机制来使不同的js编译单元松耦合