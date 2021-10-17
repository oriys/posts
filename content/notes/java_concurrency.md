---
title: "Java 并发知识大纲"
date: 2021-09-03T22:16:38+08:00
draft: true
---

## 实现多线程

### 继承 Thread 类，重写 run 方法

```java
class Demo extends Thread {

    @Override
    public void run() {
        System.out.println("implements thread via thread class");
    }
}
```

### 实现 Runnable 接口，并把 Runnable 对象传给 Thread 类

```java
class Demo implements Runnable {

    @Override
    public void run() {
        System.out.println("implements thread via runnable interface");
    }
}
```

### 对比

实现 Runnable 更好，解耦方法实现，可以节省资源，利用线程池，无需与 Thread 类的创建销毁绑定，Thread 有单继承限制。

### 本质

```java

    /**
     * If this thread was constructed using a separate
     * {@code Runnable} run object, then that
     * {@code Runnable} object's {@code run} method is called;
     * otherwise, this method does nothing and returns.
     * <p>
     * Subclasses of {@code Thread} should override this method.
     *
     * @see     #start()
     * @see     #stop()
     * @see     #Thread(ThreadGroup, Runnable, String)
     */
    @Override
    public void run() {
        if (target != null) {
            target.run();
        }
    }
```

最终调用`targe.run()`方法

重写`run()`方法

### 同时传入 Runnable 对象和重写 Thread 的 run 方法

```java
public class Main extends Thread implements Runnable {

    public static void main(String[] args) throws InterruptedException {
        Thread t = new Thread(() -> {
            System.out.println("Runnable");
        }) {
            @Override
            public void run() {
                System.out.println("Thread");
            }
        };
        t.start();
        t.join();
    }
    // output: Thread
```

### 线程池创建线程

最终通过 Thread 传入 Runnable 来创建线程

### 通过 Callable 和 FutureTask 创建线程

本质上 FutureTask 通过实现了 Runnable 接口创建线程

### 定时器 Timer

Timer 通过传入 TimerTask 实现多线程，TimerTask 本质上继承了 Runnable 接口

```java

public abstract class TimerTask implements Runnable {
    //...
}

```

### 匿名内部类和 Lambda 表达式

本质上都是通过传入 Thread 传入 Runnable 对象

```java
public class Main  {
    public static void main(String[] args){
        new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println("anonymous inner class");
            }
        }).start();

        new Thread(() -> {
            System.out.println("lambda expression");
        }).start();
    }
}

```

## 启动线程的方法

### `start()`方法的含义

```java
public class Main {
    public static void main(String[] args) {
        new Thread(()->{
            System.out.println("RUN");
        }).start();
    }
}
```

调用`start()`方法通知 JVM 有资源时启动新线程

不能重复执行`start()`方法，启动新线程时检查线程状态，加入新线程，调用`start0()`方法

### `run()`方法的含义

在主线程中直接执行 Runnable 的`run()`方法，相当于执行一个普通方法  
在 Thread 类中执行调用`start()`方法，再由`start()`方法调用`run()`方法，从而创建多线程

### 正确停止线程

使用`interrupt()`来通知，而不是强制停止线程，把停止的实现的交给线程本身，是否相应中断，何时中断。  
停止线程的时机：执行完毕，未捕获异常

1. 普通情况，用轮训判断`isInterrupted()`是否被中断

```java
public class Main {
    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(() -> {
            long sum = 0L;
            for (long i = 0; i < 10_000_000_000_000_000L && !Thread.currentThread().isInterrupted(); i++) {
                sum += i;
                if (sum > 10_000_000_000_000_000L) {
                    Thread.currentThread().interrupt();
                    System.out.println(i);
                }
            }
            System.out.println(sum);
        });
        thread.start();
        thread.join();
    }
}
```

2. 线程阻塞

当线程在 sleep 时执行它的`interrupt()`方法，sleep 会响应中断，抛出异常

```java
public class Main {
    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(() -> {
            long sum = 0L;
            try {
                while (sum < 300 && !Thread.currentThread().isInterrupted()) {
                    if (sum % 100 == 0) {
                        System.out.println(sum);
                    }
                    sum++;
                }
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
        thread.start();
        Thread.sleep(100);
        thread.interrupt();
    }
}
```

输出

```log
0
100
200
java.lang.InterruptedException: sleep interrupted
	at java.base/java.lang.Thread.sleep(Native Method)
	at me.ychy.Main.lambda$main$0(Main.java:16)
	at java.base/java.lang.Thread.run(Thread.java:829)
```

3. 每次迭代后阻塞

