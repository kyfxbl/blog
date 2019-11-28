title: asyncjs笔记
date: 2014-01-26 21:37
categories: javascript 
---
![asyncjs](http://pic.kyfxbl.com/asyncjs.png)
async在node和浏览器里都可以用，API基本没区别
<!--more-->

node环境：
```
var async = require("async");
async.xxx();
```

浏览器环境：
```
<script type="text/javascript" src="async.js"></script>
<script type="text/javascript">

    async.xxx();

</script>
```

本文总结下各API调用的方法。本文的示例代码在：[AsyncExample](https://github.com/kyfxbl/AsyncExample)

# 集合类

集合类的API

## each

Array的迭代器方法，很常用

```
async.each(["abc", "def", "ghi"], function(item, callback){

    console.log(item);
    callback(null);// must call once

}, function(err){

    if(err){
        console.error("error");
    }
});
```

## eachSeries

和each基本一样，但是顺序执行，API接口和参数都一样

## eachLimit

也和each差不多，多出来的limit参数，是限制允许并发执行的任务数

```
async.eachLimit(["123", "456", "789"], 2, function(item, callback){

    console.log(item);
    callback();// 必须调用，才能触发下一个任务执行

}, function(error){

    if(error){
        console.error("error: " + error);
    }

});
```

## map

将一个Array中的元素，按照一定的规则转换，得到一个新的数组（元素个数不变），也比较常用

```
async.map([1,3,5], function(item, callback){

    var transformed = item + 1;
    callback(null, transformed);

}, function(err, results){

    if(err){
        console.error("error: " + err);
        return;
    }

    console.log(results);// [2, 4, 6]

});
```

## mapSeries和mapLimit

基本差不多，就不多介绍了

## filter

用于过滤Array中的元素

```
async.filter([1, 5, 3, 7, 2], function(item, callback){

    callback(item > 3);

}, function(results){

    console.log(results);// [5, 7]

});
```

## filterSeries

类似

## reject和rejectSeries

和filter正好相反，filter是保留true的item，而reject是删除true的item

```
async.reject([4, 7, 88, 11, 36], function(item, callback){

    callback(item > 11);

}, function(results){

    console.log(results);// [4, 7, 11]

});
```

## reduce和reduceRight

将一个数组中的元素，归并成一个元素，看看就好了，用得不是很多，可以需要的时候再查

```
async.reduce([3, 2, 1], 0, function(memo, item, callback){

    callback(null, memo + item)

}, function(err, result){

    if(err){
        console.error("error: " + err);
        return;
    }

    console.log(result);// 6
});
```

reduceRight差不多，只是Array中元素迭代的顺序是相反的：

```
async.reduceRight(["!", "ld", "wor"], "hello ", function(memo, item, callback){

    callback(null, memo + item)

}, function(err, result){

    if(err){
        console.error("error: " + err);
        return;
    }

    console.log(result);// hello world!
});
```

## detect和detectSeries

从数组中找出符合条件的元素

这个API很像filter，参数也都一样，但是只会返回一个结果

```
async.detect([13, 44, 23], function(item, callback){

    callback(item > 37);

}, function(result){

    console.log(result);// 44

});
```

## sortBy

数组元素排序

```
var person1 = {"name": "aaa", age:79};
var person2 = {"name": "bbb", age:23};
var person3 = {"name": "ccc", age:54};

async.sortBy([person1, person2, person3], function(person, callback){

    callback(null, person.age);

}, function(err, sorted){

    console.log(sorted);
});
```

## some

同名函数any。在数组中找至少一个元素，类似于filter和detect。区别在于，filter和detect是返回找到的元素，而some是返回bool

```
async.some([1, 5, 9], function(item, callback){

    callback(item > 10);

}, function(result){

    console.log(result);// false

});
```

## every

同名函数all。跟some相反，如果数组中所有元素都满足条件，则返回true，否则返回false

```
async.every([1, 21, 23], function(item, callback){

    callback(item > 10);

}, function(result){

    console.log(result);// false

});
```

## concat

对数组中的元素进行迭代操作，形成一个新数组

```
async.concat([1, 2, 3], function(item, callback){

    callback(null, [item+1, item+2]);

}, function(err, results){

    console.log(results);// [2, 3, 3, 4, 4, 5];

});
```

# 流程控制类

流程控制类的API

## series

一组函数顺序执行，这个API很常用。当一个步骤完成时，调用callback()，不传递参数；如果其中一个步骤出错，则调用callback(err)，后续的步骤就不会继续执行。每个步骤执行的结果，会汇总到最终callback的results参数中

```
async.series([function(callback){

    console.log("first task");
    callback(null, 1);

}, function(callback){

    console.log("second task");
    callback({message:"some error"}, 2);// callback with a error parameter

}, function(callback){

    console.log("third task");// not called
    callback(null, 3);

}],function(error, results){

    if(error){
        console.error("error happend: " + error);
    }
    console.log(results);// [1, 2]
});
```

## parallel

基本上和series一样，包括API的参数，区别在于，series是顺序执行，而parallel是同时执行

```
async.parallel([function(callback){

    callback(null, 1);

}, function(callback){

    callback({message:"error"}, null);

}, function(callback){

    callback(null, 3);

}], function(error, results){

    if(error){
        console.error(error.message);
    }

    console.log(results);

});
```

## whilst和until

重复执行一个函数，直到test function的值为false或true。类似的还有doWhilst和doUntil

```
var count = 0;

async.whilst(
    function () { return count < 5; },
    function (callback) {
        console.log("call once");
        count++;
        setTimeout(callback, 1000);
    },
    function (err) {
        if(err){
            console.log("error: " + err);
        }
        console.log("whilst done");
    }
);
```
```
var count3 = 0;

async.until(function(){

    return (count3 > 5);

}, function(callback){

    count3 ++;
    setTimeout(callback, 1000);

}, function(error){

    console.log("until done");
    count3 = 0;

});
```

## forever

循环执行一个函数，直到抛出错误

```
async.forever(function(callback){

    setTimeout(function(){
        console.log("forever...");
        console.log(flag);
        flag ++;
        if(flag > 5){
            callback({message:"hehe"});
        }else{
            callback();
        }
    }, 1000);

}, function(err){

    // once this callback be called, the "forever" stops
    if(err){
        console.error("there is an error");
    }

});
```

## waterfall

这个可能是async库中最重要的一个方法，可以解决callback嵌套的问题。上一个流程的执行结果，会传给下一个流程的参数。如果其中一个流程出错，则会中止后续流程的执行，直接调用最终的callback。否则最后一个流程的结果，会传递给最终callback

```
async.waterfall([function(callback){

    callback(null, "kyfxbl", 29);

}, function(name, age, callback){

    console.log(name);// kyfxbl
    console.log(age);// 29
    callback(null, 10000, 200);

}, function(salary, bonus, callback){

    console.log(salary);// 10000
    console.log(bonus);// 200
    callback(null, [1, 2, 3]);

}], function(err, results){

    console.log(results);// [1, 2, 3]

});
```

## compose

可以将几个function组合成function，这个方法不是很常用

```
// f(g(h(n)))
async.compose(function(n, callback){
    setTimeout(function(){
        callback(null, n + 1);
    }, 10);
}, function(n, callback){
    setTimeout(function(){
        callback(null, n * 3);
    },10);
})(4, function(err, result){
    console.log("compose result: " + result);// 13
});
```

## queue

创建一个执行任务的队列，类似oc中的NSOperationQueue，好像用得不多

```
var q = async.queue(function(task, callback){

    console.log("hello " + task.name);
    callback(null);

}, 3);

q.push({name:"kyfxbl"}, function(err){
    console.log("done");
});

q.push({name:"liting"}, function(err){
    console.log("finish");
})
```

## nextTick

node里其实已经有nextTick方法，这个方法是为了统一node和浏览器环境的行为，在浏览器环境，实际调用的是setTimeout(func, 0)函数

```
var order = [];

async.nextTick(function(){
    order.push(222);
});

order.push(111);

process.nextTick(function(){
    console.log(order);// [111, 222]
});
```

## times

重复执行函数n次，并收集最终结果

```
async.times(5, function(n, next){

    console.log("n: " + n);
    next(null, n * 2);

}, function(err, results){

    console.log(results);// [0, 2, 4, 6, 8]
});
```