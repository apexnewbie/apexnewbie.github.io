---
layout: post
author: Aki
tags: [バグ]
---

## 线程进程相关

### Process & Thread

- **定义：**
  - **进程**：程序在运行时，系统为其分配资源的基本单位，拥有独立的内存空间。
  - **线程**：程序执行的最小单位，属于某个进程，线程间共享该进程的内存。

- **内存和资源：**
  - **进程**：每个进程有独立的内存空间，资源相对隔离。
  - **线程**：同一进程内的线程共享内存，但各自拥有独立的栈空间。

- **创建与销毁：**
  - **进程**：创建和销毁的开销较大。
  - **线程**：创建和销毁开销较小，运行效率高，适用于高并发场景。

- **上下文切换：**
  - **进程**：上下文切换时开销较大。
  - **线程**：上下文切换开销较小。

- **通信方式：**
  - **进程**：由于独立的地址空间，进程间通信（IPC）需要使用管道、共享内存、socket等。
  - **线程**：同一进程内的线程可以直接通过共享内存进行通信。

- **稳定性与安全性：**
  - **进程**：由于相互隔离，一个进程出错通常不会直接影响其他进程，提高了系统的稳定性和安全性。
  - **线程**：同一进程内线程间紧密耦合，一个线程的异常可能导致整个进程崩溃，影响整体稳定性。

- **并发和调度：**
  - **进程**：多进程并发适合独立任务或服务隔离，但系统调度开销大。
  - **线程**：多线程能更高效地利用多核资源，实现轻量级并发，但需要处理线程同步与数据一致性问题。

## 死锁相关

- **死锁的四个必要条件：**
  - **互斥条件**：某些资源不能被多个进程同时使用，即资源一次只能被一个进程占用。
  - **占有且等待**：进程在持有至少一个资源的同时，还在等待其他进程占有的资源。
  - **不可抢占**：资源分配后，不能被系统强制回收，只能由进程主动释放。
  - **循环等待**：存在一个进程环，每个进程都在等待下一个进程持有的资源，形成闭环（两个进程首尾相接）。


---

## Java 线程

### 1. 继承 Thread 类

**代码示例：**

```java
class MyThread extends Thread {
    @Override
    public void run() {
        System.out.println("Thread running: " + Thread.currentThread().getName());
    }
}

public class ThreadTest {
    public static void main(String[] args) {
        MyThread t1 = new MyThread();
        t1.start();  // 调用 start() 后内部会自动调用 run() 方法
    }
}
```

**讲解要点：**  
- 通过继承 `Thread` 类并重写 `run()` 方法来定义线程任务。  
- 调用 `start()` 方法启动线程，而不是直接调用 `run()`。

---

### 2. 实现 Runnable 接口

**代码示例：**

```java
class MyRunnable implements Runnable {
    @Override
    public void run() {
        System.out.println("Runnable thread: " + Thread.currentThread().getName());
    }
}

public class RunnableTest {
    public static void main(String[] args) {
        Thread thread = new Thread(new MyRunnable());
        thread.start();
    }
}
```

**讲解要点：**  
- 实现 `Runnable` 接口，将线程任务封装在 `run()` 方法中。  
- 将 `Runnable` 实例传给 `Thread` 构造器，调用 `start()` 启动线程。  
- 这种方式可以避免 Java 的单继承局限。

---

### 3. 使用 synchronized 实现线程安全

**代码示例：**

```java
public class SynchronizedExample {
    private int count = 0;
    
    // 使用 synchronized 关键字保证同一时刻只有一个线程访问该方法
    public synchronized void increment() {
        count++;
    }
    
    public void doIncrement() {
        for (int i = 0; i < 10000; i++) {
            increment();
        }
    }
    
    public static void main(String[] args) throws InterruptedException {
        SynchronizedExample example = new SynchronizedExample();
        Thread t1 = new Thread(() -> example.doIncrement());
        Thread t2 = new Thread(() -> example.doIncrement());
        
        t1.start();
        t2.start();
        
        t1.join();
        t2.join();
        
        System.out.println("Final count: " + example.count);
    }
}
```

**讲解要点：**  
- 使用 `synchronized` 确保对共享资源 `count` 的访问是互斥的，避免线程间竞态条件。  
- 使用 `join()` 方法等待线程执行完毕，再输出最终结果。

---

### 4. 使用 wait/notify 实现线程间通信

**代码示例：**

```java
public class WaitNotifyExample {
    private static final Object lock = new Object();
    
    public static void main(String[] args) {
        Thread producer = new Thread(() -> {
            synchronized (lock) {
                System.out.println("Producer: holding lock, doing work...");
                try {
                    Thread.sleep(1000); // 模拟生产工作
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("Producer: releasing lock and notifying");
                lock.notify(); // 通知等待线程
            }
        });
        
        Thread consumer = new Thread(() -> {
            synchronized (lock) {
                System.out.println("Consumer: waiting for notification");
                try {
                    lock.wait(); // 等待通知
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("Consumer: notified and proceeding");
            }
        });
        
        consumer.start();
        producer.start();
    }
}
```

**讲解要点：**  
- `wait()` 使当前线程释放锁并等待其他线程调用 `notify()` 或 `notifyAll()`。  
- `notify()` 唤醒在同一对象上等待的线程，但调用前必须持有该对象的锁。  
- 这种机制常用于线程间的协调与数据共享。

---

### 5. 使用线程池（ExecutorService）

**代码示例：**

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class ThreadPoolExample {
    public static void main(String[] args) {
        // 创建固定大小的线程池
        ExecutorService executor = Executors.newFixedThreadPool(2);
        
        // 提交任务给线程池执行
        executor.execute(() -> {
            System.out.println("Task 1 executed by " + Thread.currentThread().getName());
        });
        executor.execute(() -> {
            System.out.println("Task 2 executed by " + Thread.currentThread().getName());
        });
        
        // 关闭线程池，等待任务执行完毕
        executor.shutdown();
    }
}
```

**讲解要点：**  
- 使用 `ExecutorService` 管理线程，可避免频繁创建和销毁线程的开销。  
- 线程池可固定大小，也可以根据需要动态调整。  
- 通过 `execute()` 或 `submit()` 提交任务，实现线程复用和高效管理。

---

### 总结

- **线程创建方式**：继承 `Thread` 类和实现 `Runnable` 接口各有优势，前者简单直观，后者更灵活且避免了单继承限制。  
- **线程安全**：通过 `synchronized` 关键字、锁机制等方式保证共享资源的互斥访问。  
- **线程间通信**：利用 `wait/notify` 实现线程的协作，控制执行顺序。  
- **线程管理**：采用线程池（如 `ExecutorService`）高效管理线程资源，适用于大量并发任务场景。