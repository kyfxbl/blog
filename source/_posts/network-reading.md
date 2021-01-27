title: 《计算机网络》阅读感想
date: 2013-09-24 11:25
categories: 读书笔记
---
![networkbook](http://pic.kyfxbl.com/a37.jpg)
最近出差罗马尼亚，下班之后就在看《计算机网络》，今天看完了“网络层”，谈谈感想
<!--more-->

# 分层 

无论是OSI的7层模型，还是TCP/IP的5层模型，都体现了分层的思想。各层只做自己的事情，依赖下层提供的服务，但不依赖下层的细节，下层不依赖上层（NAT是一个例外）

看起来和应用开发的分层架构多么相像。我认为最大的好处就在于，每一层的改动，都不会对其他层造成影响，于是可以放心地修改；另外，分层便于实现分布式部署（比如说，数据链路层在网卡和交换机里实现，网络层在路由器里实现） 

# 服务与协议分离 

N层为N+1层提供的服务，可以类比为JAVA中的“接口”；支撑服务的协议，则类比为JAVA中的“实现” 

从N+1层来看，只是简单地调用第N层的服务，而不关注第N层的协议细节。比如说，在网络层拿到帧数据包后，只是简单地将下层的“帧头”抛弃掉，只需要关注该层的“IP数据包”；而传输层拿到IP数据包之后，又简单地抛弃了“IP头”，只关注该层的“TCP数据包”。第N层协议的细节，在N+1层是不感知的 

这使得下层的实现可以随意替换。对于应用层来说，传输层使用TCP协议还是UDP协议，都是毫无影响的，这大大降低了应用层开发的难度。就像在JAVA开发中，DAL层使用hibernate，或者mybatis，或者jdbc，对于Service层也是毫无影响，只要保证DAO的接口是稳定的 

# 对等进程 

两个交互的进程，通信实际上是先自顶向下，再自底向上传输的；但是在逻辑上，仿佛这两个进程是直接通信的一样。比如两个应用要想通信，他们的传输层必须都理解TCP协议，或者都理解UDP协议，否则就无法正常通信 

类似的，web service的客户端和服务端，经常是异构的。比如服务端是用java开发的，而客户端是用C#开发的，那么在应用层（不是说网络模型里的应用层，这里可以大致理解为业务逻辑层），是不能互相理解的。但是在应用层下面，都需要理解soap协议。服务端的这一层，负责将java数据和SOAP协议的转换；在客户端，则要转换c#数据和SOAP协议 

# 逻辑和物理分离 

物理的以太网，称为LAN；逻辑上的以太网，则是VLAN 

多个LAN可以组成一个VLAN，相应的，一个LAN也可以拆分成多个VLAN。在控制网络连通性的同时，却可以不影响物理的连接（并不需要把网线插来拔去），非常方便 

类似的，在servlet规范里，每个servlet都有一个URL name，和一个class name，通过deployment name来映射 

如果class name的实现替换或者修改了，完全不影响访问。最直接的一个好处是，页面里写的URL，都不需要修改了。这也算是逻辑和物理的分离 

如果URL直接对应实现类，那么一旦实现类发生变化，原有的URL就全部失效了，只要是稍微大一点的应用，到处修改URL的工作量都是非常大的，相当于要把整个公司的网线都检查重连一遍 

# 总结 

这些都是最基本的原理，如果不提上下文的话，很难分辨说的是计算机网络，还是应用程序开发。很有一种“万法同一”的感觉，最近几年，所学很杂，但是常常会有这样的感觉：不同领域的知识，往往在本质上是相通的，而且可以互相印证，联系。有时候想想，觉得十分奇妙