# 9 ReentrantLock

相对于`synchronized`，都支持可重入，但是它另外具备如下特点：

- 可中断
- 可以设置超时时间
- 可以设置为公平锁
- 支持多个条件变量

基本语法：

```java
reentrantLock.lock();
try {
  // 临界区
} finally {
  // 释放锁
  reentrantLock.unlock();
}
```



## 9.1 可重入

可重入是指同一个线程如果首次获得了这把锁，那么因为它是这把锁的拥有者，因此有权利再次获取这把锁。如果是不可重入锁，那么第二次获得锁时，自己也会被锁挡住

测试代码：

```java
@Slf4j(topic = "c.ReentrantLockTest")
public class ReentrantLockTest {
    private static ReentrantLock lock = new ReentrantLock();

    public static void main(String[] args) {
        lock.lock();
        try {
            log.debug("main method");
            m1();
        } finally {
            lock.unlock();
        }
    }

    public static void m1() {
        lock.lock();
        try {
            log.debug("m1 method");
            m2();
        } finally {
            lock.unlock();
        }
    }

    public static void m2() {
        lock.lock();
        try {
            log.debug("m2 method");
        } finally {
            lock.unlock();
        }
    }
}
```

输出：

```java
20:12:40.271 [main] c.ReentrantLockTest - main method
20:12:40.278 [main] c.ReentrantLockTest - m1 method
20:12:40.278 [main] c.ReentrantLockTest - m2 method
```

## 9.2 可打断

首先，如果主线程首先获得了锁，那么线程t1就无法获得锁：

```java
@Slf4j(topic = "c.ReentrantLockTest2")
public class ReentrantLockTest2 {
    private static ReentrantLock lock = new ReentrantLock();

    public static void main(String[] args) {
        Thread t1 = new Thread(() -> {
            try {
                // 如果没有竞争，那么此方法就会获取lock对象锁
                // 如果有竞争，就进入阻塞队列，可以被其他线程用interrupt方法去打断
                log.debug("try to get lock");
                lock.lockInterruptibly();
            } catch (InterruptedException e) {
                e.printStackTrace();
                log.debug("lock is not acquired.");
                return;
            }
            try {
                log.debug("lock is acquired.");
            } finally {
                lock.unlock();
            }
        }, "t1");

        lock.lock();
        t1.start();
    }
}
```

输出如下，且一直阻塞：

```java
20:22:25.326 [t1] c.ReentrantLockTest2 - try to get lock
```

此时主线程可以打断正在阻塞的t1线程（加锁时使用`lockInterruptibly`）：

```java
@Slf4j(topic = "c.ReentrantLockTest2")
public class ReentrantLockTest2 {
    private static ReentrantLock lock = new ReentrantLock();

    public static void main(String[] args) {
        Thread t1 = new Thread(() -> {
            try {
                // 如果没有竞争，那么此方法就会获取lock对象锁
                // 如果有竞争，就进入阻塞队列，可以被其他线程用interrupt方法去打断
                log.debug("try to get lock");
                lock.lockInterruptibly(); // 注意，要使用lockInterruptibly()
            } catch (InterruptedException e) {
                e.printStackTrace();
                log.debug("lock is not acquired.");
                return;
            }
            try {
                log.debug("lock is acquired.");
            } finally {
                lock.unlock();
            }
        }, "t1");

        lock.lock();
        t1.start();

        sleep(1);
        log.debug("main thread to interrupt t1 thread");
        t1.interrupt();
    }
}
```

输出如下：

```java
20:25:47.093 [t1] c.ReentrantLockTest2 - try to get lock
20:25:48.091 [main] c.ReentrantLockTest2 - main thread to interrupt t1 thread
20:25:48.092 [t1] c.ReentrantLockTest2 - lock is not acquired.
java.lang.InterruptedException
	at java.util.concurrent.locks.AbstractQueuedSynchronizer.doAcquireInterruptibly(AbstractQueuedSynchronizer.java:898)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer.acquireInterruptibly(AbstractQueuedSynchronizer.java:1222)
	at java.util.concurrent.locks.ReentrantLock.lockInterruptibly(ReentrantLock.java:335)
	at com.nasuf.concurrency.ReentrantLockTest2.lambda$main$0(ReentrantLockTest2.java:19)
	at java.lang.Thread.run(Thread.java:750)

```

