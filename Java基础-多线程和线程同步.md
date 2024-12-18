# Java 多线程和线程同步

#### 进程和线程

- 进程和线程的区别？需要更详细，线程间共享内存吗？
  
  - 操作系统中运行多个软件
  - 一个运行中的软件可能包含多个进程
  - 一个运行中的进程可能包含多个线程

- CPU 线程和操作系统线程
  
  - CPU 线程
    
    - 多核 CPU 的每个核各自独立运行，因此每个核一个线程
    - 四核八线程：CPU硬件方在硬件级别对 CPU 进行了一核多线程的支持（超线程技术），本质上依然是每个核一个线程
  
  - 操作系统线程
    
    操作系统利用时间分片的方式，把 CPU 的运行拆分给多条运行逻辑，即为操作系统的线程
  
  - 单核 CPU 也可以运行多线程操作系统

- 线程是什么：按代码顺序执行下来，执行完毕就结束的一条线。线程是操作系统能够进行运算调度的最小单位
  
  - UI 线程为什么不会结束？因为它在初始化完毕后会执行死循环，循环的内容是刷新界面
  - Android 每隔 16.6ms 会刷新一次屏幕

#### 多线程的使用

- Thread 和 Runnable，实际工作中不会使用
  
  ```java
  Thread thread = new Thread() {
      @Override
      public void run() {
          System.out.println("Thread started!");
      }
  };
  thread.start();
  ```
  
  ```java
  Runnable runnable = new Runnable() {
      @Override
      public void run() {
          System.out.println("Thread with Runnable started!");
      }
  };
  Thread thread = new Thread(runnable);
  thread.start();
  ```

- ThreadFactory
  
  ```java
  ThreadFactory factory = new ThreadFactory() {
      AtomicInteger count = new AtomicInteger(0);
  
      @Override
      public Thread newThread(Runnable r) {
          return new Thread(r, "Thread-" + count.incrementAndGet());
      }
  };
  
  Runnable runnable = new Runnable() {
      @Override
      public void run() {
          System.out.println(Thread.currentThread().getName() + " started!");
      }
  };
  
  Thread thread = factory.newThread(runnable);
  thread.start();
  Thread thread1 = factory.newThread(runnable);
  thread1.start();
  ```

- **Executor 和线程池**
  
  - 常用：newCachedThreadPool()
    
    创建一个带缓存线程池，如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程
    
    ```java
    Runnable runnable = new Runnable() {
        @Override
        public void run() {
            System.out.println("Thread with Runnable started!");
        }
    };
    
    Executor executor = Executors.newCachedThreadPool();
    executor.execute(runnable);
    executor.execute(runnable);
    executor.execute(runnable);
    ```
  
  - 短时批量处理：newFixedThreadPool()
    
    创建一个定长线程池，可控制线程最大并发数，超出的线程会在队列中等待
    
    ```java
    ExecutorService executor = Executors.newFixedThreadPool(20);
    for(Bitmap bitmap: bitmaps){
        executor.execute(bitmapProcessor(bitmap));
    }
    // 一次性批量操作，保守型结束
    executor.shutdown();
    ```
  
  - new SingleThreadExecutor() 单线程
  
  - new ScheduleThreadPool() 
  
  - new ThreadPoolExecutor
    
    - 最大线程数是线程池允许创建的最大线程数量。当工作队列满了之后，线程池会创建新的线程来处理任务，直到线程数量达到这个最大值。

- Callable 和 Future，不常用也比较难用
  
  Callable 接口类似于 Runnable，但是 Runnable 不会返回结果，并且无法抛出返回结果的异常，而 Callable功能更强大一些，被线程执行后，可以返回值，这个返回值可以被 Future 拿到，也就是说，Future 可以拿到异步执行任务的返回值。
  
  Future 就是对于具体的 Runnable 或者 Callable 任务的执行结果进行取消、查询是否完成、获取结果。必要时可以通过 get 方法获取执行结果，该方法会阻塞直到任务返回结果。
  
  ```java
  Callable<String> callable = new Callable<String>() {
      @Override
      public String call() {
          try {
              Thread.sleep(1500);
          } catch (InterruptedException e) {
              e.printStackTrace();
          }
          return "Done!";
      }
  };
  
  ExecutorService executor = Executors.newCachedThreadPool();
  Future<String> future = executor.submit(callable);
  try {
      String result = future.get();
      System.out.println("result: " + result);
  } catch (InterruptedException | ExecutionException e) {
      e.printStackTrace();
  }
  ```

#### 线程同步与线程安全

