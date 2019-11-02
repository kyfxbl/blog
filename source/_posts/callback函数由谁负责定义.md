title: callback函数由谁负责定义
date: 2013-11-20 15:26
categories: javascript 
---
这段时间从java切换到js，最一头雾水的就是callback，今天稍微有点感觉，总结一下
<!--more-->

以前在java里，方法就是由所在的类定义的，非常明确，比如：

```
public String transfer(String name){
    return "hello" + name;
}
```
这个方法的定义就集中在所在的类里，如果使用一个第三方的框架或类库，也只要看一下API文档或者源码就行了

但是js里的callback函数，感觉是在2个地方分别定义的，比如：

```
function acceptCallback(callback){
    // 一些逻辑
    callback(123, "helloworld", enterpriseId);
}
```
上面是函数接受另一个函数作为参数，并负责传参并调用它

```
acceptCallback(function(num, str, enterpriseId){
    // 处理num
    // 处理string
    // 处理enterpriseId
});
```

上面这段代码实际调用acceptCallback函数，并传了一个回调函数给accpetCallback作为参数，回调函数体是在这里定义的，另一个函数：

```
acceptCallback(function(num, str, enterpriseId){
    num++;
    console.log(str);
    mongo.remove(enterpriseId);
})
```
上面的代码也调用了acceptCallback函数，但是传了另一个回调函数给它，对3个参数的处理完全不同

回调函数的签名是由接受它的函数定义的，因为它负责调用回调函数，只有它才知道会传什么参数。但是，回调函数的body是在客户端代码定义的，回调函数的具体逻辑，可以完全不同

因此，回调函数的API也是由第三方组件提供的，客户端代码需要查看API文档，才能知道回调函数的定义，但是可以根据自己的需求，在函数体里写逻辑

比如使用node-mongodb-native：

```
collection.remove({"enterpriseId":enterpriseId},function(err,deleteNum){
                // how to handle error, and do something with deleteNum
            });
```
上面的remove()方法，就是mongodb-native的API，其中规定了回调函数的签名，第一个参数是error，第二个参数是删除的记录数。但是具体在回调函数里写什么逻辑，则是由客户端代码确定的

remove()方法内部类似：

```
function remove(query, callback){
    // 执行remove操作
    if(success){
        callback(null, removeNum);// no error
    }else{
        callback(err);
    }
}
```

综上所述，java的API看一下方法定义就知道了：

```
public String getName(String name){}
```
参数类型，返回值，都一目了然。对比javascript的函数定义：

```
function remove(query, callback){
}
```

不直观的原因，主要就是一眼看到这个函数定义，完全不知道callback是什么（要接受什么参数），所以调用的时候也就不知道要怎么写

```
remove(query, function(){
    // 这里面要写什么？
});
```

所以，只能查看API文档，或者读remove函数的源码