而如果使用`lock.lock()`则主线程无法打断它：

```java
@Slf4j(topic = "c.ReentrantLockTest2")
public class ReentrantLockTest2 {
    private static ReentrantLock lock = new ReentrantLock();

    public static void main(String[] args) {
        Thread t1 = new Thread(() -> {
            try {
                log.debug("try to get lock");
                lock.lock();
                log.debug("lock is acquired.");
            } finally {
                lock.unlock();
            }
        }, "t1");

        lock.lock();
        t1.start();

        sleep(1);
        log.debug("main thread to interrupt t1 thread");
        t1.interrupt();
    }
}
```

输出如下且一直阻塞：

```java
20:28:21.188 [t1] c.ReentrantLockTest2 - try to get lock
20:28:22.191 [main] c.ReentrantLockTest2 - main thread to interrupt t1 thread
```

## 9.3 锁超时

使用`lock.tryLock()`尝试获取锁，返回是否获得锁的结果：

```java
@Slf4j(topic = "c.ReentrantLockTest3")
public class ReentrantLockTest3 {
    private static ReentrantLock lock = new ReentrantLock();

    public static void main(String[] args) {
        Thread t1 = new Thread(() -> {
            log.debug("try to acquire lock");
            if (!lock.tryLock()) {
                log.debug("can't acquire lock");
                return;
            }
            try {
                log.debug("lock acquired.");
            } finally {
                lock.unlock();
            }
        }, "t1");
        t1.start();
    }
}
```

输出：

```java
19:10:58.513 [t1] c.ReentrantLockTest3 - try to acquire lock
19:10:58.517 [t1] c.ReentrantLockTest3 - lock acquired.
```

如果主线程优先获得锁，那么`tryLock()`立即返回false：

```java
@Slf4j(topic = "c.ReentrantLockTest3")
public class ReentrantLockTest3 {
    private static ReentrantLock lock = new ReentrantLock();

    public static void main(String[] args) {
        Thread t1 = new Thread(() -> {
            log.debug("try to acquire lock");
            if (!lock.tryLock()) {
                log.debug("can't acquire lock");
                return;
            }
            try {
                log.debug("lock acquired.");
            } finally {
                lock.unlock();
            }
        }, "t1");

        lock.lock(); // 主线程先获得锁
        t1.start();
    }
}
```

输出：

```java
19:12:33.080 [t1] c.ReentrantLockTest3 - try to acquire lock
19:12:33.084 [t1] c.ReentrantLockTest3 - can't acquire lock
```

另一种使用方式是，加上等待时间：

```java
@Slf4j(topic = "c.ReentrantLockTest3")
public class ReentrantLockTest3 {
    private static ReentrantLock lock = new ReentrantLock();

    public static void main(String[] args) {
        Thread t1 = new Thread(() -> {
            log.debug("try to acquire lock");
            try {
                if (!lock.tryLock(1, TimeUnit.SECONDS)) {
                    log.debug("can't acquire lock");
                    return;
                }
            } catch (InterruptedException e) {
                log.debug("can't acquire lock");
                return;
            }
            try {
                log.debug("lock acquired.");
            } finally {
                lock.unlock();
            }
        }, "t1");

        lock.lock();
        t1.start();
    }
}
```

输出：

```java
19:16:12.832 [t1] c.ReentrantLockTest3 - try to acquire lock
19:16:13.841 [t1] c.ReentrantLockTest3 - can't acquire lock
```

在一秒钟后，t1线程无法获取锁，直接返回。

