## 多线程
### 进程和线程
- 进程：一个应用程序的执行过程。应用程序一旦执行，就是一个进程。
- 线程：进程内部的一个执行单元。（不能独立执行，依存在应用程序之中）

通俗理解：启动QQ，开了一个进程。在QQ这个进程里，传输文字开了一个线程，传输语音开了一个线程。

### 创建线程的五种方式
- **继承Thread类并重写其run()，run()方法中定义需要执行的任务。**  
创建后的子类通过调用**start()**方法即可执行线程方法。  
```java
package javaP;
/*
 * 继承Thread类并重写run()创建线程
 * 步骤：
 * 1.定义UserThread类，继承Thread
 * 2.继承run()方法
 * 3.创建UserThread对象
 * 4.调用start()方法启动线程
 */
public class UserThread extends Thread {
	public void run() {
		for (int i = 0; i < 10; i++) {
			System.out.println(Thread.currentThread().getName() + " is running " + i);
		}
	}
}
```
```java
package javaP;

public class UserThreadTest {
	public static void main (String[] args) {
		Thread t1 = new UserThread();
		Thread t2 = new UserThread();
		t1.start();
		t2.start();
	}
}
```
   - 创建不同的Thread对象，多个线程之间自然不共享资源。  
  
- **创建Thread实例时，传入一个Runnable实例：**  
定义一个类创建Runnable 接口并重写该接口的**run()** 方法，此run方法是线程执行体。接着创建Runnable实现类的对象，作为创建Thread对象的参数target。  
```java
package javaP;
/*
 * 实现Runnable接口创建线程：
 * 步骤
 * 1. 定义一个类UserRun， 实现Runnable接口
 * 2. 重写run()方法
 * 3. 创建UserRun类的对象
 * 4. 创建Thread的对象，把UserRun类的对象构造方法的参数
 * 5. 启动线程
 */
public class UserRun implements Runnable{

	@Override
	public void run() {
		for (int i = 0; i < 10; i++) {
			System.out.println(Thread.currentThread().getName() + " is running " + i);
		}		
	}
}
```
```java
package javaP;

public class UserRunTest {
	
	public static void main(String[] args) {
		UserRun userRun = new UserRun();
		new Thread(userRun).start();
		new Thread(userRun).start();
	}

}
```
    - 实现线程间的资源共享。  

- **实现Callable接口实现带有返回值的线程**   
Callable接口如同Runnable接口的升级版，其提供的**call()** 方法作为现成的执行体，同时允许有 **返回值**。  
Callable对象不能直接作为Thread对象的参数target，因为Callable接口是Java5新增的接口，不是Runnable的子接口。  
为解决这个问题，引入了Future接口，此接口可以接受call()的返回值，**RunnableFuture接口** 是Future接口和Runnable接口的子接口，可以作为Thread对象的target。  
```java
package javaP;

import java.util.concurrent.Callable;

/*
 * 实现Callable接口实现带有返回值的线程
 * 步骤： 
 * 1. 定义类UserCallable，实现Callable接口
 * 2. 重写class方法
 * 3. 创建UserCallable的对象
 * 4. 创捷RunnableFuture接口的子类FutureTask的对象，构造函数的参数是UserCallable的对象
 * 5. 创建Thread类的对象，构造函数是FutureTask的对象
 * 6. 启动线程
 */
public class UserCallable implements Callable{

	@Override
	public Object call() throws Exception {
		System.out.println(Thread.currentThread().getName() + ": learning");
		return "learning";
	}
	
}
```
```java
package javaP;

import java.util.concurrent.ExecutionException;
import java.util.concurrent.FutureTask;

public class UserCallableTest {
	public static void main(String[] args) throws InterruptedException, ExecutionException {
		UserCallable userCallable = new UserCallable();
		FutureTask futureTask = new FutureTask(userCallable);
		
		Thread t = new Thread(futureTask);
		t.start();
		System.out.println(futureTask.get());
	}
}
```

