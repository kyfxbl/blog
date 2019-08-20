title: seajs项目构建的总结
date: 2013-10-30 21:33
categories: javascript
---
最近连续折腾了将近一个星期的seajs构建。。踩了不少坑，总结了一些经验，在这里贴一下
<!--more-->

# require()参数的二义性

用seajs做模块化开发，经常用到这个函数：

```
require("module/path/file");
```
或者```
require("./file");
```
前者是顶级标识，后者是相对标识，总之都是传递给require()的参数，表示要加载别的模块

这个参数有2个意思，首先是代表路径，根据这个路径要能够找到模块所在的文件，否则直接返回了404，什么也别谈了。其次，还代表模块的ID，如果根据路径找到了模块文件，但是模块的ID（也就是define()的第一个参数）不匹配，模块也无法加载成功，这称为路径与ID的一致性

总之就是2个约束：路径要能访问到文件，而且路径和模块ID要一致。这并不是CommonJS规范的要求，这只是seajs自己的规定

# 前端模块化的特殊性

这个规定看起来有点怪，比如node.js里就没有这个奇怪的约束，只要能根据路径找到相应的文件就行了。但是这是前端开发的特殊性决定的

server端的js，没有合并的需求。但是在HTTP2之前，前端的js在发布时一般都需要合并，以减少http请求数。所以开发的时候，可能根据业务逻辑，把模块划分得很清楚：

user.js

employee.js

......

但是在部署的时候，会合并压缩成只有一个文件：total.js

所以这就带来一个问题，require()的参数要如何写？

在开发的时候：

```
require("user");// 可以找到user.js
require("total");// 此时还不存在total.js，因为还没有合并
```
而在部署的时候正好相反：

```
require("user");// 此时已经不存在
require("total");// 可以找到total.js
```
问题在于，总不能每次上线之前还把代码改一遍吧。所以就涉及到合并文件的module_id要设置成什么，合并后的文件名是什么，合并后要如何压缩，require()参数自动修改等，所以就需要配套的构建工具，来自动化处理这些由合并衍生的问题

不过这里有一个建议：每个模块都只对外暴露唯一的接口，并且就将这个接口的文件名，作为合并后的文件名。这是一个取巧的方法，可以解决合并前后文件名不一致的问题

```
define(function(require, exports){
    exports.add = function(){
        // code here...
    };
});
```

```
// package.js
define(function(require, exports){
    exports.a = require("./a");
});
```

```
// outter
define(function(require){
    var a = require("package").a;
});
```

上面的示例代码，前面2个文件都是某个模块内部的，package是唯一的出口。第3个文件需要依赖这个模块，只通过package.js来依赖，不直接依赖模块内部的a.js。然后合并以后的文件名，也叫做package.js，正好就一样了，不会发生由于文件名改变而找不到文件的问题```
require("package");
```
这行代码，无论在开发态还是部署态都是OK的

# 两种构建的方式

理论上可以自己构建seajs项目，但是实际上没有人这么做，因为太复杂。seajs的作者也强烈建议用户使用官方配套的构建工具来构建项目

构建工具有2种，分别是spm-build和grunt-cmd-xxx

## spm-build

spm-build是spm的附属项目。spm是seajs的包管理工具，可以理解成前端的npm。spm不在本文的讨论范围内，就不多说了

spm-build要求模块目录下有package.json文件，其中用<spm>属性声明要如何构建，然后执行spm-build就一条龙构建完了，包括文件的uglify。方便倒挺方便，不过可配置性比较弱，而且如果用了angular的话，spm-build在uglify的时候，会把不该替换的变量名也替换了，还是需要人工干预一下

总的来说，如果想省事的话，用spm-build也就可以了。但是如果项目有一些特殊的需求，spm-build可能就满足不了。另外spm-build不再是官方推荐的构建方式，后续的发展还不清楚

## grunt-cmd-xxx

其实只有2个grunt插件，分别是grunt-cmd-transport和grunt-cmd-concat，分别完成提取依赖和合并的工作，后续的uglify还是用grunt-contrib-uglify来做的

这种构建方法其实就把构建的流程打散，只提供2个Task插件，用户自己控制构建过程，所以比较灵活。这也是官方推荐的构建方式。不过说实话，还比较复杂。构建过程可以看我的另外几篇帖子

下面是一个关键的：截至到目前，grunt-cmd-transport有一个BUG，提取后的js，会把间接依赖也写入deps数组，所以浏览器会发起很多无效的http请求。这个BUG目前还没有解决，但是用spm-build不会遇到这个问题。所以一定要慎重，详情在：[grunt-cmd-transport的BUG](https://github.com/spmjs/grunt-cmd-transport/issues/56)