- synchronized
  
  - synchronized 方法
    
    ```java
    // monitor 默认是这个类
    private synchronized void count(int newValue) {
        x = newValue;
        y = newValue;
        if (x != y) {
            System.out.println("x: " + x + ", y:" + y);
        }
    }
    ```
  
  - synchronized 代码块
    
    ```java
    private final Object monitor1 = new Object();
    private final Object monitor2 = new Object();
    
    private void count(int newValue) {
        // this 指的是 monitor
        synchronized (this) {
            x = newValue;
            y = newValue;
            if (x != y) {
                System.out.println("x: " + x + ", y:" + y);
            }
        }
    }
    
    private void minus(int delta) {
        // 分别指定 Monitor，互不干扰
        synchronized (monitor1) {
            x -= delta;
            y -= delta;
        }
    }
    ```
  
  - synchronized 的本质
    
    - 保证方法内部或代码块内部资源（数据）的互斥访问，即同一时间，由同一个 Monitor 监视的代码，最多只能有一个线程在访问
    - 保证线程之间对监视资源的数据同步。即任何线程在获取到 Monitor 后的第一时间，会先将共享内存中的数据复制到自己的缓存中；任何线程在释放 Monitor 的第一时间，会先将缓存中的数据复制到共享内存中
  
  - synchronized 锁静态方法和实例方法区别
    
    - synchronized 实例方法锁定的是对象实例，而 synchronized 静态方法锁定的是类的 Class 对象。
    
    - synchronized 实例方法会受到对象实例的限制，同一实例的不同同步方法之间也会相互阻塞。而 synchronized 静态方法对于所有实例对象都是同一把锁，因此同一类的不同实例之间也会相互阻塞。
  
  - 死锁
  
  - 乐观锁和悲观锁
    
    - 乐观锁先不加锁，发现数据被改了才加锁
  
  - volatile
    
    - 保证加了 volatile 关键字的字段的操作具有原子性和同步性。其中原子性相当于实现了针对单一字段的线程间互斥访问。因为 volatile 可以看做事简化版的 synchronized 
    - volatile 只对基本类型（byte、char、short、int、long、float、double、boolean）的赋值操作和对象的引用赋值操作有效。
  
  - java.util.concurrent.atomic 包
    
    - 下面有 AtomicInteger、AtomicBoolean、AtomicReference 等类，作用和 volatile 基本一致，可以看做是通用版的 volatile
      
      ```java
      AtomicInteger count = new AtomicInteger(0);
      count.getAndIncrement();
      ```
  
  - Lock / ReentrantReadWriteLock 读写锁
    
    - 同样是加锁机制，但使用方式更灵活，同时也更麻烦一些
      
      ```java
      Lock lock = new ReentrantLock();
      // ...
      lock.lock();
      try{
          x++;
      } finally {
          // 保证在方法提前结束或出现 Exception 的时候，依然能正常释放锁
          lock.unlock();
      }
      ```
    
    - 一般并不会只是使用 Lock，而是会使用更复杂的锁，例如 ReadWriteLock：
      
      ```java
      ReentrantReadWriteLock lock = new ReentrantReadWriteLock();
      // 读锁
      Lock readLock = lock.readLock();
      // 写锁
      Lock writeLock = lock.writeLock();
      
      private int x = 0;
      
      private void count() {
          writeLock.lock();
          try {
              x++;
          } finally {
              writeLock.unlock();
          }
      }
      
      private void print(int time) {
          readLock.lock();
          try {
            for (int i = 0; i < time; i++) {
                  System.out.print(x + " ");
               }
              System.out.println();
          } finally {
              readLock.unlock();
          }
      }
      ```

- 线程安全问题的本质
  
  在多个线程访问共同的资源时，在某一线程对资源进行写操作的中途，其他线程对这个写了一半的资源进行了读操作，或者基于这个写了一半的资源进行了写操作，导致出现数据错误

- 锁机制的本质
  
  通过对共享资源进行访问限制，让同一时间只有一个线程可以访问资源，保证了数据的准确性

- 不论是线程安全问题，还是针对线程安全问题所衍生出的锁机制，它们的核心都在于共享的资源，而不是某个方法或者某几行代码

#### 生产者消费者模型代码实现

```java
// 使用BlockingQueue或者synchronized关键字来进行同步
public class ProducerConsumerExample {
    public static void main(String[] args) {
        Warehouse warehouse = new Warehouse();
        Thread producerThread = new Thread(new Producer(warehouse));
        Thread consumerThread = new Thread(new Consumer(warehouse));
        producerThread.start();
        consumerThread.start();
    }
}
class Warehouse {
    private static final int CAPACITY = 5;
    private String[] products = new String[CAPACITY];
    private int count = 0;
    public synchronized void produce(String product) {
        try {
            while (count == CAPACITY) {
                wait(); // 仓库已满，等待消费
            }
            products[count++] = product;
            System.out.println("Produced: " + product);
            notify(); // 唤醒消费者
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
    public synchronized void consume() {
        try {
            while (count == 0) {
                wait(); // 仓库为空，等待生产
            }
            String product = products[--count];
            System.out.println("Consumed: " + product);
            notify(); // 唤醒生产者
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
}
class Producer implements Runnable {
    private final Warehouse warehouse;
    public Producer(Warehouse warehouse) {
        this.warehouse = warehouse;
    }
    @Override
    public void run() {
        while (true) {
            warehouse.produce("Product-" + System.currentTimeMillis());
        }
    }
}
class Consumer implements Runnable {
    private final Warehouse warehouse;
    public Consumer(Warehouse warehouse) {
        this.warehouse = warehouse;
    }
    @Override
    public void run() {
        while (true) {
            warehouse.consume();
        }
    }
}
```