如果主线程在一秒钟后释放了锁，那么t1线程可以继续获得锁向下执行：

```java
@Slf4j(topic = "c.ReentrantLockTest3")
public class ReentrantLockTest3 {
    private static ReentrantLock lock = new ReentrantLock();

    public static void main(String[] args) {
        Thread t1 = new Thread(() -> {
            log.debug("try to acquire lock");
            try {
                if (!lock.tryLock(2, TimeUnit.SECONDS)) {
                    log.debug("can't acquire lock");
                    return;
                }
            } catch (InterruptedException e) {
                log.debug("can't acquire lock");
                return;
            }
            try {
                log.debug("lock acquired.");
            } finally {
                lock.unlock();
            }
        }, "t1");

        lock.lock();
        log.debug("main thread acquires lock.");
        t1.start();
        sleep(1);
        log.debug("main thread releases lock.");
        lock.unlock();
    }
}
```

输出：

```java
19:19:06.844 [main] c.ReentrantLockTest3 - main thread acquires lock.
19:19:06.846 [t1] c.ReentrantLockTest3 - try to acquire lock
19:19:07.853 [main] c.ReentrantLockTest3 - main thread releases lock.
19:19:07.853 [t1] c.ReentrantLockTest3 - lock acquired.
```

### 9.3.1 解决哲学家就餐问题

在之前的章节我们讨论过哲学家就餐问题，代码容易出现死锁：

```java
@Slf4j(topic = "c.Philosopher")
class Philosopher extends Thread {
    final Chopstick left;
    final Chopstick right;

    public Philosopher(String name, Chopstick left, Chopstick right) {
        super(name);
        this.left = left;
        this.right = right;
    }

    @Override
    public void run() {
        while (true) {
            synchronized (left) {
                synchronized (right) {
                    eat();
                }
            }
        }
    }

    private void eat() {
        log.debug("eating...");
        ThreadUtils.sleep(1);
    }
}
```

现在我们尝试使用`ReentrantLock`来代替`synchronized`来解决死锁问题，完整代码如下：

```java
@Slf4j(topic = "c.TestDeadLock")
public class TestDeadLock {
    public static void main(String[] args) {
        Chopstick c1 = new Chopstick("1");
        Chopstick c2 = new Chopstick("2");
        Chopstick c3 = new Chopstick("3");
        Chopstick c4 = new Chopstick("4");
        Chopstick c5 = new Chopstick("5");
        new Philosopher("苏格拉底", c1, c2).start();
        new Philosopher("柏拉图", c2, c3).start();
        new Philosopher("亚里士多德", c3, c4).start();
        new Philosopher("赫拉克利特", c4, c5).start();
        new Philosopher("阿基米德", c5, c1).start();
    }
}

@Slf4j(topic = "c.Philosopher")
class Philosopher extends Thread {
    final Chopstick left;
    final Chopstick right;

    public Philosopher(String name, Chopstick left, Chopstick right) {
        super(name);
        this.left = left;
        this.right = right;
    }

    @Override
    public void run() {
        while (true) {
            if (left.tryLock()) {
                try {
                    if (right.tryLock()) {
                        try {
                            eat();
                        } finally {
                            right.unlock();
                        }
                    }
                } finally {
                    left.unlock();
                }
            }
        }
    }

    private void eat() {
        log.debug("eating...");
        ThreadUtils.sleep(1);
    }
}

class Chopstick extends ReentrantLock {
    String name;

    public Chopstick(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "Chopstick{" +
                "name='" + name + '\'' +
                '}';
    }
}
```

如上使用`tryLock()`可以解决死锁问题

## 9.4 公平锁

ReentrantLock默认是不公平的，源码如下：

```java
    /**
     * Creates an instance of {@code ReentrantLock}.
     * This is equivalent to using {@code ReentrantLock(false)}.
     */
    public ReentrantLock() {
        sync = new NonfairSync();
    }

    /**
     * Creates an instance of {@code ReentrantLock} with the
     * given fairness policy.
     *
     * @param fair {@code true} if this lock should use a fair ordering policy
     */
    public ReentrantLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();
    }
```

