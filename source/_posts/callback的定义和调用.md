title: callback的定义和调用
date: 2013-10-04 10:27
categories: javascript   
---
回调函数定义以后，是谁调用的呢？参数是如何确定的呢？
<!--more-->

通常，方法是开发者自己定义的，调用也是开发者自己控制的。回调函数的情况则不太一样，回调函数是开发者定义的，而调用则通常是由框架完成的。这里指的就是node，比如以下这段代码

```
chatServer.on("connection",function(client){
    client.write("hello!\n");
    client.write("bye!\n");
    client.end();
});
```

第二个参数是一个匿名的callback function，函数体是开发者定义的，但是调用则是由node控制的。当connection事件发生时，node会调用这个匿名回调函数，并传递client作为参数。当然，也可以主动调用emit()方法来触发这个回调函数

```
chatServer.emit("connection",{
    write:function(){
        console.log("callback called")
    },
    end:function(){
        console.log("end")
    }
});
```

也可以触发callback，并且第2个参数会被传过去作为client——但是很明显，除了测试之外，这样做并没有太大的意义

所以这里很有趣，虽然callback是开发者自己定义的，但是调用的时机（event type），以及参数的个数和意义，却是node事先定义的。或者可以这么说：node定义了callback function的签名，而开发者负责定义callback body。这里有点像JAVA的SPI。另一方面，如果将callback绑定到一个不存在的事件上（比如wakeup），那么意义也不大，因为运行时node永远不会去调用它。这类callback一般是为了特殊的目的定义的

所以关键还是node预定义了哪些事件，以及调用时会传递哪些参数。除了参考API手册，似乎也没有更好的办法