- **继承TimerTask**  
Timer和TimerTask可以做为实现线程的另一种方式。  
Timer是一种线程设施，用于安排以后在后台线程中执行的任务。可安排任务执行一次，或者定期重复执行，可以看成一个**定时器** ，可以调度TimerTask。  
TimerTak是一个抽象类，实现了Runnable接口，所以具备了多线程的能力。  
```java
import java.util.Date;
import java.util.TimerTask;

/*
 * 步骤：
 * 1. 定义类UserTask，继承抽象类TimerTask
 * 2. 创建UserTask类的对象
 * 3. 创建Timer类的对象，设置任务的执行策略
 */
public class UserTask extends TimerTask{

	@Override
	public void run() {
		System.out.println(Thread.currentThread().getName() + " is running " + new Date());
	}

}
```
```java
import java.util.Timer;

public class UserTaskTest {
	public static void main(String[] args)  {
		UserTask userTask = new UserTask();
		Timer timer = new Timer();
		
		timer.schedule(userTask, 5000, 3000);  
		// 等待5秒钟后执行，每3秒输出
	}

}
```

- **通过线程池启动多线程**
#### 通过Excutors的工具类可以创建线程池。
 - **提高系统响应速度**，当有任务到达时，通过复用已存在的线程，无需等待新线程的创建便能立即执行。  
 - **降低系统资源能耗**，通过重用已存在的线程，降低线程创建和销毁造成的消耗。  
 - **方便线程并发数的管控**，因为线程若是无限制的创建，可能会导致内存占用过多而产生OOM，并且会造成cpu过度切换。
##### 线程池一：newFixedThreadPool固定大小线程池  
```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class FixThreadTest {
	
	public static void main(String[] args) {
	 //1. 创建固定大小的线程池  nThreads: 3
		ExecutorService ex = Executors.newFixedThreadPool(3);
	// 2. 使用线程池执行任务
		for (int i = 0; i < 5; i++) {
			ex.submit(new Runnable() {
				public void run() {
					for(int j = 0; j < 10; j++) {
						System.out.println(Thread.currentThread().getName() + "  " + j);
					}
				}
			});
		}
	// 3.关闭线程池（继续执行原任务，不再接受新任务）
		ex.shutdown();
	}
}
```
##### 线程池二：SingleThreadPoolExecutor单线程池  
可以串行执行任务，保证顺序按输入顺序执行。  
```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class SingleThreadExecutorTest {

	public static void main(String[] args) {
		ExecutorService ex = Executors.newSingleThreadScheduledExecutor();
		
		for (int i = 0; i < 5; i++) {
			ex.submit(new Runnable() {
				public void run() {
					for(int j = 0; j < 10; j++) {
						System.out.println(Thread.currentThread().getName() + "  " + j);
					}
				}
			});
		}
		ex.shutdown();
	}
}
```

##### 线程池三： CashedThreadPool()缓存线程池
```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class CashedThreadTest {
	public static void main(String[] args) {
		ExecutorService ex = Executors.newCachedThreadPool();
		
		for (int i = 0; i < 5; i++) {
			ex.submit(new Runnable() {
				public void run() {
					for(int j = 0; j < 10; j++) {
						System.out.println(Thread.currentThread().getName() + "  " + j);
					}
				}
			});
		}
		ex.shutdown();
	}
}
```

##### 线程池四：newScheduledThreadPool()创建一个周期性的线程池，支持定时及周期性执行任务
```java
import java.util.Date;
import java.util.concurrent.Executors;
import java.util.concurrent.ScheduledExecutorService;
import java.util.concurrent.TimeUnit;

public class ScheduledThreadPoolTest {
	public static void main(String[] args) {
		ScheduledExecutorService ex = Executors.newScheduledThreadPool(5);
	    ex.scheduleAtFixedRate(new Runnable() {
	    	public void run() {
	    		System.out.println(Thread.currentThread().getName() + " 执行 " + new Date());
	    	}
	    }, 5, 3, TimeUnit.SECONDS);
	}	
}
```

##### 线程池五：newWorkStealingPool新的线程池类ForkJoinPool的扩展
```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class NewWorkStealingPoolTest {
	public static void main(String[] args) throws InterruptedException{
		System.out.println("--start--");
		ExecutorService ex = Executors.newWorkStealingPool();
		for(int j = 0; j < 10; j++) {
			ex.submit(new Runnable() {
				public void run() {
					System.out.println(Thread.currentThread().getName());
				}
			});
		}
		//让主线程等待3秒，等子线程执行完
		Thread.sleep(3000);
		System.out.println("--end--");
	}	
}
```
Executors 的 5 个功能线程池虽然方便，但现在已经不建议使用了，而是建议直接通过使用 **ThreadPoolExecutor** 的方式，这样的处理方式让写的同学更加明确线程池的运行规则，规避资源耗尽的风险。

