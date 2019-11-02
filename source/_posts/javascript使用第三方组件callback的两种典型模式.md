title: javascript使用第三方组件callback的两种典型模式
date: 2013-11-20 17:30
categories: javascript  
---
用javascript开发应用，经常需要使用第三方组件，90%都涉及到callback的调用，总结了2种典型的模式
<!--more-->

# 调用第三方API

这种模式比较常见，比如fs，node-mongo-native等，代码类似：

```
collection.remove({"enterpriseId": enterpriseId}, function (err, removeNum) {
    // 操作err和removeNum                
});
```
此模式是调用第三方组件提供的API，同时提供一个callback函数，该callback函数是由第三方组件调用的，remove的伪代码类似：```
function remove(query, callback){
    // do remove
    if(error){
        callback(error);
    }else{
        callback(null, removeNum);
    }
}
```

# 被第三方API调用

这种模式比较少见，典型的如async是这么用的，代码：

```
async.series([queryEnterprise, deleteEnterprise, deleteUser, deletePayment], function (err, results) {
        if (err) {
            // 处理错误
        } else {
            // 处理最终数据
        }
    });
```
上面的自定义函数如queryEnterprise都是被async框架调用的，在自定义函数中需要根据API的要求，调用callback方法```
function queryEnterprise(callback) {
        // 逻辑处理
        if(error){
            callback(error);
        }else{            
            callback(null, result);   
        }            
    }
```
总的来说，模式就这2种，还是蛮复杂的。具体的参数只能看第三方组件提供的API文档