# 1.**锁**

`Synchronized` 关键字是 Java 中用来实现同步的机制，它可以修饰代码块或方法，用于控制多个线程对共享资源的访问。在 `synchronized (lock) {}` 中，`lock` 是一个对象引用，通常用作同步锁。

具体来说，`synchronized (lock) {}` 的作用是获取 `lock` 对象的监视器（或称为锁），然后执行同步代码块。在同一个时间点，只有一个线程可以获取到 `lock` 对象的锁，其他线程必须等待直到锁被释放。这样可以保证同一时刻只有一个线程执行同步块中的代码，从而保证了共享资源的安全访问。

关于同步的其他实现方式还有：

1. **同步方法：** 使用 `synchronized` 关键字修饰方法，整个方法体都被视为同步代码块，相当于使用方法所属对象（或者类，对于静态方法）的实例作为锁。

   ```
   java复制代码public synchronized void synchronizedMethod() {
       // 同步方法体
   }
   ```

2. **静态同步方法：** 对于静态方法，可以使用 `synchronized` 修饰符来实现静态同步方法，锁定的是当前类的 `Class` 对象。

   ```
   java复制代码public static synchronized void synchronizedStaticMethod() {
       // 静态同步方法体
   }
   ```

3. **同步代码块（this）：** 在非静态方法中，可以使用 `this` 关键字来作为锁对象，锁定当前对象实例。

   ```
   java复制代码public void synchronizedBlock() {
       synchronized (this) {
           // 同步代码块
       }
   }
   ```

4. **同步代码块（类对象）：** 对于静态方法或需要锁定类对象的场景，可以使用类的 `Class` 对象来作为锁。

   ```
   java复制代码public static void synchronizedStaticBlock() {
       synchronized (MyClass.class) {
           // 静态同步代码块
       }
   }
   ```

5. **ReentrantLock：** `java.util.concurrent.locks.ReentrantLock` 是 Java 提供的显示锁实现，提供了比 `synchronized` 更多的灵活性和功能，如可中断性、公平锁和尝试锁定等特性。

```java
private final ReentrantLock lock = new ReentrantLock();

public void doSomething() {
    lock.lock();
    try {
        // 同步代码块
    } finally {
        lock.unlock();
    }
}
```

# 2.线程池

### 2.2.关闭线程池（工具类）

在项目中使用线程池执行长期运行的任务时，确保在适当的时候关闭线程池是非常重要的。如果你不关闭线程池，线程池中的线程会一直存在，可能会导致以下问题：

1. **资源泄露：** 线程池占用系统资源，如果不关闭线程池，这些资源（如线程对象）将不会被释放，可能导致内存泄露或资源耗尽。
2. **应用程序性能问题：** 长期运行的线程可能会占用较多的系统资源，影响应用程序的整体性能。

因此，在使用完毕后，建议通过合适的方式关闭线程池。以下是一些常见的关闭线程池的方法：

### 1. 调用 `shutdown()` 方法

调用 `shutdown()` 方法会平缓地关闭线程池，**它不会立即停止执行现有的任务**，而是等待所有任务执行完成后关闭。调用后不再接受新任务。

```
java
复制代码
executorService.shutdown();
```

### 2. 调用 `shutdownNow()` 方法

调用 `shutdownNow()` 方法**会尝试立即停止所有活动的线程**，并返回尚未执行的任务列表。这可能会导致一些任务未完成，适用于需要快速停止线程池的场景。

```
java
复制代码
executorService.shutdownNow();
```

### 3. 等待线程池中任务执行完成后再关闭

在调用 `shutdown()` 或 `shutdownNow()` 方法后，可以通过 `awaitTermination()` 方法等待所有任务执行完成后关闭线程池。这个方法会阻塞当前线程，直到所有任务完成或者超时。

```
java复制代码try {
    if (!executorService.awaitTermination(60, TimeUnit.SECONDS)) {
        executorService.shutdownNow(); // 如果超时还未完成，强制关闭
        if (!executorService.awaitTermination(60, TimeUnit.SECONDS)) {
            System.err.println("线程池未能完全关闭");
        }
    }
} catch (InterruptedException e) {
    executorService.shutdownNow(); // 如果等待过程中被中断，强制关闭
    Thread.currentThread().interrupt();
}
```

### 注意事项

