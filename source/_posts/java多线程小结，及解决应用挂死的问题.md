title: java多线程小结，及解决应用挂死的问题
date: 2013-09-24 10:59
categories: java 
---
这两天为了定位JBOSS老是挂死的问题，学习了一下JAVA多线程方面的知识，在此总结一下
<!--more-->

# 在Java程序中，JVM负责线程的调度

线程调度是指按照特定的机制为多个线程分配CPU的使用权。 

调度的模式有两种：分时调度和抢占式调度。分时调度是所有线程轮流获得CPU使用权，并平均分配每个线程占用CPU的时间；抢占式调度是根据线程的优先级别来获取CPU的使用权。

JVM的线程调度模式采用了抢占式模式。 

# Thread类实际上也是实现了Runnable接口的类

在启动的多线程的时候，需要先通过Thread类的构造方法Thread(Runnable target) 构造出对象，然后调用Thread对象的start()方法来运行多线程代码。 实际上所有的多线程代码都是通过运行Thread的start()方法来运行的。因此，不管是扩展Thread类还是实现Runnable接口来实现多线程，最终还是通过Thread的对象的API来控制线程的，熟悉Thread类的API是进行多线程编程的基础

# JAVA多线程涉及到2个问题

一个是线程的调度，另一个是线程的同步

# 线程的状态

new、runnable、running、waiting、timed_waiting、blocked、dead 

当执行new Thread(Runnable r)后，新创建出来的线程处于new状态，这种线程不可能执行 

当执行thread.start()后，线程处于runnable状态，这种情况下只要得到CPU，就可以开始执行了。

runnable状态的线程，会接受JVM的调度，进入running状态，但是具体何时会进入这个状态，是随机不可知的 

running状态中的线程最为复杂，可能会进入runnable、waiting、timed_waiting、blocked、dead状态： 

如果CPU调度给了别的线程，或者执行了Thread.yield()方法，则进入runnable状态，但是也有可能立刻又进入running状态 

如果执行了Thread.sleep(long)，或者thread.join(long)，或者在锁对象上调用object.wait(long)方法，则会进入timed_waiting状态 

如果执行了thread.join()，或者在锁对象上调用了object.wait()方法，则会进入waiting状态 

如果进入了同步方法或者同步代码块，没有获取锁对象的话，则会进入blocked状态 

处于waiting状态中的线程，如果是因为thread.join()方法进入等待的话，在目标thread执行完毕之后，会回到runnable状态；如果是因为object.wait()方法进入等待的话，在锁对象执行object.notify()或者object.notifyAll()之后会回到runnable状态 

处于timed_waiting状态中的线程，和waiting状态中的差不多，只不过是设定时间到了，就会回到runnable状态 

处于blocked状态中的线程，只有获取了锁之后，才会脱离阻塞状态 

当线程执行完毕，或者抛出了未捕获的异常之后，会进入dead状态，该线程结束 

# 当线程池中线程都具有相同的优先级，调度程序的JVM实现自由选择它喜欢的线程

这时候调度程序的操作有两种可能：一是选择一个线程运行，直到它阻塞或者运行完成为止。二是时间分片，为池内的每个线程提供均等的运行机会。 

# 设置线程的优先级

线程默认的优先级是创建它的执行线程的优先级。可以更改线程的优先级。 JVM从不会改变一个线程的优先级。然而，1-10之间的值是没有保证的。一些JVM可能不能识别10个不同的值，而将这些优先级进行每两个或多个合并，变成少于10个的优先级，则两个或多个优先级的线程可能被映射为一个优先级。 

# Thread.yield()

方法作用是：暂停当前正在执行的线程对象，并执行其他线程。 

yield()应该做的是让当前运行线程回到可运行状态，以允许具有相同优先级的其他线程获得运行机会。因此，使用yield()的目的是让相同优先级的线程之间能适当的轮转执行。但是，实际中无法保证yield()达到让步目的，因为让步的线程还有可能被线程调度程序再次选中。 结论：yield()从未导致线程转到等待/睡眠/阻塞状态。在大多数情况下，yield()将导致线程从运行状态转到可运行状态，但有可能没有效果。 

# 另一个问题是线程的同步，这个我感觉比调度更加复杂一些

Java中每个对象都有一个“内置锁”，也有一个内置的“线程表” 

