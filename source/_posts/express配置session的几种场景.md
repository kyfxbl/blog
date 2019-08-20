title: express配置session的几种场景
date: 2014-11-22 00:28
categories: javascript
---
多个node实例同时运行的情况下，session的处理比单实例的场景更复杂，以2个node节点为例
<!--more-->

# 单一域名的场景

只有一个域名的场景比较简单，只要把session的store和secret配置成一样，就可以实现session共享

```
.use("/svc", express.session({   
                store: new MongoStore({
                    url: 'mongodb://127.0.0.1:2222/my_session'
                }),
                cookie: {
                    maxAge: 60 * 60 * 1000
                },
                secret: "abcdefg"
            }))
```
因为2个node在同一个域名下，所以浏览器每次请求会携带相同的cookie，而2个node的session也存储在同一个地方，因此根据此cookie就能找到同一个session，实现共享

但是，如果把数据源配置得不同，情况就完全不同，比如一个配置成mySession1，一个配置成mySession2（配置为不同的mongodb也同理）：

1、首先请求了第一个node节点，node生成一个session，然后发回一个Set-Cookie

2、浏览器携带此cookie请求node节点2，node2根据此cookie找不到session，于是重新生成session，又发回新的cookie

3、浏览器携带新的cookie请求node1，node1又找不到了，再次生成新session，返回新cookie

4、重复上述步骤……

结果就是，2个node节点不但无法共享session，甚至连自己的session机制都无法生效，因为另一个node节点的cookie，会对自己造成干扰

总结，单域名情况下运行多个node节点，session的源必须一致，否则session机制不可用

# 多域名的场景

node部署在2个不同域名的情况下，结果又有区别

假设node1跑在a.company.com，node2跑在b.company.com，那么在默认情况下，浏览器会发送不同的cookie，因为被看做是2个不同的站点

这时候session的存储位置不管是否配置成相同，结果都是一样的：互不干扰，也不能共享。部分场景下，这种行为是可接受的，但是如果2个站点需要共享session，就需要想办法令浏览器请求这2个node节点时，携带相同的cookie，方法是配置domain参数

```
.use("/svc", express.session({   
                store: new MongoStore({
                    url: 'mongodb://127.0.0.1:2222/my_session'
                }),
                cookie: {
                    maxAge: 60 * 60 * 1000,
                    domain: "company.com"
                },
                secret: "abcdefg"
            }))
```
增加了domain参数之后，a.company和b.company会被浏览器视为同一个域名（原理是在cookie中设置了domain字段），于是浏览器就会始终发送同一个cookie。这种场景和单域名场景是一样的，源配置成一样则session共享，否则session失效

总结，多域名情况下，如果不配置domain，则session互不干扰；如果配置domain为同一个域名，则相当于单域名场景