- **确保及时关闭：** 在项目中合适的时机调用关闭方法，以避免资源泄露和性能问题。
- **异常处理：** 在调用 `awaitTermination()` 方法时，需要适当处理可能抛出的 `InterruptedException` 异常。
- **线程池的生命周期管理：** 确保了解每个线程池的使用场景，选择合适的关闭方式，以确保任务的正常执行和系统资源的合理利用。

综上所述，对于项目中使用的线程池，建议在不再需要执行任务时及时关闭，以确保应用程序的稳定性和性能。

## 4.关闭线程（对象）

#### 4.1.关闭线程池并重新创建

> 在关闭线程池后，可以立即重新创建一个新的线程池，并将新的任务提交到新创建的线程池中执行。这样可以确保即使旧线程池关闭了，新的任务也能够继续被调度执行

```java
import java.util.concurrent.Executors;
import java.util.concurrent.ScheduledExecutorService;
import java.util.concurrent.TimeUnit;

public class RescheduleTasksWithNewThreadPool {
    private static ScheduledExecutorService scheduler;

    public static void main(String[] args) {
        // 启动定时任务
        scheduleTasks();

        // 模拟关闭线程池并尝试调度新任务
        shutdownScheduler();

        // 在需要时重新创建线程池并重新调度任务
        scheduler = Executors.newScheduledThreadPool(5);
        scheduleTasks();
    }

    // 定时任务示例
    private static void scheduleTasks() {
        scheduler.scheduleAtFixedRate(() -> {
            // 这里放置定时执行的任务逻辑
            System.out.println("定时任务执行，当前时间：" + System.currentTimeMillis());
        }, 0, 1, TimeUnit.HOURS); // 每隔1小时执行一次
    }

    // 关闭线程池
    private static void shutdownScheduler() {
        scheduler.shutdown();
        try {
            if (!scheduler.awaitTermination(10, TimeUnit.SECONDS)) {
                scheduler.shutdownNow();
                if (!scheduler.awaitTermination(10, TimeUnit.SECONDS)) {
                    System.err.println("线程池未能完全关闭");
                }
            }
        } catch (InterruptedException e) {
            scheduler.shutdownNow();
            Thread.currentThread().interrupt();
        }
    }
}

```

​	在这个示例中，首先使用 `scheduler.shutdown()` 方法平缓关闭线程池，然后通过 `scheduler.awaitTermination()` 方法等待线程池中的任务执行完毕。如果超过指定的超时时间仍有未完成的任务，使用 `scheduler.shutdownNow()` 方法强制关闭线程池。然后立即通过 `Executors.newScheduledThreadPool(5)` 创建一个新的线程池，并重新调度任务



#### 4.2.使用 ScheduledThreadPoolExecutor

> 如果需要更精细的控制和监视，可以直接使用 `ScheduledThreadPoolExecutor` 类，它是 `ScheduledExecutorService` 接口的实现类，提供了更多的控制和监视方法

```java
import java.util.concurrent.ScheduledThreadPoolExecutor;
import java.util.concurrent.TimeUnit;

public class RescheduleTasksWithThreadPoolExecutor {
    private static ScheduledThreadPoolExecutor executor;

    public static void main(String[] args) {
        // 创建线程池
        executor = new ScheduledThreadPoolExecutor(5);

        // 启动定时任务
        scheduleTasks();

        // 模拟关闭线程池并尝试调度新任务
        shutdownExecutor();

        // 在需要时重新创建线程池并重新调度任务
        executor = new ScheduledThreadPoolExecutor(5);
        scheduleTasks();
    }

    // 定时任务示例
    private static void scheduleTasks() {
        executor.scheduleAtFixedRate(() -> {
            // 这里放置定时执行的任务逻辑
            System.out.println("定时任务执行，当前时间：" + System.currentTimeMillis());
        }, 0, 1, TimeUnit.HOURS); // 每隔1小时执行一次
    }

    // 关闭线程池
    private static void shutdownExecutor() {
        executor.shutdown();
        try {
            if (!executor.awaitTermination(10, TimeUnit.SECONDS)) {
                executor.shutdownNow();
                if (!executor.awaitTermination(10, TimeUnit.SECONDS)) {
                    System.err.println("线程池未能完全关闭");
                }
            }
        } catch (InterruptedException e) {
            executor.shutdownNow();
            Thread.currentThread().interrupt();
        }
    }
}

```

在这个示例中，使用 `ScheduledThreadPoolExecutor` 类来管理定时任务。通过 `executor.shutdown()` 和 `executor.awaitTermination()` 方法来关闭和等待线程池中的任务完成。然后根据需要重新创建新的 `ScheduledThreadPoolExecutor` 实例，并重新调度任务



