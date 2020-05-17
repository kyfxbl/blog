title: 基于CAS实现前后端分离的SSO
date: 2020-05-17 21:15:22
categories: web 
---
![sso](http://pic.kyfxbl.com/a49.jpg)

一种基于标准CAS，实现前后端分离的SSO的方案
<!--more-->

# 基础知识

一般同域的SSO，用共享session就可以实现了，常见于各微服务都是自己开发的情况。更普遍的场景是跨域集成的SSO，这时候一般采用标准的CAS方案。一个标准的CAS认证流程如下图，已经画得非常清楚了

![cas](http://pic.kyfxbl.com/cas_flow_diagram.png)

大致上是这样的一个流程，当用户请求应用服务器提供的某个服务`abc`，如果没有登录过，那么会返回一个302，跳转到`cas/login?service=abc`，在CAS登录后，又会302到`abc?ticket=STxxxxx`（这里abc就是前面service填的地址），应用服务器用这个ticket到CAS Server认证，如果OK的话，再一次302到`abc`，完成整个认证过程

## 重要的概念

TGT：Ticket Granting Ticket，这是CAS Server里保存的，用于认证用户已经在CAS登录过

TGC：Ticket Granting Cookie，跟上面的TGT对应，是已经认证过的用户在浏览器里保存的cookie

ST：Service Ticket，这是CAS Server发给应用服务器的凭证，应用服务器再用ST到CAS Server认证

## 常用filter

如果应用服务器是用JAVA写的，那么通常会用到下面几个filter：

一个是AuthenticationFilter，这个filter拦截请求，判断是否认证。如果没有认证，就跳转到CAS Server认证

另一个是TicketFilter，这个filter用于将ST拿到CAS Server认证

# 前后端分离的场景

现在web应用一般采用前后端分离的架构，通过ajax请求后端服务，这样会无法跳转回原来的静态页面，需要在CAS流程的基础上稍加改造

主要思路是让页面在发起业务请求之前，先确保登录完成。在后端提供auth服务用于鉴权，和validate服务用于验证ST，并跳转回初始静态页面，流程如下图

![portal](http://pic.kyfxbl.com/portalsso.png)

这种方式比起标准的CAS流程，主要差别在于不完全依赖于HTTP response code和浏览器默认行为，在前端javascript中做了控制。比如说普通的情况，后端是返回一个302 `cas/login?service=abc`，在这个方案里，跳转逻辑是直接写在前端javascript里的

这里对service参数做了定制并encoding，传的是`cas/login?service=abc?originUrl=xxx`，于是当CAS Server登录之后，返回的是302 `abc?originUrl=xxx&ticket=STxxxxx`，这里的xxx是验证完成后跳转回的原始静态页面