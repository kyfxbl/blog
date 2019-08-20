title: httpd的url匹配
date: 2013-09-24 11:24
categories: web
---
用httpd配置URL
<!--more-->

官方reference：[http://httpd.apache.org/docs/2.4/en/urlmapping.html](http://httpd.apache.org/docs/2.4/en/urlmapping.html) 

httpd主要是一个静态文件服务器。当然不限于此，通过各种mod，httpd也可以作为一个前端服务器，把请求转发到servlet container、cgi等。不过主要还是静态文件服务器，所以官方的reference也是从处理静态文件说起 

# DocumentRoot 

首先要把浏览器里的url，映射到server的文件系统上 这是通过DocumentRoot directive配置的（httpd的各种配置，都是用directive完成的，各种mod提供了不同的directive）

```
DocumentRoot "/usr/local/httpd/htdocs"
```

这里把url的"/"，映射到了/usr/local/httpd/htdocs目录下，比如： http://localhost/abc，会映射到/usr/local/httpd/htdocs/abc这个文件 http://localhost/web/，会映射到/usr/local/httpd/htdocs/web/这个文件夹下（区别在于结尾的"/"） 

# DirectoryIndex 

另外有一个directive叫做DirectoryIndex，是用来自动处理文件夹的

```
<IfModule dir_module>
    DirectoryIndex index.html
</IfModule>
```

这样配置之后，如果url请求访问一个目录，则httpd会先到这个目录下寻找index.html，找到则返回;如果没有找到，则显示目录列表 这也是为什么，访问http://localhost，会显示success page 

![](http://dl2.iteye.com/upload/attachment/0086/6251/e9bbc7b4-9a4c-39c3-b6c7-f00a853213a2.png)
![](http://dl2.iteye.com/upload/attachment/0086/6253/4e11656e-2f84-37e7-8cea-85f66111933e.png)

# 文件和文件夹 

这里有一个问题，http://localhost/servlet，httpd怎么知道请求的是servlet文件，还是servlet目录呢？ 

一般来说，如果结尾有"/"，就认为请求的是目录;否则认为是文件。但是httpd做了一些透明处理，可能会造成误导 

http://localhost/servlet/，这种情况比较简单。如果在DocumentRoot下存在servlet目录，则匹配成功;找不到则直接返回404 
http://localhost/servlet，稍微有点不一样： 如果在DocumentRoot下存在文件，则返回; 如果没有文件，但是有servlet文件夹，则会透明地当作文件夹处理;如果都没有，才会返回404 

# 映射到其他文件夹 

也可以通过Alias和AliasMatch，配置特定的URL不在DocumentRoot下，而是到别的地方查找。不过实践发现会返回403，不知道是为什么，网上也有很多人碰到这个问题。有空再研究下，今天先跳过了