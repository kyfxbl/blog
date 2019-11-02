title: express路由链和错误处理
date: 2014-08-14 15:58
categories: javascript 
---
从express 4.x开始,路由配置方法
<!--more-->

官方建议如下配置路由链：

[Migrating from 3.x to 4.x](https://github.com/strongloop/express/wiki/Migrating-from-3.x-to-4.x)

```
// 在route之前的middleware
app.use(path, middleware1);
app.use(path, middleware2);
...

// route
app.get(path, function(req, res, next){
    // logic
});

// route之后的middleware
app.use(path, middleware3);
...

// 错误处理，一般都放在最后面
app.use(path, function(err, req, res, next){
    // error handling
})
```
然后在route里，一般这样写：```
app.get(path, function(req, res, next){

    // logic

    if(err){
        next(err);// 跳转到error handler
        return;
    }

    res.send(result);// 返回结果到客户端
});
```
一般的middleware和error handler，基本上差不多，区别在于middleware有3个参数，error handler有4个参数，多了一个error

当next()传参数时，会走进error handler；否则走进下一个middleware或者route