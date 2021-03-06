title: 松鼠书读书笔记
date: 2013-09-24 11:15
categories: 读书笔记  
---
![songshu](http://pic.kyfxbl.com/a41.jpg)
大名鼎鼎的松鼠书读书笔记
<!--more-->

# HTTP概述

## URI、URL、URN 

这3个概念很容易混淆

URI：是Uniform Resource Identifier的缩写，统一资源标识符 

URL：是Uniform Resource Location的缩写，统一资源定位符 

URN：是Uniform Resource Name的缩写，统一资源名 

关系上，URI就是用来唯一标识一个信息资源，而它有两种表示方式，就是URL和URN；但是URN现在还在试验阶段，没有大规模推广，所以基本上现在所有的URI都是URL。也就是说，在目前，可以粗略认为URI=URL，无视URN 

URL的格式一般是schema://address/resource，比如`http://www.abc.com/special/xxx.gif`，这个URL里，协议是http，服务器是`www.abc.com`，资源是/special/xxx.gif（在这里，http服务器应用，负责实现提供/special/xxx.gif这个资源的方法即可，不一定就放在special文件夹下） 

## http事务 

指的就是一次http请求 + http响应 

## http事务的过程 

在http客户端向服务端发送请求报文之前，需要用IP地址和TCP端口号，在客户端和服务端之间建立一条tcp/ip连接。在http服务端向客户端发送响应报文之后，服务端会关闭这条tcp/ip连接 

以浏览器输入一个URL`http://www.abc.com/special/xxx.gif`为例 

a) 浏览器从URL中解析截取出主机名`www.abc.com`
b) 浏览器通过DNS，将`www.abc.com`转换成服务端ip地址 
c) 浏览器从URL中解析截取出端口号，此处没有，使用默认的80端口 
d) 浏览器建立tcp/ip连接 
e) 浏览器向服务端发送http请求报文 
f) 服务端向浏览器发送http响应报文 
g) 服务端关闭tcp/ip连接 

书里没有明确说tcp连接是由服务端还是浏览器负责断开的。但是试验了一下，用telnet向一个http服务器发送请求，在得到响应之后，会看到Connection closed by foreign host。所以认为是http服务端负责关闭连接的 

## 一些HTTP应用的概念 

