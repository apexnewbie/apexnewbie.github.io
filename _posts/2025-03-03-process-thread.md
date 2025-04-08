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

---

# 🧠 Java 多线程学习笔记：`Thread` vs `Runnable`

---

## 🧩 一、两者是什么？

| 项目 | Thread 类 | Runnable 接口 |
|------|-----------|----------------|
| 类型 | 类（实现了 Runnable） | 函数式接口（只有一个 run 方法） |
| 作用 | 表示一个线程 + 包含要执行的代码 | 只定义“线程要执行的任务” |
| 核心方法 | `start()`, `run()`, `join()`, `sleep()` | `run()` |
| 本质 | 是一个线程对象 | 是任务内容，线程通过它执行任务 |

---

## 🧪 二、使用方式对比

### ✅ 1. 继承 Thread 类

```java
class MyThread extends Thread {
    public void run() {
        System.out.println("任务执行中：" + getName());
    }
}

new MyThread().start();
```

✅ 优点：
- 写法简单
- 适合快速 demo

⚠️ 缺点：
- Java 单继承，不能再继承其他类
- 耦合高，不利于任务复用

---

### ✅ 2. 实现 Runnable 接口

```java
class MyTask implements Runnable {
    public void run() {
        System.out.println("任务执行中：" + Thread.currentThread().getName());
    }
}

new Thread(new MyTask(), "线程名").start();
```

✅ 优点：
- 任务逻辑与线程控制解耦
- 可共享同一任务对象（支持资源共享）
- 更适用于线程池等高级并发框架

⚠️ 缺点：
- 需要多一步封装到 Thread

---

## 🌈 三、线程私有 vs 共享变量实现方式

| 类型 | Thread 方式 | Runnable 方式 |
|------|-------------|----------------|
| 私有变量 | 每个线程 new 一个 Thread 子类对象 | 每个线程持有不同的 Runnable 对象 |
| 共享变量 | 使用 static 或外部共享对象 | 所有线程共用同一个 Runnable 对象 |

---

### 示例：
#### ✅ 私有变量
```java
new Thread(new MyTask(5), "T1").start();  // Runnable
new MyThread(5).start();                 // Thread
```

#### ✅ 共享变量
```java
SharedTask task = new SharedTask();
new Thread(task).start();               // 多线程共享
```

---

## 🔍 四、Thread 和 Runnable 类图结构（简化）

```plaintext
Thread implements Runnable

Thread {
    Runnable target;
    
    public void run() {
        if (target != null)
            target.run(); // 委托执行 Runnable 任务
        else
            // 自己的 run()
    }
}
```

---

## 🔐 五、最佳实践建议

| 需求 | 建议用法 |
|------|-----------|
| 简单测试 | 继承 Thread |
| 需要共享资源 | Runnable（共享同一个实例） |
| 多个线程执行不同任务 | Runnable（多个实例） |
| 大型项目 / 多线程复用 / 线程池 | Runnable ✅✅✅ |
| 需要任务有返回值 | Callable + Future |

---

## ✅ 总结口诀：

> 🚀 **“Thread 是演员，Runnable 是剧本”**  
> 🚀 **“要共享，就传同一个 Runnable；要隔离，就 new 多个”**  
> 🚀 **“实际开发推荐 Runnable，Thread 用于学习理解结构”**

---

我来帮你用**图解 + 类比 + 编程视角**讲清楚它们的**异同点**。

---

## 🧠 一句话区分（面试可用）：

> **并发（Concurrency）是逻辑上的同时发生，重点在切换；**  
> **并行（Parallelism）是物理上的同时发生，重点在同时执行。**

---

## 🔍 举个现实类比

假设你是一个单人窗口在处理顾客：

### ✅ 并发（Concurrency）：
- 你在同时处理多个顾客，但你一个一个切换过去处理一点点。
- 比如：你先处理 A 的单子 → 再去问 B 的问题 → 再回来继续 A → …
- **看起来多个任务一起进行，其实是你在快速切换处理**。

### ✅ 并行（Parallelism）：
- 你和你的同事一起上岗，一个人处理 A，一个人处理 B，**真正同时进行**。
- 这就像多个线程运行在多个 CPU 核心上。

---

## 🧪 程序视角看差异

| 概念 | 并发（Concurrency） | 并行（Parallelism） |
|------|----------------------|----------------------|
| 含义 | “同时应对”多个任务 | “同时运行”多个任务 |
| 是否真正同时 | ❌（可能是轮流切片执行） | ✅（真的同时执行） |
| 依赖硬件 | 不强依赖，可以在单核实现 | 必须多核/多CPU支持 |
| 重点 | 任务调度与切换 | 同时运行与加速 |
| 例子 | 单核 CPU 多线程交替执行 | 多核 CPU 每个核心跑一个线程 |
| 应用场景 | 异步 IO / 多客户端响应 | 高性能计算 / 大数据处理 |

---

## 📌 图解帮助理解：

```
🧠 并发（一个核心轮流切任务）：

    时间 →
线程A  ──┐      ┌───────┐     ┌──────
         │切换  │ 切换 │切换 │
线程B     └──────┘     └───────┘     ──

🧠 并行（多个核心各跑一个任务）：

线程A  ─────────────▶
线程B  ─────────────▶
```

---

## ☀️ 并发与并行的关系？

> **并发是更宽泛的概念，目标是提高系统的“响应能力”**，  
> **并行是并发的一种实现方式，目标是提高“执行效率”**。

你可以有并发但不并行（比如单核上跑多个线程）；  
你也可以并发 + 并行（多线程 + 多核同时执行）。

---

## 🚀 Java 中怎么体现这俩？

| 技术 | 属于并发还是并行？ | 说明 |
|------|--------------------|------|
| 多线程（Thread） | 并发 | 多任务轮流执行 |
| 线程池（Executor） | 并发 + 可并行 | 多任务交替执行，调度灵活 |
| 并行流（parallelStream） | 并行 | Java8 特性，自动利用多核 |
| Fork/Join 框架 | 并行 | 任务分治并行计算 |

---

## ✅ 一句话背诵总结（送你去面试）：

> 🚀 **“并发是解决多个任务如何交替执行的问题，并行是解决多个任务如何同时执行的问题。”**

---