### Thread类和Runnable接口区别  
实现Runnable接口相对于继承Thread类来说，有如下优势：

- 适合相同程序的多个线程去处理同一资源的情况。

- 可以避免由于Java的单继承特性带来的局限。

- 增强了程序的健壮性，代码能够被多个线程共享，代码与数据是独立的。

Runnable接口是Thread类的父类。

#### 继承Thread类实现卖票操作
```java
public class MyThread extends Thread{
	
	private int ticket = 10;
	
	public MyThread(String name) {
		super(name);
	}
	
	public void run() {
		while (true) {
			if(ticket>0) {
				System.out.println(Thread.currentThread().getName() + "卖出第" + (10 - ticket-- + 1)+"张票");
			}
		}
	}
	
}

public class MyThreadTest {
	
	public static void main(String[] args) {
		MyThread myThread1 = new MyThread("窗口1");
		MyThread myThread2 = new MyThread("窗口2");
		MyThread myThread3 = new MyThread("窗口3");
		myThread1.start();
		myThread2.start();
		myThread3.start();
	}

}
```
**运行结果：**
```
窗口2卖出第1张票
窗口3卖出第1张票
窗口3卖出第2张票
窗口3卖出第3张票
窗口1卖出第1张票
窗口3卖出第4张票
窗口2卖出第2张票
窗口3卖出第5张票
...
```
各个窗口（线程）独立从第1张票卖到第10张票。
#### 实现Runnable接口实现卖票操作
```java
public class MyRunnable implements Runnable{
	
	private int ticket = 10;
	
	public void run() {
		while (true) {
			if(ticket>0) {
				System.out.println(Thread.currentThread().getName() + "卖出第" + (10 - ticket-- + 1)+"张票");
			}
		}
	}

}

public class MyRunnableTest {
	public static void main(String[] args) {
		MyRunnable myRunnable = new MyRunnable();
		
		Thread myThread1 = new Thread(myRunnable, "窗口1");
		Thread myThread2 = new Thread(myRunnable, "窗口2");
		Thread myThread3 = new Thread(myRunnable, "窗口3");
		myThread1.start();
		myThread2.start();
		myThread3.start();
	}
}
```
**运行结果：**  
```
窗口2卖出第1张票
窗口1卖出第2张票
窗口2卖出第3张票
窗口1卖出第5张票
窗口3卖出第4张票
窗口1卖出第7张票
窗口2卖出第6张票
窗口1卖出第9张票
窗口3卖出第8张票
窗口2卖出第10张票
```
虽然以上程序启动了3个线程，3个线程一共才卖出10票，即ticket的属性被所有的线程对象共享。

- 结论：继承Thread类，就是多个线程各自完成自己的任务；实现Runnable接口，就是多个线程共同完成一个任务。

### 线程start和run方法的区别

直接调用Thread实例的run()方法是无效的。

```java
public class UserThread extends Thread{
	
	public void run() {
		System.out.println(Thread.currentThread().getName() + "，执行run方法");
	}

}

public class UserTreadTest {
	
	public static void main(String[] args) {
		System.out.println(Thread.currentThread().getName()+",执行main方法");
		UserThread userThread = new UserThread();
		userThread.run();
	}

}
```

运行结果：

```
main,执行main方法
main，执行run方法
```

必须调用Thread实例的start()方法才能启动新线程。

```java
public class UserTreadTest {	
	public static void main(String[] args) {
		System.out.println(Thread.currentThread().getName()+",执行main方法");
		UserThread userThread = new UserThread();
		userThread.start(); 
	}
}
```

执行结果：
```
main,执行main方法
Thread-0，执行run方法
```

- 总结：执行start()可以启动线程，执行run方法并不会启动线程。

###  线程的优先级

Java可以对线程设定优先级，设定优先级的方法是：
```java
Thread.setPriority(int n) // 1~10, 默认值5
```

优先级范围1~10，默认5，10最高

不能通过设置优先级来确保高优先级的线程一定会先执行。（优先级无法保障线程执行次序）

优先级高的线程被操作系统调度的优先级较高，操作系统对高优先级线程可能调度更频繁（优先级的线程获取CPU资源的概率较大，优先级低的并非没机会执行）

主线程main的优先级是5。

