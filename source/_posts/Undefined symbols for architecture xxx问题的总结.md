title: Undefined symbols for architecture xxx问题的总结
date: 2014-10-15 21:51
categories: iOS 
---
这2天升级到xcode6，用ios8 SDK编译老项目，各种Undefined symbols for architecture xxx，精神差点崩溃了。不过最后还是解决了，本文简单总结一下
<!--more-->

简单来说，Undefined symbols基本上等于JAVA的ClassNotFoundException，最常见的原因有这几种：

# build的时候没有加framework

比如说，有一段代码我用了OpenGL，引入了头文件

```
#import <OpenGLES/ES2/glext.h>
```
build的时候，compile阶段没有问题，但是link就报错Undefined symbols for architecture xxx（这里xxx可能是armv7s，armv7或者arm64，取决于配置，稍后会说）。解决方法是在Build Phases的Link Binary With Libraries里加入OpenGLES.framework，再编译就ok了

# 工程依赖的库，编译时Architectures不匹配

在Build Settings里，第一项Architectures，是配置项目的编译体系结构，主要有下面3个配置项：

Architectures：将要创建的Bundle支持的ARCH

Valid Architectures：有效的ARCH，这个配置项没什么用，一般配置成armv7，armv7s，arm64就行了

Build Active Architecture Only：是否只打当前连接设备对应的arch

在真机上，常见的ARCH有3个：armv7，armv7s，arm64

armv7：对应iPhone4和iPhone4S

armv7s：对应iPhone5和iPhone5C，还有早期的iPad

arm64/armv8：对应iPhone5S和iPhone6系列，以及比较新的iPad，如iPad mini2，iPad Air

而ARCH是向下兼容的，比如用armv7打出来的包，可以运行在arm64架构的设备上；反之不行。所以如果应用要支持iPhone4系列，Architectures就一定要包含armv7才行

而Build Active Architecture Only是指是否仅当前连接的设备的架构打包。比如Architectures配置了armv7和arm64，Build Active Architecture Only设置为YES，那么连接iPhone4的时候，就会以armv7打包；连接iPhone5S的时候，就会以arm64打包。如果Build Active Architecture Only设置为NO，那么就会2种架构都打，在运行期根据实际的设备架构来执行。所以最后打出来的Bundle体积会比较大

说了这么多，这个为什么造成Undefined symbols呢？因为还有另外一条规则，就是build link阶段，用arm64生成的.o文件，无法link用armv7s或者armv7生成的.o文件，所以就会link error

常见的情况是，项目引用了一个第三方库（比如从pod来的库），而这个第三方库打包的时候只支持armv7s和armv7，而项目有使用arm64打包，这个时候就会由于无法link，而报错Undefined symbols

解决的办法是，或者重新打包第三方库，加入arm64；或者自己的项目去掉arm64

# 有时候在模拟器上无法构建，在真机上可以

这种情况我只遇到过一次。我们的app可以连接一个外厂商的蓝牙打印机，对方提供了一个lib。当我们的项目引入了这个lib之后，就无法在模拟器上build通过了，但是在真机上是没问题的