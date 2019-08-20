title: java的方法调用，参数是按值传递还是按引用传递
date: 2013-09-24 11:22
categories: java  
---
各种语言都涉及到方法调用，一个基本的问题就是：参数是怎么传递的
<!--more-->

通常认为有2种方式：按值传递，按引用传递 

按值传递指的是，方法内部对参数的赋值，在方法外部对参数无影响；按引用传递则相反。比如

```
public static void main(String[] args) {

		int i = 2;

		changeNumber(i);

		System.out.println(i);

	}

	private static void changeNumber(int old) {
		old = 3;
	}

```
上面的代码，如果最后输出的是2，就是按值传递；如果输出3，就是按引用传递 

在java里，当然输出的是2。也就是说，java语言的方法调用，是按值传递来处理的 

问题是，这种定义不一定是准确的。前面的例子传递的参数是基本类型，但是当传递的参数是对象实例的引用时，就不一样了

```
public static void main(String[] args) {

		ForTest t = new ForTest(2);
		System.out.println(t.i);

		changeTest(t);
		System.out.println(t.i);

		changeTest2(t);
		System.out.println(t.i);

	}

	public static void changeTest(ForTest t) {
		t = new ForTest(3);
	}

	public static void changeTest2(ForTest t) {
		t.i = 3;
	}

	static class ForTest {

		public int i;

		public ForTest(int old) {
			this.i = old;
		}
	}

```
可以看到，在方法内部给引用赋一个新的对象，并没有影响：引用本身，仍然是按值传递的。

但是在方法内部对引用指向的对象实例做的操作，却是持久性的 

所以，不能简单地说java是按值调用或是按引用调用，或许这种定义本身就是不精确的 

java的这个特性，也引入了一些影响： 

比如由于引用本身是按值调用的，就没有办法在方法内部给参数赋新值了； 

比如有时候会不小心在方法内部改变了对象实例的字段，对方法调用者来说，这是不可知的。各种编程规范都不鼓励这种做法，但是在技术上没有办法强制保证这一点