## 5.多个任务共享一个线程池

> 在使用线程池时，如果希望在一个循环中提交多个任务，并且希望确保在循环结束前线程池不被意外关闭，可以通过以下方法来保护线程池的状态

### 5.1.**使用独立的控制变量**

> 在循环外部定义一个变量来控制线程池的关闭。这个变量可以是 `volatile` 类型的布尔变量，用来标记线程池是否可以关闭。在循环内部提交任务时，首先检查这个变量的状态，确保线程池在循环结束前不会被关闭。

### 5.2.同步块或者方法

> 可以使用同步块或者同步方法来保护线程池的操作，确保在提交任务和检查线程池状态之间的操作是原子性的，避免并发问题。

下面是一个示例代码，演示如何在循环中提交任务时保护线程池的状态：

```java
java复制代码import java.util.concurrent.*;

public class SubmitTasksInLoop {
    private static final int THREAD_POOL_SIZE = 10;
    private static final ThreadPoolExecutor THREAD_POOL_EXECUTOR = new ThreadPoolExecutor(
            THREAD_POOL_SIZE, THREAD_POOL_SIZE, 0L, TimeUnit.MILLISECONDS,
            new LinkedBlockingQueue<>()
    );

    private static volatile boolean canShutdown = false; // 控制线程池关闭的变量

    public static void main(String[] args) {
        // 启动线程池
        THREAD_POOL_EXECUTOR.prestartAllCoreThreads();

        // 模拟循环提交任务
        for (int i = 0; i < 100; i++) {
            // 提交任务前检查线程池是否可以关闭
            if (!canShutdown) {
                // 提交任务到线程池
                submitTask(new MyTask(i));
            } else {
                System.out.println("线程池已经关闭，停止提交任务。");
                break;
            }
        }

        // 等待任务执行完毕
        shutdownThreadPool();
    }

    private static void submitTask(Runnable task) {
        THREAD_POOL_EXECUTOR.submit(task);
    }

    private static void shutdownThreadPool() {
        // 标记可以关闭线程池
        canShutdown = true;
        THREAD_POOL_EXECUTOR.shutdown();
        try {
            if (!THREAD_POOL_EXECUTOR.awaitTermination(10, TimeUnit.SECONDS)) {
                THREAD_POOL_EXECUTOR.shutdownNow();
                if (!THREAD_POOL_EXECUTOR.awaitTermination(10, TimeUnit.SECONDS)) {
                    System.err.println("线程池未能完全关闭");
                }
            }
        } catch (InterruptedException e) {
            THREAD_POOL_EXECUTOR.shutdownNow();
            Thread.currentThread().interrupt();
        }
    }

    static class MyTask implements Runnable {
        private final int taskId;

        MyTask(int taskId) {
            this.taskId = taskId;
        }

        @Override
        public void run() {
            System.out.println("任务 " + taskId + " 执行，线程名：" + Thread.currentThread().getName());
            // 这里放置任务逻辑
            try {
                Thread.sleep(1000); // 模拟任务执行时间
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
    }
}
```

在这个示例中：

- 使用 `volatile boolean canShutdown` 变量来控制线程池的关闭。循环中的任务提交前首先检查这个变量的状态，确保在关闭标记被设置前不会关闭线程池。
- 在 `shutdownThreadPool` 方法中，设置 `canShutdown` 为 `true`，并关闭线程池。这样可以安全地关闭线程池，而不会影响正在执行的任务。

通过这种方法，可以确保在循环提交任务时线程池不会意外被关闭，同时能够在任务提交完成后安全地关闭线程池。



# 3.使用线程优化代码

## 3.1.任务并行化

> 将原本串行执行的任务拆分成可以并行执行的子任务，并将这些子任务分配给多个线程同时执行

## 3.2.线程池管理

> 使用线程池来管理线程的生命周期，避免频繁创建和销毁线程带来的开销，同时控制并发线程数量，避免资源耗尽和系统负载过高

## 3.3.任务划分

> 将大任务划分为多个小任务，每个小任务由一个线程处理，可以采用分治法等策略来划分任务

## 3.4.同步与通信

>  在多线程环境下，确保线程间的同步与通信是关键。使用适当的同步机制（如锁、信号量、条件变量等）来控制线程之间的访问顺序和共享资源的安全访问