不再需要判断是否发生中断，而是由异常处理来捕获发生的异常，捕获异常后会清空中断标记位，需要二次中断

```java
public class Main {
    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(() -> {
            long sum = 0L;
            while (sum < 10000) {
                if (sum % 100 == 0) {
                    System.out.println(sum);
                }
                try {
                    Thread.sleep(10);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                sum++;
            }
        });
        thread.start();
        Thread.sleep(5000);
        thread.interrupt();
    }
}
```

---

中断处理

1. 在方法签名中抛出中断异常，交给上层调用者处理

```java
public class Main {
    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(() -> {
            while (true) {
                try {
                    throwException();
                } catch (InterruptedException e) {
                    System.out.println("catch you!!");
                }
            }
        });
        thread.start();
        Thread.sleep(5000);
        thread.interrupt();
    }

    private static void throwException() throws InterruptedException {
        if (Thread.currentThread().isInterrupted()) {
            throw new InterruptedException("throw an exception!!");
        }
    }
}
```

2. 在 catch 块中恢复中断

```java
public class Main {
    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(() -> {
            while (!Thread.currentThread().isInterrupted()) {
                System.out.println(Thread.currentThread().isInterrupted());
                throwException();
                System.out.println(Thread.currentThread().isInterrupted());
            }
            System.out.println("geme over");
        });
        thread.start();
        Thread.sleep(1000);
        thread.interrupt();
    }

    private static void throwException() {
        if (!Thread.currentThread().isInterrupted()) {
            try {
                System.out.println(Thread.currentThread().isInterrupted());
                throw new InterruptedException("throw an exception!!");
            } catch (InterruptedException e) {
                e.printStackTrace();
                System.out.println(Thread.currentThread().isInterrupted());
                Thread.currentThread().interrupt();
                System.out.println(Thread.currentThread().isInterrupted());
            }
        }
    }
}
```

3. 不应该屏蔽中断

---

能够响应中断的方法

```java
Object.wait()/wait(long)/wait(long,int)

Thread.sleep(long)/sleep(long,int)

Thread.join()/join(long)/join(long,int)

java.util.concurrent.BlockingQueue.take()/put(E)

java.util.concurrent.locks.Lock.lockInterruptibly()

java.util.concurrent.CountDownLatch.await()

java.util.concurrent.CyclicBarrier.await()

java.utill.concurrent.Exchanger.exchange(V)

java.nio.channels.InterruptibleChannel相关方法

java.nio.channels.Selector的相关方法

```

---

java 异常体系

- Throwablw
  - Error
    - AWTError
    - VirtualMachineError
      - StackOverflowError
      - OutOfMemoryError
  - Exception
    - RuntimeException
      - ArithmeticException
      - NullPointerException
      - IndexOutOfException
    - IOException
      - EOFException
      - FileNotFoundException

---

错误停止线程的方法

1. 使用 stop，suspend 和 resume 方法  
   用 stop 停止线程会造成线程突然停止，造成脏数据，资源泄漏等问题，会释放所有 monitor 锁  
   用 suspend 会造成线程挂起，但是不释放锁，造成死锁，不推荐使用 suspend 和 resume 来挂起恢复线程

2. 使用 volatile 变量设置标记位  
   当方法中存在阻塞方法时，可能无法正确停止线程

## 线程状态

- Java 进程状态
  - New
  - Runnable
  - Blocked
  - Waiting
  - Timed Waiting
  - Terminated

Java 进程状态机

![Java Thread FSM](https://i.imgur.com/NNL4TSl.png)

## Thread 和 Object 类中和线程相关的重要方法

Thread 和 Object 类中的重要方法

方法概览

| 类     | 方法名                | 简介                      |
| ------ | --------------------- | ------------------------- |
| Thread | sleep                 | 让线程休眠一段时间        |
|        | join                  | 等待其他线程执行完毕      |
|        | yield                 | 放弃已经获取到的 CPU 资源 |
|        | currentThread         | 获取当前执行线程的引用    |
|        | start/run             | 局动线程相关              |
|        | interrtupt            | 中断线程                  |
|        | stop/suspend/resuem   | 已废弃                    |
| Object | wait/notify/notifyAll | 让线程暂时休息和唤醒      |

---

线程被唤醒的条件

- 另一个线程调用这个对象的`notify()`方法且刚好被唤醒的是本线程
- 另一个线程调用这个对象的`notifyAll()`方法
- 过了`wait(long timeout)`规定的超时时间，如果传入 0 就是永久等待
- 线程自身调用了`interrupt()`

## 线程属性

## 线程的未捕获异常 UncaughtException 处理

## 线程的问题

## 常见面试题
