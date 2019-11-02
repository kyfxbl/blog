title: ios background task
date: 2014-02-15 15:49
categories: iOS 
---
今天要实现一个需求，当用户触摸HOME键，将应用切换到后台时，启动自动备份的任务。这涉及到ios的后台任务处理，本文简单总结一下
<!--more-->

首先，ios app有5种状态，分别是：not running, inactive, active, background, suspended，详情请看官方的guide：

[apple guide](https://developer.apple.com/library/ios/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/ManagingYourApplicationsFlow/ManagingYourApplicationsFlow.html#//apple_ref/doc/uid/TP40007072-CH4-SW3)

# 机制

如果应用处于background状态，又希望它继续做一些事的话，ios提供了几种途径：

## 推送

苹果提供的PUSH机制，叫APNS。腾讯的QQ和微信就是使用这种方式。实际上，使用长连接会更好，但是苹果不支持。我理解其实应用已经suspended，但是当接收到push的数据以后，会短暂地回到background进行处理，处理完毕以后又回到suspended状态

从ios7开始，分为local push和remote push，我们应用现在还没用到，暂不深究

## 智能调度

太玄乎了，不太了解

## 特定的多任务

某些特定的任务可以在后台长时间运行，比如VOIP，location service等，只有特定类型的任务，才能用这种方式，适用性不强

## 后台上传下载

类似于特定多任务，只有特殊的任务才能用

## task completion

这种方式的适用性比较强，我最后也是采取这种方式来实现的。因此本文重点介绍这种

通常情况下，应用在进入background之后，很快就会转到suspended状态。但是，如果应用有需要的话（比如我们这个需求），可以向系统申请一点额外的时间来完成当前的任务

代码：

```
- (void)applicationDidEnterBackground:(UIApplication *)application
{
    __block UIBackgroundTaskIdentifier bgTask;// 后台任务标识

    // 结束后台任务
    void (^endBackgroundTask)() = ^(){
        [application endBackgroundTask:bgTask];
        bgTask = UIBackgroundTaskInvalid;
    };

    bgTask = [application beginBackgroundTaskWithExpirationHandler:^{
        endBackgroundTask();
    }];

    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{

        double start_time = application.backgroundTimeRemaining;// 记录后台任务开始时间

        BOOL networkAvailable = [YLSGlobalUtils isNetworkAvailable];
        if(!networkAvailable){
            NSLog(@"网络不可用，取消自动备份");
            endBackgroundTask();
            return;
        }

        BOOL need = [backupService checkNeedBackup];
        if(!need){
            NSLog(@"无需备份");
            endBackgroundTask();
            return;
        }

        [backupService doBackupProcessHandler:^(float done, float total){
            // nothing to do with progress
        } CompletionHandler:^(NSError* error, NSArray* statistics){
            double done_time = application.backgroundTimeRemaining;
            double spent_time = start_time - done_time;
            NSLog(@"后台备份完成，耗时: %f秒", spent_time);
            endBackgroundTask();
        }];
    });
}
```
核心是这个方法：```
- (UIBackgroundTaskIdentifier)beginBackgroundTaskWithExpirationHandler:(void(^)(void))handler
```
注册一个后台任务，这个任务最多只有10分钟时间，如果超时，则会调用参数中的block，在此block中，必须调用这个方法：```
- (void)endBackgroundTask:(UIBackgroundTaskIdentifier)identifier
```
否则，应用会crash。只要调用了beginBackgroundTaskWithExpirationHandler:方法，就必须在handler里相应地调用endBackgroundTask:方法

实际的逻辑，在下面的block里完成，这里没什么特别的，只是如果提前结束了任务，也调用一次endBackgroundTask:方法，这样就不会超时，前面的expirationHandler就不会被执行

另外，通过UIApplication的backgroundTimeRemaining属性，可以获取此后台任务还剩余的时间（当此值变成0，expirationHandler就被执行）

# 使用时机

这段代码，是写在ApplicationDelegate的生命周期方法里：

```
- (void)applicationDidEnterBackground:(UIApplication *)application
```

这是因为，我们希望仅当应用切换到后台时才自动备份。但是后台任务并不是只能在这种情况下启动，如果应用中有一些关键性的任务，希望即使被切换到后台也要先完成再suspend，就可以随时调用

```
- (UIBackgroundTaskIdentifier)beginBackgroundTaskWithExpirationHandler:(void(^)(void))handler
```

以确保此任务完成，写法只要参考上面的代码就行了