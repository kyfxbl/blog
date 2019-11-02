title: express处理多域名环境下的session
date: 2014-10-12 15:06
categories: javascript  
---
微站子系统上线以后，我们有2个域名，分别是www和wx。测试发现，当页面交替请求这2个域名下的资源和服务时，会造成session反复切换，于是依赖session的一些方案都失效了
<!--more-->

最后定位到，是因为我们错误地使用express session造成的

express的session中间件的原理是，对于配置了使用session middleware的path，大致上有以下的流程：

1、看request是否携带了sid的cookie

2、如果没有sid，则在服务端（内存，redis，mongodb）创建一个session，并分配一个sid在响应中给客户端，下次客户端就会带着这个sid cookie上来

3、如果有sid，但是在服务端找不到对应的session，那么也创建一个，并重新分配sid

4、如果有sid，并且能找到对应的session，则把session中储存的值取出来，放在req.session对象上

因此，在多域名的环境下，要保证session机制正确，需要注意以下几点：

1、不同的url配置的mongodb源必须一致，既要是同一个mongodb，还要是同一个database

2、需要配置domain属性

参考配置如下：

```
.use("/svc", express.session({   
                store: new MongoStore({
                    url: 'mongodb://' + global['_g_topo'].dataServer.mongodb.connectionurl + '/yilos_session'
                }),
                cookie: {
                    domain: "yilos.com",
                    maxAge: maxAge
                },
                secret: secret
            }))
```
如果使用的mongodb源不同，那么当URL切换的时候，根据sid就找不到对应的session，于是会重新创建；如果没有配置domain，那么在www和wx域名切换的时候，客户端会携带不同的cookie，也会造成express下发不同的sid，还是找不到