公平锁一般没有必要，会降低并发度。

## 9.5 条件变量

`synchronized`中也有条件变量，就是我们讲原理章节那个`waitSet`休息室，当条件不满足时进入`waitSet`等待。`ReentrantLock`的条件变量比`synchronized`强大之处在于，它是支持多个条件变量的。就好比：

- `synchronized`是那些不满足条件的线程都在一间休息室里等待消息
- `ReentrantLock`支持多间休息室，有专门等烟的休息室，有专门等早餐的休息室；被唤醒时也是按照休息室来唤醒

>  使用流程

- `await`前需要获得锁
- `await`执行后，会释放锁，进入`conditionObject`等待
- `await`的线程被唤醒（或打断、或超时）去重新竞争锁
- 竞争锁成功后，从`await`后继续执行

考虑之前我们测试`wait-notify`使用的一个例子：

```java
@Slf4j(topic = "c.ReentrantLockTest4")
public class ReentrantLockTest4 {
    static final Object room = new Object();
    static boolean hasCigarette = false;
    static boolean hasTakeout = false;

    public static void main(String[] args) {
        new Thread(() -> {
            synchronized (room) {
                log.debug("有烟没？[{}]", hasCigarette);
                while (!hasCigarette) {
                    log.debug("没烟，歇会儿！");
                    try {
                        room.wait();
                    } catch (InterruptedException e) {
                        throw new RuntimeException(e);
                    }
                }
                log.debug("有烟没？[{}]", hasCigarette);
                if (hasCigarette) {
                    log.debug("可以开始干活了！");
                } else {
                    log.debug("没干成活...");
                }
            }
        }, "PersonA").start();

        new Thread(() -> {
            synchronized (room) {
                log.debug("外卖送到没？[{}]", hasTakeout);
                while (!hasTakeout) {
                    log.debug("没外卖，歇会儿！");
                    try {
                        room.wait();
                    } catch (InterruptedException e) {
                        throw new RuntimeException(e);
                    }
                }
                log.debug("外卖送到没？[{}]", hasTakeout);
                if (hasTakeout) {
                    log.debug("可以开始干活了！");
                } else {
                    log.debug("没干成活...");
                }
            }
        }, "PersonB").start();

        sleep(1);
        new Thread(() -> {
            synchronized (room) {
                hasTakeout = true;
                log.debug("外卖送到了！");
                room.notifyAll();
            }
        }, "ManA").start();
    }
}
```

我们使用`condition`来优化上述代码：

```java
@Slf4j(topic = "c.ReentrantLockTest4")
public class ReentrantLockTest4 {
    static boolean hasCigarette = false;
    static boolean hasTakeout = false;
    static ReentrantLock ROOM = new ReentrantLock();
    static Condition waitCigratteSet = ROOM.newCondition();
    static Condition waitTakeoutSet = ROOM.newCondition();

    public static void main(String[] args) {
        new Thread(() -> {
            ROOM.lock();
            try {
                log.debug("有烟没？[{}]", hasCigarette);
                while (!hasCigarette) {
                    log.debug("没烟，歇会儿！");
                    try {
                        waitCigratteSet.await();
                    } catch (InterruptedException e) {
                        throw new RuntimeException(e);
                    }
                }
                log.debug("可以开始干活了！");
            } finally {
                ROOM.unlock();
            }
        }, "PersonA").start();

        new Thread(() -> {
            ROOM.lock();
            try {
                log.debug("外卖送到没？[{}]", hasTakeout);
                while (!hasTakeout) {
                    log.debug("没外卖，歇会儿！");
                    try {
                        waitTakeoutSet.await();
                    } catch (InterruptedException e) {
                        throw new RuntimeException(e);
                    }
                }
                log.debug("可以开始干活了！");
            } finally {
                ROOM.unlock();
            }
        }, "PersonB").start();

        sleep(1);
        new Thread(() -> {
            ROOM.lock();
            try {
                hasTakeout = true;
                waitTakeoutSet.signal();
            } finally {
                ROOM.unlock();
            }
        }, "ManA").start();

        sleep(1);
        new Thread(() -> {
            ROOM.lock();
            try {
                hasCigarette = true;
                waitCigratteSet.signal();
            } finally {
                ROOM.unlock();
            }
        }, "ManB").start();
    }
}
```

