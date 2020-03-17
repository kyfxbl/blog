title: UILabel的几个技巧
date: 2015-05-18 17:27
categories: iOS 
---
![radius](http://pic.kyfxbl.com/a15.jpg)
在iOS8绘制圆角，以及正确地消除UILabel的黑边
<!--more-->

# 在iOS8绘制圆角

直接通过layer的cornerRadius属性，设置圆角，发现在iOS8下显示错误（故意设置了红色边框，看得清楚）

![](http://pic.kyfxbl.com/radius1.jpg)

虽然边框是圆角，但是UILabel本身还是直角

在iOS8下，需要这样写：

```
cancelLabel.layer.cornerRadius = 5.f;
cancelLabel.layer.borderColor = [UIColor whiteColor].CGColor;// 设置成UILabel的背景色
[cancelLabel.layer setMasksToBounds:YES];// 关键
```
效果：

![](http://pic.kyfxbl.com/radius2.jpg)

# 当UILabel的size存在小数时，消除黑边

自定义了一个柱状图的组件，发现UILabel有一条黑边，检查代码发现：

```
CGFloat barLength = value * lengthPerValue;
UILabel *bar = [[UILabel alloc] initWithFrame:CGRectMake(80, 5, barLength, 30)];
```

关键在于barLength计算出来不是整数，结果可能是比如181.332这样的小数，所以graph hardware难以处理。解决的办法是用ceil或者round的函数，去掉小数部分就可以了

```
CGFloat barLength = ceil(value * lengthPerValue);
UILabel *bar = [[UILabel alloc] initWithFrame:CGRectMake(80, 5, barLength, 30)];
```