proxy：位于client和server之间，负责处理和转发client的http请求，可以实现安全、突破限制等许多好处。[详细介绍](http://zh.wikipedia.org/wiki/%E4%BB%A3%E7%90%86%E6%9C%8D%E5%8A%A1%E5%99%A8) 

cache：缓存网络资源，这个很好理解，我觉得应该跟proxy有点关系，或者可以一起实现。maven私服nexus就有这个功能，可以认为是一个proxy，也是一个cache 

gateway：这个以前看RFC2616不大理解，今天就懂了，也就是一个协议转换的应用，比如一个http/ftp网关。客户端对其发起http请求，它实际通过ftp协议获取资源之后，再向客户端发回http响应 

tunnel：这个好难懂，原来也不知道是干啥的。貌似是将实际要传输的数据，封装到http报文里，来达到穿越防火墙的目的。这样的话，tunnel应该也是需要有client和server两端的，tunnel client将请求封装到http报文里，然后tunnel server从http报文里解析出请求，再将响应封装到http报文里，最后tunnel client再从报文里解析出响应 

agent：客户端代理，这个比较好理解，所有发起http请求的应用，都是client agent，最常见的当然就是各种浏览器，此外还有网络爬虫之类的，都是agent。前面说的tunnel client，按这个定义来说，也是agent了；包括proxy，对于最终服务器来说，也是agent

# HTTP报文

## 报文的一些术语 

在规范里有这样2组术语，需要知道它们的意思，才能理解后面的内容 

一组是流入/流出，即inbound和outbound。“流入”总是指http message从client agent发往server；“流出”总是指http message从server发往client agent 

另一组术语是上游/下游，即upstream和downstream，http message总是从上游发往下游。两个http节点A和B，如果对http request来说，A是B的上游；那么对http response来说，A就是B的下游 

## http message的结构 

无论是http request还是http response，http message都由3部分组成，分别是starting line、header、entity-body 

entity-body翻译为“实体的主体部分”，很拗口，而且实际上并没有“实体的不是主体的部分”，还不如就翻译成body，或者entity好点 

用apache的http client组件的话，API里有一个getEntity()方法，得到的就是http message的entity-body部分 

## 状态码 

http response的起始行里，会返回一个状态码： 

1xx：信息提示 
2xx：成功 
3xx：重定向 
4xx：客户端请求错误 
5xx：服务端错误 

一般特别常见的就是200、304、404、500等 

## header 

首部就是一行一行的键值对，用冒号分隔，最后用一个空行表示结束。首部对于http message是至关重要的，很多功能都是依靠首部来完成的

# 连接管理

## 与tcp/ip的关系

世界上几乎所有的HTTP通信都是由TCP/IP承载的，所以基本上，一次HTTP事务的过程，就是客户端首先与服务端建立TCP连接，然后客户端发送一条http请求，服务端发送一条http响应，最后断开TCP连接 

我们常说的端口号，是TCP端口号；而主机名，则会被解析成IP地址。IP地址用于连接到正确的计算机上，而端口号则用于连接到正确的应用程序上

TCP连接通过4个值来识别： 源IP、源端口、目的IP、目的端口 

比如从我的机器上，开了一个IE连到百度，又开了一个chrome也连到百度，那么这个时候就是这样的： 

本机IP、IE端口、220.181.111.147、80 
本机IP、chrome端口、220.181.111.147、80 

## 与socket的关系

socket是一组操纵TCP/IP的编程API。由于其设计精巧，并且对应用层程序员完全隐藏了TCP/IP的细节，所以基本上，所有需要用到TCP/IP的程序，都会使用socket接口来完成 

但是并不能说“HTTP是建立在socket基础上的”。HTTP是实实在在的应用层协议，而socket是简化TCP/IP编程的API接口，其本身并不是一个协议 

socket最早是为UNIX开发的，现在几乎所有的操作系统和语言中都有它，比如JAVA中，就有整套Socket API 

## 长连接

HTTP/1.1允许HTTP在事务结束之后，将TCP连接保持在打开状态，未来的HTTP请求可以重用该连接。这样的话可以节省重新建立TCP连接的时间，提升性能。这样的连接叫做“持久连接” 

但是，其实持久连接，是可以用来做更多事情的，比如服务端推送。但是HTTP协议仅规定client agent未来发送http request时重用，没有规定server利用该连接实现推送的相关细节。所以server push在HTTP/1.1里并没有官方标准

即使持久连接已经建立，服务端并不知道通过什么标准往client agent反推消息；client agent也没有标准的做法，在连接上监听等待server发来的消息，并进行相应的处理

# proxy

个人理解，这些概念（代理、缓存、web server）有重合，只是RFC2616上给出的一种指导性的分类，并没有严格的区分，都是互联网上的一些节点，或者说是http应用而已

比如说，我开发了一个私有的代理服务器，部署在internet的入口处，然后配置浏览器请求往这个私有代理服务器上发。这个proxy有缓存的功能，如果是已经有的资源，就不向服务器发起请求了，那么这个proxy，也就可以认为是一个cache。同时，它也可以算web server。所以这个私人代理，同时是proxy、cache、web server 

## http proxy的主要功能
 
利用http proxy可以实现很多功能

## 资源filter 

这个很好理解，很多公司为了限制员工访问互联网，都搞一个公司代理，把白名单之外的请求都过滤了 

## 防火墙

同理，从安全角度考虑，在LAN和WAN中间放一个proxy，然后就可以在proxy上部署一些杀毒软件，流量监控之类的东西 

## 缓存
 
由proxy给回响应，不需要重复请求

## 反向代理（reverse proxy） 

很多大型的互联网网站都会这么做，一方面可以增强性能，另一方面可以保护server的安全 

## 转码器（transcoder）

比如同一个页面，在返回给client agent之前，经过转码器转一下。如果client agent是PC浏览器则不作处理；如果是手机，就转成适合手机浏览器阅读的内容。很明显transcoder这里就需要做很多逻辑处理 

## 匿名者 

转发http request时，把user-agent首部等篡改，来保护真实客户端的安全。如果个人PC被黑客攻击了作为跳板，那有时候就成了黑客的匿名者代理 

## 总结

可以看出，其实本质都是一样的。在真正的client agent，和最终的server中，部署一些http应用，对http request或者http response做一些处理，这就是http proxy的作用 

在现实中也有很多体现：

比如电信宽带，想连到网通服务器打网游，为了不卡，就花钱买个代理。当然这个一般不会是HTTP代理，但是原理都是一样的，本来直接发到网通服务器的消息，都会发到代理服务器上，由代理服务器来转发，再把网通服务器的响应给回客户端。代理服务器一般是双线的，所以玩游戏会比你直接连到网通服务器流畅一些 

还有我97年那会通过猫拨号上网，国外的网站是不能上的，那么就可以找一些代理服务器，就能上国外的网站了 

还有有些国外的服务器，会BAN掉大陆IP，那这个时候也可以找个海外服务器做代理，就可以连上去了

# cookie

## cookie的本质

cookie就是client在发送请求的时候，会额外发送一些键值对到server，这样server读取了这些信息，就可以识别client了 

server给回响应的时候，会带一个set-cookie或者set-cookie2的首部；之后client再发送请求的时候，就用cookie首部，把cookie信息发送到server去

```
GET /index.html HTTP/1.0
Host:www.abc.com
```

```
HTTP/1.0 200 OK
Set-cookie:id="123";domain="abc.com"
```

```
GET /index.html HTTP/1.0
Host:www.abc.com
Cookie:id="123"
```

## cookie的类型

Cookie的类型，有会话cookie和持久cookie。会话cookie就是关闭浏览器以后就没有了，持久cookie则会在硬盘上保留一段时间 

![](http://dl.iteye.com/upload/attachment/0079/5357/180db7a5-6ea9-36cf-845b-646cd972bbb7.jpg)

比如说这个图，就是一个会保留2个星期的持久cookie 

## cookie与session的关系

cookie里可以放很多信息，但是这样一个问题是不安全；另一个问题是每次http请求，需要发送太多cookie，会增加网络流量。所以一种常见的措施是，在server开辟一个空间，把信息保存在server端，cookie只交换一个ID作为标识。这也就是所谓的session 

session是保存在server的，所以比cookie会安全一点，同时也省一些流量。但是session的问题在于，会增加server端的负担。而且如果是集群的话，不同的server之间怎么做session同步也是一个问题 

如果客户端支持使用cookie的话，session id通过cookie传递就可以了；如果客户端禁用cookie，就需要通过其他方式来传递session id，比如通过url改写的方式 

## cookie规范

cookie规范在rfc2965里，正式名称是HTTP State Management Mechanism（HTTP状态管理机制） 

除了server端要遵循规范的要求，支持cookie以外，客户端（通常是浏览器）也要遵守规范，cookie才能正常运作。如果浏览器不支持rfc2965，根本不能识别set-cookie和cookie首部的话，那这个浏览器也就不支持cookie了

# 认证

HTTP规范中有单独的一块，叫做“认证”。目的是对服务器的资源进行保护，只允许合法用户访问。一般我们自己开发的系统，也都会有认证机制，用户输入用户名和密码之后，才允许访问系统。但是这种认证是应用层面的认证，和HTTP的认证不是一回事，本文说的是HTTP协议层面的认证，不是WEB应用层面的认证 

比如说，访问淘宝，需要输入用户名和密码，这是应用层面的认证；请求某个服务器通过http提供一份文档，server返回401，要求客户端提供用户名和密码，这是HTTP协议层面的认证 

认证分为2种，一种是“基本认证”，另一种是“摘要认证” 

## 基本认证 

基本认证是非常简单的，举一个例子

```
GET /family/jeff.jpg HTTP/1.0
```

```
HTTP/1.0 401 Authorization Required
WWW-Authenticate:Basic realm="Family"
```

```
GET /family/jeff.jpg HTTP/1.0
Authorization:Basic fjsfieqe4jkvkldfe
```

```
HTTP/1.0 200 OK
......
```

流程非常简单，client按照“用户名:密码”的格式得到字符串，然后对此字符串进行BASE64编码，然后发到server 

基本认证的安全性很低，只能说是聊胜于无，有以下几个主要问题： 

1、在client和server中传输了密码，所以很容易被截获，比如经过proxy，这个proxy很容易就可以得到密码；或者用户不小心在浏览器上装了恶意插件，密码也就泄露了 

2、密码等于是明文传输的，BASE64只是一种编码，不是加密，很容易就可以逆向算出密码 

3、得到Authorization首部后，可以很容易地重放 

## 摘要认证 

摘要认证比基本认证在安全性上要强一些，不过也有缺陷，也举个例子进行说明：

```
GET /family/jeff.jpg HTTP/1.0
```

```
HTTP/1.0 401 Unauthorized
WWW-Authenticate:Digest realm="Family" nonce="nonce123"
```

```
GET /family/jeff.jpg HTTP/1.0
Authorization:Digest username="bri" realm="Family" nonce="nonce123" response="MD5123456"
```

```
HTTP/1.0 200 OK
......
```

这里有所简化（没有包括预授权、保护质量协商等内容），但是已经可以说明摘要认证与基本认证的区别 

这里传输的MD5123456，不是完整的密码，而是通过摘要算法（MD5）计算出的密码摘要。所以在网络上传输的，只是密码的摘要，而不是完整的密码，这样就避免了完整的密码被泄露 

其次，MD5算法与BASE64不同，仅通过摘要字符串，很难逆向计算出原始字符串 

并且，这里server发送Authorization首部时，还发送了一个nonce作为随机数，nonce要参与密码摘要的计算。由于每次的随机数都是不同的，所以计算得到的摘要结果也是不同的。这就避免了恶意用户截取到摘要结果后，发起重放攻击 

这里要说明的是，认证（基本认证、摘要认证）是HTTP规范的一部分，具体实现依赖于server和client 

比如说，有一些早期的浏览器，只实现了基本认证，而没有实现摘要认证，这是完全可能的。那么这种浏览器，就无法访问要求摘要认证的http server了 

http认证，我认为只能是网络安全的一个补充，保护一些不太敏感的文档，还是有点作用的。因为你可以不需要在应用层面额外做编码工作，将认证的工作交给服务器，来节省一点工作量 

但是现在的很多WEB应用，用HTTP认证的方式来保护是不可行的。比如说很多的B2C应用，如果用http认证，那很多只是想看看，懒得去注册的用户（比如我），肯定就会把网页关掉了 

## 密码传输的问题 

我多年前做的一个内部OA系统，虽然用MD5算法，将密码摘要以后存在数据库里，但是这样做，仅仅是保证了，“如果database server被攻破”，用户的密码不会被泄露而已，如果攻击是发生在internet传输过程之中，还是防不胜防的 

其实现在非常多的网站依然有这个问题，包括很大的知名B2C网站。就拿当当网举个例子： 

![](http://dl.iteye.com/upload/attachment/0079/5372/fa4e26e8-9564-3619-91d0-c86e786abad2.jpg)

这就是html里的一个普通的form，所以当用户填完用户名和密码，点击“登陆”按钮之后，浏览器就会发起一个post请求，把表单的内容放在http request的entity-body里发出去。

下面是我用chrome的开发者工具截取到的原始http request： 

```
__VIEWSTATE:/wEPDwULLTE4ODM3NDAxMTFkZA== 
txtUsername:xxx@xxx.com（这是我在当当的注册邮箱） 
txtPassword:xxxxxx（这是我在当当的注册密码） 
login_type:0 
A12B56CD78EF90G:ATkRHcR2iJwy2e1s13/1kg== 
wdvalidatetoken:kW3+YmOUtaU= 
```

由于现在大部分web应用的UI，还是基于HTML的，所以很难避免这个问题 

一种办法，是在客户端通过javascript做一些处理。另一种办法，就是WEB应用自行开发浏览器安全输入的插件，比如说支付宝就是这么做的： 

![](http://dl.iteye.com/upload/attachment/0079/5378/7edc687f-884f-3166-a0a6-19bf48709f53.jpg)

可以看到，上图并不是一个html表单，而是一个私有的控件，那么用户名和密码就可以在其中加密后传输了，到了支付宝的server再解密，避免了密码明文传输

# HTTPS

## HTTPS模型 

![](http://dl.iteye.com/upload/attachment/0079/6189/a62bfcc7-b833-3563-80f9-ea6ccc061b19.jpg)

和HTTP的区别，就在于HTTP和TCP之间多了一层SSL或者叫TLS层。这样一来，HTTP报文就不是明文发送到TCP层了，而是经过安全层做了加密，从而保证了HTTP的安全 

像细节的安全握手，密钥交换，编码解码，都是在安全层完成的。所以HTTPS和HTTP的区别，也就在这一层 

## 加密、解密、密钥 

看完这一章，我总算知道什么是密钥了 

原来的报文叫明文，经过编码以后，就得到了密文。然后密文要还原成明文，这个过程就叫做解码 

在HTTPS里，编码也就是“加密”的过程；解码就是“解密”的过程 

在另外一些场合，也有编码和解码的过程。比如说字符串存储成字节，也是编码（encode）；字节还原成字符串，也叫解码（decode）。但是这个是公开的，所以不能叫“加密”、“解密”。类似的，HTTP的基本认证，也会把“username:password”用BASE64编码后发送，这个也不能算加密 

“加密”和“解密”都需要算法。比如ROT3加密算法，就很简单 

这种算法是没有密钥的，也就是说，一段明文，用这个算法加密以后，得到的密文是一定的 

另一种高端的算法，是有密钥的，根据不同的密钥，加密的结果也不一样。比如ROT-N加密算法：对于N=3密钥，明文ABC会变成DEF；对于N=5密钥，明文ABC会变成FGH 

对于这种高端加密算法，算法是公开的，而且还开放源代码让人学习；私密保存的是密钥而已 

## 对称密钥、非对称密钥 

本来早期的加密算法，大多数是对称密钥算法。也就是说，加密用的密钥，和解密用的密钥是一样的 

这就有2个问题： 

1、client和server要通过HTTPS通信，就必须要交换这个密钥才行，密钥在网络上传播，本身就不安全了，所以就无法保证后续通信的安全性 

2、如果server和100个client通信，就得有100个密钥；如果100台机器，互相通信，那大概需要10000个密钥。实际上internet上的通信规模，远不止这个数量级。所以密钥的管理是一个大麻烦 

后来就进化出了非对称密钥算法。加密（编码）用的密钥是一个，解密（解码）用的密钥是另一个。加密用的密钥称为“公钥”，所有客户端都可以通过某种方式得到这个公钥；解密用的密钥称为“私钥”，只有服务端有这个私钥 

这样，私钥就不在网络上传输了，只有服务端才有。安全性比对称密钥算法要高了很多。另外，也解决了密钥太多难管理的问题，现在不管有多少client，都只有一对公钥/私钥 

## 数字签名、数字证书 

这就没什么好说的了，都是上面讲的加密和解密的原理的应用 

利用密钥，对报文的摘要来编码，解码，证明client和server的身份、保护报文内容等等