输出如下：

```java
12:43:34.777 [PersonA] c.ReentrantLockTest4 - 有烟没？[false]
12:43:34.785 [PersonA] c.ReentrantLockTest4 - 没烟，歇会儿！
12:43:34.785 [PersonB] c.ReentrantLockTest4 - 外卖送到没？[false]
12:43:34.785 [PersonB] c.ReentrantLockTest4 - 没外卖，歇会儿！
12:43:35.780 [PersonB] c.ReentrantLockTest4 - 可以开始干活了！
12:43:36.786 [PersonA] c.ReentrantLockTest4 - 可以开始干活了！
```

## 9.6 同步模式顺序控制

### 9.6.1 固定顺序

**要求：先输出`2`再输出`1**`

#### 9.6.1.1 wait-notify 方式

```java
@Slf4j(topic = "c.OrderControlTest1")
public class OrderControlTest1 {
    static final Object lock = new Object();
    static boolean t2Executed = false;

    public static void main(String[] args) {
        Thread t1 = new Thread(() -> {
            synchronized (lock) {
                while (!t2Executed) {
                    try {
                        lock.wait();
                    } catch (InterruptedException e) {
                        throw new RuntimeException(e);
                    }
                }
                log.debug("1");
            }
        }, "t1");

        Thread t2 = new Thread(() -> {
            synchronized (lock) {
                log.debug("2");
                t2Executed = true;
                lock.notify();
            }
        }, "t2");

        t1.start();
        t2.start();
    }
}
```

输出：

```java
11:12:52.813 [t2] c.OrderControlTest1 - 2
11:12:52.818 [t1] c.OrderControlTest1 - 1
```

#### 9.6.1.2 ReentrantLock & Condition 方式

```java
@Slf4j(topic = "c.OrderControlTest2")
public class OrderControlTest2 {

    static final ReentrantLock lock = new ReentrantLock();
    static boolean t2Executed = false;

    public static void main(String[] args) {
        Condition condition = lock.newCondition();

        Thread t1 = new Thread(() -> {
            lock.lock();
            try {
                while (!t2Executed) {
                    try {
                        condition.await();
                    } catch (InterruptedException e) {
                        throw new RuntimeException(e);
                    }
                }
                log.debug("1");
            } finally {
                lock.unlock();
            }
        }, "t1");

        Thread t2 = new Thread(() -> {
            lock.lock();
            try {
                t2Executed = true;
                log.debug("2");
                condition.signal();
            } finally {
                lock.unlock();
            }
        }, "t2");

        t1.start();
        t2.start();
    }
}
```

输出：

```java
11:22:45.766 [t2] c.OrderControlTest2 - 2
11:22:45.771 [t1] c.OrderControlTest2 - 1
```

#### 9.6.1.3 park & unpart 方式

```java
@Slf4j(topic = "c.TestOrderControlTest3")
public class TestOrderControlTest3 {
    public static void main(String[] args) {
        Thread t1 = new Thread(() -> {
            LockSupport.park();
            log.debug("1");
        }, "t1");
        t1.start();

        new Thread(() -> {
            log.debug("2");
            LockSupport.unpark(t1);
        }, "t2").start();
    }
}
```

输出：

```java
12:58:35.637 [t2] c.TestOrderControlTest3 - 2
12:58:35.639 [t1] c.TestOrderControlTest3 - 1
```

