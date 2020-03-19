title: iOS中view和controller设计原则的小结
date: 2014-02-21 13:34
categories: iOS 
---
![uiview](http://pic.kyfxbl.com/uiview.jpg)
针对UIView和UIViewController的设计原则，当前个人的一些思考
<!--more-->

# 是否把UIView作为独立的类

我的想法是，如果该视图只对当前的controller适用，那么就没有必要独立成一个类；反之，如果是一个通用的控件，就有必要独立出去实现复用

不复用的view，直接写在controller的loadView方法里就可以了：

```
-(void) loadView
{
    UIView *rootview = [[UIView alloc] init];

    UIButton *button = [UIButton buttonWithType:UIButtonTypeRoundedRect];
    button.frame = CGRectMake(200, 200, 80, 80);
    [button setTitle:@"click" forState:UIControlStateNormal];
    [button addTarget:self action:@selector(clicked) forControlEvents:UIControlEventTouchUpInside];

    [rootview addSubview:button];

    self.view = rootview;
}
```
如果view需要复用的话：

```
-(void) loadView
{
    CGRect screenSize = [[UIScreen mainScreen] bounds];
    YLSSimplePasswordView *passwordView = [[YLSSimplePasswordView alloc] initWithFrame:screenSize Delegate:self];
    self.view = passwordView;
}
```

# View的布局和行为，应该由Controller控制

要想实现View的复用，必须把view的布局和行为委托给Controller控制

控制view的布局，一般是通过

```
- (id)initWithFrame:(CGRect)frame;
```
controller调用此方法创建view，传CGRect作为参数，确定此view在整个页面中的位置和尺寸。同时，自定义的view定义若干delegate method，由controller实现这些方法

一般来说，view不需要持有controller的引用，只需要持有protocol的引用就可以了

```
id<YLSSimplePasswordViewDelegate> myDelegate;
```

# View内部推荐使用相对布局

view有2个property，frame和bounds，分别用来表示此view在super view中的坐标系，及自身的坐标系：

The frame rectangle, which describes the view’s location and size in its superview’s coordinate system.

The bounds rectangle, which describes the view’s location and size in its own coordinate system.

在实践中发现，frame更好用一些。比如这个view

![](http://pic.kyfxbl.com/zsd3.jpeg)

下方这些button，指定它们的位置时，如果直接设置它们的frame，则需要使用root view的坐标系统（因为其super view是root view），计算起来很不方便

更好的做法，是把它们放在一个容器view中，这时候设置frame，就是相对于容器view的坐标系，计算起来会容易很多：

```
UIView *buttonViews = [[UIView alloc] init];// 作为button的容器
buttonViews.frame = CGRectMake((width-300)/2, 180, 300, 410);// 自身的frame相对于root view计算

// button的frame相对于button容器计算
YLSPasswordButton *button1 = [[YLSPasswordButton alloc] initWithTitle:@"1" Value:@"1" Color:pointColor Origin:CGPointMake(0, 0) Delegate:self];
YLSPasswordButton *button2 = [[YLSPasswordButton alloc] initWithTitle:@"2" Value:@"2" Color:pointColor Origin:CGPointMake(110, 0) Delegate:self];
YLSPasswordButton *button3 = [[YLSPasswordButton alloc] initWithTitle:@"3" Value:@"3" Color:pointColor Origin:CGPointMake(220, 0) Delegate:self];

[buttonViews addSubview:button1];
[buttonViews addSubview:button2];
[buttonViews addSubview:button3];

[self addSubview:buttonViews];
```
本文先简单总结这么多，感觉View的布局和自适应是最麻烦的，以后还需要多总结
