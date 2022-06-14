---

---

# Java

## 并发

### 一、使用线程

- 实现Runnable接口
- 实现Callable接口
- 继承Thread类

实现Runnable和Callable接口的类只能当作一个可以在线程中运行的任务，不是真正意义上的线程，因此最后还需要通过Thread来调用。可以理解为任务是通过线程驱动从而执行的。

#### **实现Runnable接口**

需要实现接口中的run()方法

```java
public class MyRunnable implements Runnable{
	@Override
	public void run(){
		//...
	}
}
```

使用Runnable实例在创建一个Thread实例，然后调用Thread实例的start()方法来启动线程。

```java
public static void main(String[] args){
	MyRunnable instance = new MyRunnable();
	Thread thread = new Thread(instance);
	thread.start();
}
```



#### **实现Callable接口**

与Runnable相比，Callable可以有返回值，返回值通过FutureTask进行封装。

```java
public class MyCallable implements Callable<Integer>{
	public Integer call(){
		return 123;
	}
}
```

```java
public static void main(String[] args) throws ExecutionException, InteruptedException{
    MyCallable mc = new MyCallable();
    FutureTask<Integer> ft = new FutureTask<>(mc);
    Thread thread = new Thread(ft);
    thread.start();
    System.out.println(ft.get());
}
```

#### **继承Thread类**

同样也需要实现run()方法，因为Thread类也实现了Runnable接口

当调用start()方法启动一个线程时，虚拟机会将该线程放入就绪队列中等待被调度，当一个线程被调度时会执行该线程的run()方法

```java
public static MyThread extends Thread{
	public void run(){
        //...
    }
}
```

```java
public static void main(String[] args){
    MyThread mt = new MyThread();
    mt.start();
}
```

#### 实现接口 VS 继承Thread

实现接口会更好一些，因为：

- Java不支持多重继承，因此继承了Thread类就无法继承其他类，但是可以实现多个接口
- 类可能只要求执行就行，继承整个Thread类开销过大

### 二、基础线程机制

#### Executor

Executor管理多个异步任务的执行，而无需程序员显式地管理线程的生命周期。这里的异步是指多个人物的执行互不干扰，不需要同步操作。

主要有三种Executor:

- CachedThreadPool: 一个任务创建一个线程

- FixedThreadPool: 所有任务只能使用固定大小的线程

- SingleThreadExecutor: 相当于大小为1的FixedThreadPool

  ```java
  public static void main(String[] args){
      ExecutorService executorService = Executor.newCachedThreadPool();
      for(int i = 0; i < 5; i++){
          executorService.executor(new MyRunnable());
      }
      executorService.shutdown();
  }
  ```

  

#### Daemon

守护线程是程序运行时在后台提供服务的线程，不属于程序中不可或缺的部分

当所有非守护线程结束时，程序也就终止，同时会杀死所有守护线程

main()属于非守护线程

在线程启动之前使用setDaemon()方法可以将一个线程设置为守护线程

```java
public static void main(String[] args){
    Thread thread = new Thread(new Runnable());
    thread.setDaemon(true);
}
```



#### sleep()

Thread.sleep(millisec)方法会休眠当前正在执行的线程，millisec单位为毫秒

sleep()可能会抛出InterruptedException, 因为异常不能跨线程传播回main()中，因此必须在本地进行处理。线程中抛出的其他异常也同样需要在本地进行处理。

```java
public voia run(){
    try{
        Thread.sleep(3000);
    }catch(InterruptedException e){
        e.printStackTrace();
    }
}
```



#### yield()

对静态方法Thread.yield()的调用声明了当前线程已经完成了生命周期中最重要的部分，可以切换给其他线程来执行。该方法只是对线程调度的一个建议，而且也只是建议具有相同优先级的其他线程可以运行。

```java
public void run()[
    Thread.yield();
]
```

### 三、中断

一个线程执行完毕后会自动结束，如果在运行过程中发生异常也会提前结束。

#### InterruptedException

通过调用一个线程的interrupt()来中断该线程，如果该线程处于阻塞、限期等待或者无限期等待状态，那么就会抛出InterruptedException, 从而提前结束该线程。但是不能终端I/O阻塞和synchronized锁阻塞



对于以下代码，在main()中启动一个线程之后再中断它，由于线程中调用了Thread.sleep()方法，因此会抛出一个InterruptedException，从而提前结束线程，不执行之后的语句。

```java
public class InterruptExample{
    private static class MyThread1 extends Thread{
        @Override
        public void run(){
            try{
                Thread.sleep(2000);
                System.out.println("Thread run");
            }catch(InterruptedException e){
                e.printStackTrace();
            }
        }
    }
}
```

```java
public static void main(String[] args) throws InterruptedException{
    Thread thread1 = new MyThread1();
    thread1.start();
    thread1.interrupt();
    System.out.println("Main run");
}
```

```html
Main run
java.lang.InterruptedException: sleep interrupted
    at java.lang.Thread.sleep(Native Method)
    at InterruptExample.lambda$main$0(InterruptExample.java:5)
    at InterruptExample$$Lambda$1/713338599.run(Unknown Source)
    at java.lang.Thread.run(Thread.java:745)
```

#### Interrupted

如果一个线程的run()方法执行一个无线循环，并且没有执行sleep()等会抛出InterruptedException的操作，那么调用线程的interrupt()方法就无法使线程提前结束。

但是调用线程interrupted()方法会设置线程的中断标记，此时调用interrupted()方法会返回true。因此可以在循环体中使用interrupted()方法来判断线程是否处于中断状态，从而提前结束线程。

```java
public class InterruptExample{
    private static class MyThread2 extends Thread {
        @OVerride
        public void run(){
            while(!interrupted()){
                //..
            }
            System.out.println("Thread end");
        }
    }
}
```

```java
public static void main(String[] args) throws InterruptedException{
    Thread thread2 = new Thread2();
    thread2.start();
    thread2.interrupt();
}
```

```html
Thread end
```

#### Executor的中断操作

调用Executor的shutdown()方法会等待线程都执行完毕之后再关闭，但是如果调用的是shutdownNow()方法，则相当于调用每个线程的interrupt()方法。

以下使用Lambda创建线程，相当于创建了一个匿名内部线程。

```java
public static void main(String[] args){
    ExecutorService executorService = Executors.newCachedThreadPool();
    executorService.executor(() -> {
        try{
            Thread.sleep(2000);
            System.out.println("Thread run");
        }catch(InterruptedException e){
            e.printStackTrace();
        }
    });
    executorService.shutdownNow();
    System.out.println("Main run");
}
```

```html
Main run
java.lang.InterruptedException: sleep interrupted
    at java.lang.Thread.sleep(Native Method)
    at ExecutorInterruptExample.lambda$main$0(ExecutorInterruptExample.java:9)
    at ExecutorInterruptExample$$Lambda$1/1160460865.run(Unknown Source)
    at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
    at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
    at java.lang.Thread.run(Thread.java:745)
```

如果只想中断Executor中的一个线程，可以通过使用submit()方法来提交一个线程，它会返回一个Future<?>对象，通过调用该对象的cancel(true)方法就可以中断线程。

```java
Future<?> future = executorService.submit(() -> {
    //..
});
future.cancel(true);
```

### 四、互斥同步

Java提供了两种锁机制来控制多个线程共享资源的互斥访问，第一个是JVM实现的synchronized,而另一个是JDK实现的ReentrantLock。

#### synchronized

1. 同步一个代码块

```java
public void func(){
    synchronized(this){
        //..
    }
}
```

它只作用于同一个对象，如果调用两个对象上的同步代码块，就不会进行同步。