当程序运行到非静态的synchronized方法上时，会获得与正在执行代码类的当前实例（this实例）有关的锁；当运行到同步代码块时，获得与声明的对象有关的锁 释放锁是指持锁线程退出了synchronized方法或代码块。 当程序运行到synchronized同步方法或代码块时对象锁才起作用。 一个对象只有一个锁。所以，如果一个线程获得该锁，就没有其他线程可以获得锁，直到第一个线程释放（或返回）锁。这也意味着任何其他线程都不能进入该对象上的synchronized方法或代码块，直到该锁被释放。 

# 当提到同步（锁定）时，应该清楚是在哪个对象上同步（锁定）

要明确当前的锁对象,对理解线程的行为非常有帮助

# obj.wait() obj.notify() obj.notifyAll() 

关于这3个方法，有一个关键问题是： 必须从同步环境内调用wait()、notify()、notifyAll()方法。只有拥有该对象的锁的线程，才能调用该对象上的wait()、notify()、notifyAll()方法 

与每个对象具有锁一样，每个对象也可以有一个线程列表，他们等待来自该对象的通知。线程通过执行对象上的wait()方法获得这个等待列表。从那时候起，它不再执行任何其他指令，直到调用对象的notify()方法为止。如果多个线程在同一个对象上等待，则将只选择一个线程（不保证以何种顺序）继续执行。如果没有线程等待，则不采取任何特殊操作。 

# 代码实例

```
public class ThreadA {

	public static void main(String[] args) {

		ThreadB b = new ThreadB();// ThreadB status: new

		b.start();// ThreadB status: runnable

		synchronized (b) {
			try {				
				System.out.println("等待对象b完成计算。。。");
				Thread.sleep(60000);
				b.wait();
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
			System.out.println("b对象计算的总和是：" + b.total);
		}
	}

}

public class ThreadB extends Thread {

	int total;

	public void run() {
		synchronized (this) {
			for (int i = 0; i < 101; i++) {
				total += i;
			}
			notifyAll();
		}
	}

} 

```

jstack输出的结果是： 

```
"main" prio=6 tid=0x00846800 nid=0x1638 waiting on condition [0x0092f000]    java.lang.Thread.State: TIMED_WAITING (sleeping) at java.lang.Thread.sleep(Native Method) at net.kyfxbl.lock.ThreadA.main(ThreadA.java:20) - locked <0x22a18a90> (a net.kyfxbl.lock.ThreadB) 
"Thread-0" prio=6 tid=0x02bbb800 nid=0x1410 waiting for monitor entry [0x02f0f000]    java.lang.Thread.State: BLOCKED (on object monitor) at net.kyfxbl.lock.ThreadB.run(ThreadB.java:11) - waiting to lock <0x22a18a90> (a net.kyfxbl.lock.ThreadB) 
```

可以看到，主线程和新线程在同一个对象上锁定，主线程的方法里执行了Thread.sleep(60000)，因此进入了TIMED_WAITING状态，而新线程则进入BLOCKED状态

```
public class ThreadA {

	public static void main(String[] args) {

		ThreadB b = new ThreadB();// ThreadB status: new

		b.start();// ThreadB status: runnable

		synchronized (b) {

			try {
				System.out.println("等待对象b完成计算。。。");
				b.wait();
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
			System.out.println("b对象计算的总和是：" + b.total);
		}
	}

}

public class ThreadB extends Thread {

	int total;

	public void run() {

		synchronized (this) {

			try {
				Thread.sleep(60000);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}

			for (int i = 0; i < 101; i++) {
				total += i;
			}
			notifyAll();
		}
	}

}

```

jstack输出的结果是： 

```
"main" prio=6 tid=0x00846800 nid=0x1684 in Object.wait() [0x0092f000]    java.lang.Thread.State: WAITING (on object monitor) at java.lang.Object.wait(Native Method) - waiting on <0x22a18b08> (a net.kyfxbl.lock.ThreadB) at java.lang.Object.wait(Object.java:485) at net.kyfxbl.lock.ThreadA.main(ThreadA.java:22) - locked <0x22a18b08> (a net.kyfxbl.lock.ThreadB) 
"Thread-0" prio=6 tid=0x02bcc800 nid=0x19c waiting on condition [0x02f0f000]    java.lang.Thread.State: TIMED_WAITING (sleeping) at java.lang.Thread.sleep(Native Method) at net.kyfxbl.lock.ThreadB.run(ThreadB.java:12) - locked <0x22a18b08> (a net.kyfxbl.lock.ThreadB) 
```