##### Java线程的优先级案例

```java
public class UserThread extends Thread {
	
	public void run() {
		for (int i = 0; i < 10; i++) {
			System.out.println(Thread.currentThread().getName() + "，第" + i + "次执行");
		}
	}
}
public class UserRunn implements Runnable {
	
	public void run() {
		for (int i = 0; i < 10; i++) {
			System.out.println(Thread.currentThread().getName() + "，第" + i + "次执行");
		}
	}

}
public class Test {
	
	public static void main(String[] args) {		
		Thread t1 = new UserThread();
		Thread t2 = new Thread(new UserRunn());
		
		//设置线程的优先级
		t1.setPriority(10);
		t2.setPriority(1);
		
		t1.start();
		t2.start();
	}

}
```

### 定时线程的任务调度
#### 定时线程schedule()：延时不追加执行任务
```java
import java.util.Date;
import java.util.TimerTask;

public class UserTask extends TimerTask {
	
	public void run() {
		System.out.println(Thread.currentThread().getName()+":"+ new Date());
	}

}

import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.Timer;

public class Test {
	public static void main(String[] args) throws ParseException {		
		Timer t = new Timer();
		SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");
		Date date = sdf.parse("2022-02-26 19:00:00");
		t.schedule(new UserTask(), date, 5000);
	}
}
```
#### 定时线程scheduleAtFixedRate()：延时追加执行任务 

时间超过指定开始时间，（程序一开始执行时就会）补回应该执行的线程次数

```java
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.Timer;

public class Test {
	public static void main(String[] args) throws ParseException {		
		Timer t = new Timer();
		SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");
		Date date = sdf.parse("2022-02-26 19:00:00");
		t.scheduleAtFixedRate(new UserTask(), date, 5000);
	}
}
```

###  接口同步回调和异步回调

#### 同步调用：

一种阻塞式调用，调用方要等待对方执行完毕才返回，它是一种单向调用

接口同步回调案例
```java
import javax.security.auth.callback.Callback;

public class MyCallback implements Callback {
	
	public void process(int status) {
		System.out.println("处理成功，返回状态为："+ status);
	}

}
```
```java
public class Server {
	public void getMsg(MyCallback callback, String msg) throws InterruptedException {
		System.out.println("服务器端获得消息：" + msg);
		// 模拟消息处理，等待两秒钟
		Thread.sleep(2000);
		System.out.println("服务器端处理成功，返回状态为200");
		//处理完消息，调用回调方法，告知客户端
		callback.process(200);
	}

}
```
```java
public class Client {
	Server server;
	
	public Client(Server server) { this.server = server; }
	
	public void sendMsg(final String msg) {
		System.out.println("客户端发送消息："+ msg);
		try{
			server.getMsg(new MyCallback(), msg);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		System.out.println("客户端已经发送消息给客户端，请稍等");
	}

}
```
```java
public class Test {
	
	public static void main(String[] args) {
		Server server = new Server();
		Client client = new Client(server);
		client.sendMsg("我要充值");
	}

}
```

**运行结果：**（同步调用）

```
客户端发送消息：我要充值
服务器端获得消息：我要充值
服务器端处理成功，返回状态为200
处理成功，返回状态为：200
客户端已经发送消息给客户端，请稍等
```

#### 异步调用：

一种类似消息或事件的机制，不过它的调用方向刚好相反，接口的服务在收到某种讯息或发生某种事件时，会主动通知客户方。

(不需要等待被调用方执行完毕才返回）

接口异步调用案例
```java
public class Client {
	Server server;
	
	public Client(Server server) { this.server = server; }
	
	public void sendMsg(final String msg) {
		System.out.println("客户端发送消息："+ msg);
//		try{
//			server.getMsg(new MyCallback(), msg);
//		} catch (InterruptedException e) {
//			e.printStackTrace();
//		}
		new Thread(new Runnable() {
			public void run() {
				try {
					server.getMsg(new MyCallback(), msg);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
		}).start();
		System.out.println("客户端已经发送消息给客户端，请稍等");
	}
}
```

运行结果：

```
客户端发送消息：我要充值
客户端已经发送消息给客户端，请稍等
服务器端获得消息：我要充值
服务器端处理成功，返回状态为200
处理成功，返回状态为：200
```

- 总结：回调分为同步和异步，区别就是需不需要等待服务器端的返回结果。
