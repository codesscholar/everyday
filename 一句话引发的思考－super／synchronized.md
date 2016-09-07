# 一句话引发的思考 － super／synchronized

## 起源
最近在学习java并发编程的相关知识，学到 **synchronized** 是一种可重入的内置锁的时候，看到一个例子：

```
class Widget {
	public synchronized void doSomething() {
		// ...
	}
}

class LoggingWidget extends Widget {
	public synchronized void doSomething() {
		System.out.println("son dosomething");
		super.doSomething();
	}
}
		
```
书中原话是这样：
> 由于Widget 和 LoggingWidget中的doSomething 方法都是synchronized方法，**因此每个doSomething方法在执行前都会获取Widget上的锁**。然而如果内置锁不是可重入的，那么调用super.doSomething时将无法获得Widget上的锁，就会发生死锁。

调用的时候是这样：
	
	Widget w = new LoggingWidget();
	w.doSomething();


## 疑惑
读这段话的时候给了我深深的疑惑**为什么执行每个doSomething都是获得Widget上的锁呢?**

我最初的理解是：w是LoggingWidget对象，然后w.doSomething方法在LoggingWidget对象上获取了一个锁。super.doSomething调用的是父类对象的方法，那么会在Widget上获取一个锁。

最后的结果应该是Widget对象持有一个锁，LoggingWidget持有一个锁，由于对象不同，怎么能说是同一个对象重入了两个锁呢？

## 解惑
** 自以为是的想法 **
	
当出现疑惑的时候，我在想，难道执行子类对象的重写方法之前一定会执行一遍父类对象的被重写的方法？但是在我查了很多资料的时候，才发现这是多么荒唐的想法，有这么荒唐的相关的原因就是对于继承super, synchronized等知识不熟悉。那么问题到底出现在哪呢？

	问题在：对super关键字的理解不够透测，对继承不够了解！
	
	
### synchronized 关键字
	
	第一个问题：synchronized锁的是什么？
要解决上面那个问题，首先要弄清楚的就是这个问题，答案是可以理解为锁的是**类**或者某个**类实例对象**。

这么说可能印象还不够深刻，先来个问题：

```
public class SynchronizeDemo {
	public static void main(String[] args) {
		Animal animal = new Animal();
		new Thread(new Runnable() {
			@Override
			public void run() {
				animal.eat();
			}
		}).start();
		
		new Thread(new Runnable() {
			@Override
			public void run() {
				animal.run();
			}
		}).start();
	}
}

class Animal {
	public synchronized void eat() {
		for (int i=0;i<10;i++) {
			System.out.println("eat");
			try {
				Thread.sleep(1000);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}
	public synchronized void run() {
		for (int i=0;i<10;i++) {
			System.out.println("run");
			try {
				Thread.sleep(1000);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}
}
```
问题是上面的两个线程执行的不同的方法会不会阻塞？

大家可以自己跑一次，结果是会的。那么我再详细说一下什么叫锁的是对象：
	
	1. 如果当两个线程访问同一个对象的同一个synchronized代码块时（非静态方法可以看作代码块的一种），
	会发生阻塞；
	2. 当一个线程访问这个对象的某个synchronized代码块，另一个线程访问这个对象的另外一synchronized
	代码块的时候，会阻塞。
	3. 当一个线程访问这个对象的某个synchronized代码块，另一个线程访问这个对象的非synchronized代码
	块的时候，不会阻塞。

所以说锁是对象锁，即使是不同的synchronized代码块，但是获取的是同一个锁。

至于类锁和对象锁的区别，这里就不详述了，于讨论的问题无关。

### 子类父类实例化  和 super
然后来**正确**解析一下 Widget w = new LoggingWidget() 这句话到底实例化了几个对象，答案是只实例化了一个LoggingWidget类型的对象。

那么super调用父类的方法这句话怎么解释呢，在JVM里面同时保存了父类和子类的被重写方法的实现，当调用super关键字的时候，并没有切换对象，而是让JVM去调用父类里面的实现方法而已。

## 最终解释
LoggingWidget对象实例在JVM中同时保存了两个同名的方法实现（是理解为两个不同的方法，还是同一方法，对于解决这个问题不是很关键，这是另一个问题，希望大神可以给出答案）

同一个对象的不同synchronized代码块内置锁不冲突的原因就是因为可重入，因为这是同一个线程。
>可重入：当某个线程请求一个由其它线程持有的锁的时候，会阻塞。然后，由于内置锁是可重入的，因此如果某个线程试图获得一个已由它自己持有的锁，这个请求会成功。“重入”意味着锁的操作粒度是“线程”，而不是调用。

## 总结
	1. super 的机制
	2. synchronized 锁的是对象
	3. synchronized 是一种可重入内置锁


## 参考
	java继承中子类对象调用父类方法的问题？
	https://www.zhihu.com/question/35418866
	
	深入理解子类和父类之间关系
	http://www.cnblogs.com/chenchuangfeng/archive/2013/06/10/3130329.html
	
	java synchronized详解
	http://www.cnblogs.com/GnagWang/archive/2011/02/27/1966606.html