2个线程还是在同一个对象上同步，但这次主线程立刻执行了b.wait()方法，因此释放了对象b上的锁，自己进入了WAITING状态。接下来新线程得到了对象b上的锁，所以没有进入阻塞状态，紧接着执行Thread.sleep(60000)方法，进入了TIMED_WAITING状态

```
public class ThreadA {

	public static void main(String[] args) {

		ThreadB b = new ThreadB();// ThreadB status: new

		b.start();// ThreadB status: runnable

		synchronized (b) {

			try {
				System.out.println("等待对象b完成计算。。。");
				b.wait();// ThreadB status: running
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
			System.out.println("b对象计算的总和是：" + b.total);
		}
	}

}

public class ThreadB extends Thread {

	int total;

	public void run() {

		synchronized (this) {

			for (int i = 0; i < 101; i++) {
				total += i;
			}

			notifyAll();

			try {
				System.out.println("我要睡了");
				Thread.sleep(60000);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}

		}
	}

}
```
jstack输出的结果是：

```
"main" prio=6 tid=0x00846800 nid=0x3ec in Object.wait() [0x0092f000]    java.lang.Thread.State: BLOCKED (on object monitor) at java.lang.Object.wait(Native Method) - waiting on <0x22a18ba0> (a net.kyfxbl.lock.ThreadB) at java.lang.Object.wait(Object.java:485) at net.kyfxbl.lock.ThreadA.main(ThreadA.java:20) - locked <0x22a18ba0> (a net.kyfxbl.lock.ThreadB) 
"Thread-0" prio=6 tid=0x02bbb800 nid=0x14b4 waiting on condition [0x02f0f000]    java.lang.Thread.State: TIMED_WAITING (sleeping) at java.lang.Thread.sleep(Native Method) at net.kyfxbl.lock.ThreadB.run(ThreadB.java:19) - locked <0x22a18ba0> (a net.kyfxbl.lock.ThreadB)
```
 
当主线程执行b.wait()之后，就进入了WAITING状态，但是新线程执行notifyAll()之后，有一个瞬间主线程回到了RUNNABLE状态，但是好景不长，由于这个时候新线程还没有释放锁，所以主线程立刻进入了BLOCKED状态 

# wait()和notify()

当在对象上调用wait()方法时，执行该代码的线程立即放弃它在对象上的锁。然而调用notify()时，并不意味着这时线程会放弃其锁。如果线程仍然在完成同步代码，则线程在移出之前不会放弃锁。因此，只要调用notify()并不意味着这时该锁被释放 

# 与线程休眠类似，线程的优先级仍然无法保障线程的执行次序

只不过，优先级高的线程获取CPU资源的概率较大，优先级低的并非没机会执行。 

# 子线程与父线程的优先级

在一个线程中开启另外一个新线程，则新开线程称为该线程的子线程，子线程初始优先级与父线程相同

# JRE判断程序是否执行结束的标准是所有的前台执线程行完毕了，而不管后台线程的状态

因此，在使用后台线程时候一定要注意这个问题

# 下面说说我们这次JBOSS挂死问题的处理方法 

现象：系统运行一段时间之后，发现有几个子系统无法访问了，但是另外几个可以。CPU占用达到100% 

观察了一下，发现无法访问的应用都部署在同一个JBOSS里，于是把该JBOSS的堆栈用jstack命令输出 

发现里面有大量的线程处于BLOCKED状态，均是在执行到c3p0的一个方法里的某一行时，BLOCKED住了 

于是下载c3p0的源码，跟进去看了一下，这是一个同步方法，内部会去获取数据库连接，如果获取到连接，就进行下一步操作，如果获取不到，就执行sleep(long timeout)方法。 

反推一下，我猜测可能是这样的： 由于某段代码没有释放数据库连接,导致连接池中的连接耗尽,进而造成部分线程无限TIMED_WAITING,于是其余线程都BLOCKED,最终导致线程被占满

后来对所有涉及到数据库连接的代码进行排查，发现确实有几个地方做完数据库操作以后，没有释放连接。把这部分代码改掉，重新启动JBOSS，没有再出现JBOSS挂起的现象