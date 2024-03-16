# 5 wait-notify

## 5.1 原理

![image-20240218下午35341224](../images/concurrency/39.png)

要区分两种状态：`WAITNG`和`BLOCKED`

- Owner线程发现条件不满足，调用`wait`方法，即可进入`WaitSet`变为`WAITING`状态
- `BLOCKED`和`WAITING`的线程都处于阻塞状态，不占用cpu时间片
- `BLOCKED`线程会在Ower线程释放锁是被唤醒
- `WAITING`线程会在Owner线程调用`notify`或`notifyAll`时被唤醒，但被唤醒后并不意味着立刻获得锁，仍需要进入`EntryList`重新竞争

## 5.2 API介绍

- `obj.wait()`让进入object监视器的线程到`WaitSet`中等待
- `obj.notify()`让在object上正在`WaitSet`中等待的线程中挑选一个唤醒
- `obj.notifyAll()`让在object上正在`WaitSet`中等待的全部线程都唤醒

它们都是线程之间进行协作的手段，都属于`Object`类的方法，必须获得此对象的锁，才能调用这几个方法。

### 5.2.1 wait

```java
@Slf4j(topic = "c.WaitNotifyTest")
public class WaitNotifyTest {
    static Object lock = new Object();

    public static void main(String[] args) throws InterruptedException {
        lock.wait(); // 或者调用 lock.notify()
    }
}
```

输出：

```java
Exception in thread "main" java.lang.IllegalMonitorStateException
	at java.lang.Object.wait(Native Method)
	at java.lang.Object.wait(Object.java:502)
	at com.nasuf.concurrency.WaitNotifyTest.main(WaitNotifyTest.java:10)
```

由于主线程没有获得锁就直接调用`wait`方法，所以会直接抛出`IllegalMonitorStateException`，正确的写法是：

```java
@Slf4j(topic = "c.WaitNotifyTest")
public class WaitNotifyTest {
    static Object lock = new Object();

    public static void main(String[] args) throws InterruptedException {
        synchronized (lock) {
            lock.wait();
        }
    }
}
```

这样主线程会一直等待，不会报错。

`obj.wait()`有一个限制实现的等待方法：`obj.wait(long timeout) `，等待时间超时后，自动向下执行：

```java
@Slf4j(topic = "c.WaitNotifyTest")
public class WaitNotifyTest {
    final static Object obj = new Object();

    public static void main(String[] args) {
        new Thread(() -> {
            synchronized (obj) {
                log.debug("Executing now.");
                try {
                    obj.wait(1000);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
                log.debug("Running other codes.");
            }
        }, "t1").start();
    }
}
```

输出为：

```java
18:57:16.821 [t1] c.WaitNotifyTest - Executing now.
18:57:17.828 [t1] c.WaitNotifyTest - Running other codes.
```

如果在等待时限内被提前唤醒，那么设定的等待时限无效：

```java
@Slf4j(topic = "c.WaitNotifyTest")
public class WaitNotifyTest {
    final static Object obj = new Object();

    public static void main(String[] args) {
        new Thread(() -> {
            synchronized (obj) {
                log.debug("Executing now.");
                try {
                    obj.wait(2000); // 等待2s
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
                log.debug("Running other codes.");
            }
        }, "t1").start();

        sleep(1); // 1s后唤醒所有等待在obj上的线程
        log.debug("Notify threads waiting on obj now.");
        synchronized (obj) {
            obj.notifyAll();
        }
    }
}
```

输出：

```java
19:01:06.263 [t1] c.WaitNotifyTest - Executing now.
19:01:07.265 [main] c.WaitNotifyTest - Notify threads waiting on obj now.
19:01:07.266 [t1] c.WaitNotifyTest - Running other codes.
```

其实还有一个`wait(long timeout, int nanos)`方法，源码如下：

```java
public final void wait(long timeout, int nanos) throws InterruptedException {
  if (timeout < 0) {
    throw new IllegalArgumentException("timeout value is negative");
  }

  if (nanos < 0 || nanos > 999999) {
    throw new IllegalArgumentException(
      "nanosecond timeout value out of range");
  }

  if (nanos > 0) {
    timeout++;
  }

  wait(timeout);
}
```

从源码可以看出，该方法并不是精确到`nano`。

### 5.2.2 notify

```java
@Slf4j(topic = "c.WaitNotifyTest")
public class WaitNotifyTest {
    final static Object obj = new Object();

    public static void main(String[] args) {
        new Thread(() -> {
            synchronized (obj) {
                log.debug("Executing now.");
                try {
                    obj.wait();
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
                log.debug("Running other codes.");
            }
        }, "t1").start();

        new Thread(() -> {
            synchronized (obj) {
                log.debug("Executing now.");
                try {
                    obj.wait();
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
                log.debug("Running other codes.");
            }
        }, "t2").start();

        sleep(2);
        log.debug("Notify threads waiting on obj now.");
        synchronized (obj) {
            obj.notify(); // 唤醒等待在obj上的某一个线程
        }
    }
}
```

输出为：

```java
18:49:33.877 [t1] c.WaitNotifyTest - Executing now.
18:49:33.882 [t2] c.WaitNotifyTest - Executing now.
18:49:35.880 [main] c.WaitNotifyTest - Notify threads waiting on obj now.
18:49:35.881 [t1] c.WaitNotifyTest - Running other codes.
```

如果将最后一段代码换为：

```java
synchronized (obj) {
  obj.notifyAll(); // 唤醒等待在obj上的所有线程
}
```

输出变为：

```java
18:52:11.646 [t1] c.WaitNotifyTest - Executing now.
18:52:11.653 [t2] c.WaitNotifyTest - Executing now.
18:52:13.648 [main] c.WaitNotifyTest - Notify threads waiting on obj now.
18:52:13.649 [t2] c.WaitNotifyTest - Running other codes.
18:52:13.649 [t1] c.WaitNotifyTest - Running other codes.
```

## 5.3 wait-notify的正确使用方式

> sleep(long n)和wait(long n)的区别

- `sleep`是`Thread`类的方法，而`wait`是`Object`类的方法
- `sleep`不需要强制和`synchronized`配合使用，但`wait`需要和`synchronized`一起使用
- `sleep`在睡眠的同时，不会释放锁对象，但`wait`在等待的时候会释放锁对象
- 共同点：在进入`sleep`或者`wait`之后，线程状态一致，均为`TIMED_WAITING`状态

```java
@Slf4j(topic = "c.WaitNotifyTest")
public class WaitNotifyTest {
    final static Object obj = new Object();

    public static void main(String[] args) {
        new Thread(() -> {
            synchronized (obj) {
                log.debug("Executing now.");
                try {
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
                log.debug("Running other codes.");
            }
        }, "t1").start();

        sleep(1);
        synchronized (obj) {
            log.debug("Main thread got lock");
        }
    }
}
```

输出如下：

```java
19:21:07.603 [t1] c.WaitNotifyTest - Executing now.
19:21:09.612 [t1] c.WaitNotifyTest - Running other codes.
19:21:09.613 [main] c.WaitNotifyTest - Main thread got lock
```

可见`sleep`并不会释放锁。而如果使用`wait`，就会释放锁：

```java
@Slf4j(topic = "c.WaitNotifyTest")
public class WaitNotifyTest {
    final static Object obj = new Object();

    public static void main(String[] args) {
        new Thread(() -> {
            synchronized (obj) {
                log.debug("Executing now.");
                try {
                    obj.wait();
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
                log.debug("Running other codes.");
            }
        }, "t1").start();

        sleep(1);
        synchronized (obj) {
            log.debug("Main thread got lock");
        }
    }
}
```

输出如下：

```java
19:23:43.083 [t1] c.WaitNotifyTest - Executing now.
19:23:44.081 [main] c.WaitNotifyTest - Main thread got lock
```

### 5.3.1 代码优化 step1

```java
@Slf4j(topic = "c.WaitNotifyTest")
public class WaitNotifyTest {
    static final Object room = new Object();
    static boolean hasCigarette = false;
    static boolean hasTakeout = false;

    public static void main(String[] args) {
        new Thread(() -> {
            synchronized (room) {
                log.debug("有烟没？[{}]", hasCigarette);
                if (!hasCigarette) {
                    log.debug("没烟，歇会儿！");
                    sleep(2);
                }
                log.debug("有烟没？[{}]", hasCigarette);
                if (hasCigarette) {
                    log.debug("可以开始干活了！");
                }
            }
        }, "PersonA").start();

        for (int i = 0; i < 5; i++) {
            new Thread(() -> {
                synchronized (room) {
                    log.debug("可以开始干活了！");
                }
            }, "OtherPerson").start();
        }

        sleep(1);
        new Thread(() -> {
            hasCigarette = true;
            log.debug("烟送到了！");
        }, "ManA").start();
    }
}
```

输出如下：

```java
10:31:34.848 [PersonA] c.WaitNotifyTest - 有烟没？[false]
10:31:34.855 [PersonA] c.WaitNotifyTest - 没烟，歇会儿！
10:31:35.855 [ManA] c.WaitNotifyTest - 烟送到了！
10:31:36.857 [PersonA] c.WaitNotifyTest - 有烟没？[true]
10:31:36.858 [PersonA] c.WaitNotifyTest - 可以开始干活了！
10:31:36.858 [OtherPerson] c.WaitNotifyTest - 可以开始干活了！
10:31:36.859 [OtherPerson] c.WaitNotifyTest - 可以开始干活了！
10:31:36.859 [OtherPerson] c.WaitNotifyTest - 可以开始干活了！
10:31:36.859 [OtherPerson] c.WaitNotifyTest - 可以开始干活了！
10:31:36.859 [OtherPerson] c.WaitNotifyTest - 可以开始干活了！
```

代码分析：

- ManA线程在1s后将烟送到，但是PersonA要sleep 2s
- PersonA sleep时，不会释放对象锁，其他五个线程无法工作

而如果把ManA线程代码修改如下：

```java
sleep(1);
new Thread(() -> {
  synchronized (room) {
    hasCigarette = true;
    log.debug("烟送到了！");
  }
}, "ManA").start();
```

依然无法达到效果，因为ManA线程无法获得room锁，直到PersonA线程执行完毕。输出如下：

```java
10:40:42.308 [PersonA] c.WaitNotifyTest - 有烟没？[false]
10:40:42.313 [PersonA] c.WaitNotifyTest - 没烟，歇会儿！
10:40:44.318 [PersonA] c.WaitNotifyTest - 有烟没？[false]
10:40:44.320 [ManA] c.WaitNotifyTest - 烟送到了！
10:40:44.320 [OtherPerson] c.WaitNotifyTest - 可以开始干活了！
10:40:44.321 [OtherPerson] c.WaitNotifyTest - 可以开始干活了！
10:40:44.321 [OtherPerson] c.WaitNotifyTest - 可以开始干活了！
10:40:44.321 [OtherPerson] c.WaitNotifyTest - 可以开始干活了！
10:40:44.321 [OtherPerson] c.WaitNotifyTest - 可以开始干活了！
```

### 5.3.2 代码优化 step2

```java
@Slf4j(topic = "c.WaitNotifyTest")
public class WaitNotifyTest {
    static final Object room = new Object();
    static boolean hasCigarette = false;
    static boolean hasTakeout = false;

    public static void main(String[] args) {
        new Thread(() -> {
            synchronized (room) {
                log.debug("有烟没？[{}]", hasCigarette);
                if (!hasCigarette) {
                    log.debug("没烟，歇会儿！");
                    try {
                        room.wait(); // 替换sleep为wait方法
                    } catch (InterruptedException e) {
                        throw new RuntimeException(e);
                    }
                }
                log.debug("有烟没？[{}]", hasCigarette);
                if (hasCigarette) {
                    log.debug("可以开始干活了！");
                }
            }
        }, "PersonA").start();

        for (int i = 0; i < 5; i++) {
            new Thread(() -> {
                synchronized (room) {
                    log.debug("可以开始干活了！");
                }
            }, "OtherPerson").start();
        }

        sleep(1);
        new Thread(() -> {
            synchronized (room) {
                hasCigarette = true;
                log.debug("烟送到了！");
                room.notify(); // 烟送到后，notify等待在room锁上的线程
            }
        }, "ManA").start();
    }
}
```

输出如下：

```java
10:43:58.899 [PersonA] c.WaitNotifyTest - 有烟没？[false]
10:43:58.906 [PersonA] c.WaitNotifyTest - 没烟，歇会儿！
10:43:58.907 [OtherPerson] c.WaitNotifyTest - 可以开始干活了！
10:43:58.907 [OtherPerson] c.WaitNotifyTest - 可以开始干活了！
10:43:58.908 [OtherPerson] c.WaitNotifyTest - 可以开始干活了！
10:43:58.908 [OtherPerson] c.WaitNotifyTest - 可以开始干活了！
10:43:58.908 [OtherPerson] c.WaitNotifyTest - 可以开始干活了！
10:43:59.905 [ManA] c.WaitNotifyTest - 烟送到了！
10:43:59.906 [PersonA] c.WaitNotifyTest - 有烟没？[true]
10:43:59.906 [PersonA] c.WaitNotifyTest - 可以开始干活了！
```

代码分析：

- PersonA线程wait之后，会释放room对象锁，OtherPerson线程可以拿到锁继续执行自己的代码
- ManA线程notify等待在room锁对象上的线程后，PersonA可以继续执行自己的任务代码
- 但是，如果同时有其他线程也等待在room对象锁上，ManA线程执行notify后，有可能会错误唤醒了其他线程，而非PersonA线程。参考下面的章节代码

### 5.3.3 代码优化 step3

```java
@Slf4j(topic = "c.WaitNotifyTest")
public class WaitNotifyTest {
    static final Object room = new Object();
    static boolean hasCigarette = false;
    static boolean hasTakeout = false;

    public static void main(String[] args) {
        new Thread(() -> {
            synchronized (room) {
                log.debug("有烟没？[{}]", hasCigarette);
                if (!hasCigarette) {
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
                if (!hasTakeout) {
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
                room.notify(); // 虚假唤醒
            }
        }, "ManA").start();
    }
}
```

输出如下：

```java
10:54:16.103 [PersonA] c.WaitNotifyTest - 有烟没？[false]
10:54:16.110 [PersonA] c.WaitNotifyTest - 没烟，歇会儿！
10:54:16.110 [PersonB] c.WaitNotifyTest - 外卖送到没？[false]
10:54:16.110 [PersonB] c.WaitNotifyTest - 没外卖，歇会儿！
10:54:17.107 [ManA] c.WaitNotifyTest - 外卖送到了！
10:54:17.108 [PersonA] c.WaitNotifyTest - 有烟没？[false]
10:54:17.108 [PersonA] c.WaitNotifyTest - 没干成活...
```

代码分析：

- ManA线程随机唤醒了等待在room对象锁上的线程PersonA，而非期望的PersonB线程。导致PersonA线程并没有完成任务代码

解决方案：

```java
new Thread(() -> {
  synchronized (room) {
    hasTakeout = true;
    log.debug("外卖送到了！");
    room.notifyAll(); // 唤醒所有等待在room对象锁上的线程
  }
}, "ManA").start();
```

输出如下：

```java
11:00:45.007 [PersonA] c.WaitNotifyTest - 有烟没？[false]
11:00:45.014 [PersonA] c.WaitNotifyTest - 没烟，歇会儿！
11:00:45.014 [PersonB] c.WaitNotifyTest - 外卖送到没？[false]
11:00:45.014 [PersonB] c.WaitNotifyTest - 没外卖，歇会儿！
11:00:46.007 [ManA] c.WaitNotifyTest - 外卖送到了！
11:00:46.007 [PersonB] c.WaitNotifyTest - 外卖送到没？[true]
11:00:46.007 [PersonB] c.WaitNotifyTest - 可以开始干活了！
11:00:46.007 [PersonA] c.WaitNotifyTest - 有烟没？[false]
11:00:46.007 [PersonA] c.WaitNotifyTest - 没干成活...
```

这时可以唤醒PersonB，但是PersonA线程也被唤醒错误唤醒，无法正确执行任务代码

### 5.3.4 代码优化 step4

```java
@Slf4j(topic = "c.WaitNotifyTest")
public class WaitNotifyTest {
    static final Object room = new Object();
    static boolean hasCigarette = false;
    static boolean hasTakeout = false;

    public static void main(String[] args) {
        new Thread(() -> {
            synchronized (room) {
                log.debug("有烟没？[{}]", hasCigarette);
                while (!hasCigarette) {  // 将if替换为while循环
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
                while (!hasTakeout) {  // 将if替换为while循环
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

输出如下：

```java
11:05:48.898 [PersonA] c.WaitNotifyTest - 有烟没？[false]
11:05:48.904 [PersonA] c.WaitNotifyTest - 没烟，歇会儿！
11:05:48.904 [PersonB] c.WaitNotifyTest - 外卖送到没？[false]
11:05:48.904 [PersonB] c.WaitNotifyTest - 没外卖，歇会儿！
11:05:49.902 [ManA] c.WaitNotifyTest - 外卖送到了！
11:05:49.902 [PersonB] c.WaitNotifyTest - 外卖送到没？[true]
11:05:49.903 [PersonB] c.WaitNotifyTest - 可以开始干活了！
11:05:49.903 [PersonA] c.WaitNotifyTest - 没烟，歇会儿！
```

将if替换为while循环，这样将一次性的判断改为了循环判断，就不会出现被错误唤醒后而任务代码没有执行成功的情况。大致的模式如下：

```java
synchronized(lock) {
  while (条件不成立) {
    lock.wait()
  }
  // Other task.
}

// 另一个线程
synchronized(lock) {
  lock.notifyAll();
}
```

## 5.3 同步模式之保护性暂停

即`Guarded Suspension`，用在一个线程等待另一个线程的执行结果

要点：

- 有一个结果需要从一个线程传递到另一个线程，让它们关联同一个`GuardedObject`
- 如果有结果不断从一个线程到另一个线程，那么可以使用消息队列（生产者/消费者）
- JDK中，join的视线、Future的视线，采用的就是此模式
- 因为要等待另一方的结果，因此归类到同步模式

<img src="/Users/songtao/Projects/nasuf.github.io/src/images/concurrency/40.png" alt="image-20240224上午111412534" style="zoom:50%;" />

案例代码：

```java
@Slf4j(topic = "c.GuardedObject")
public class TestGuard {
    // 线程1等待线程2的下载结果
    public static void main(String[] args) {
        GuardedObject guardedObject = new GuardedObject();
        new Thread(() -> {
            log.debug("等待结果");
            List<String> list = (List<String>) guardedObject.get();
            log.debug("结果大小：{}", list.size());
        }, "t1").start();

        new Thread(() -> {
            log.debug("执行下载");
            try {
                List<String> list = Downloader.download();
                guardedObject.complete(list);
            } catch (IOException e) {
                throw new RuntimeException(e);
            }
        }, "t2").start();
    }
}

class GuardedObject {
    private Object response;

    public Object get() {
        synchronized (this) {
            while (response == null) {
                try {
                    this.wait();
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
            return response;
        }
    }

    public void complete(Object response) {
        synchronized (this) {
            this.response = response;
            this.notifyAll();
        }
    }
}

class Downloader {
    public static List<String> download() throws IOException {
        HttpURLConnection conn = (HttpURLConnection) new URL("https://www.baidu.com").openConnection();
        List<String> lines = new ArrayList<>();
        try (BufferedReader reader =
                     new BufferedReader(new InputStreamReader(conn.getInputStream(), StandardCharsets.UTF_8))) {
            String line;
            while ((line = reader.readLine()) != null) {
                lines.add(line);
            }
        }
        return lines;
    }
}
```

输出如下：

```java
17:47:38.553 [t2] c.GuardedObject - 执行下载
17:47:38.553 [t1] c.GuardedObject - 等待结果
17:47:39.265 [t1] c.GuardedObject - 结果大小：3
```

下面考虑一种等待超时的优化。假如设定了等待超时时间，代码应该如何设计？
```java
@Slf4j(topic = "c.GuardedObject")
public class TestGuard {
    // 线程1等待线程2的下载结果
    public static void main(String[] args) {
        GuardedObject guardedObject = new GuardedObject();
        new Thread(() -> {
            log.debug("等待结果");
            Object response = guardedObject.get(2000);
            log.debug("结果：{}", response);
        }, "t1").start();

        new Thread(() -> {
            log.debug("执行下载");
            sleep(3);
            guardedObject.complete(new Object());
        }, "t2").start();
    }
}

class GuardedObject {
    private Object response;

    public Object get(long timeout) {
        synchronized (this) {
            // 记录开始/等待时间
            long start = System.currentTimeMillis();
            long passed = 0;
            while (response == null) {
                // 经历的事件超过了最大等待时间，退出循环
                if (passed >= timeout) {
                    break;
                }
                try {
                    // 此处确保了被虚假唤醒后，再次进入等待后，导致等待时间超过timeout时间的问题
                    this.wait(timeout - passed);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
                passed = System.currentTimeMillis() - start;

            }
            return response;
        }
    }

    public void complete(Object response) {
        synchronized (this) {
            this.response = response;
            this.notifyAll();
        }
    }
}
```

上述情况下，超时时间设定为2s，但是模拟下载时间为3s，发生了超时，所以输出如下：

```java
18:03:57.259 [t1] c.GuardedObject - 等待结果
18:03:57.259 [t2] c.GuardedObject - 执行下载
18:03:59.270 [t1] c.GuardedObject - 结果：null
```

另外一种情况是“虚假唤醒”。修改代码如下：

```java
@Slf4j(topic = "c.GuardedObject")
public class TestGuard {
    // 线程1等待线程2的下载结果
    public static void main(String[] args) {
        GuardedObject guardedObject = new GuardedObject();
        new Thread(() -> {
            log.debug("等待结果");
            Object response = guardedObject.get(2000);
            log.debug("结果：{}", response);
        }, "t1").start();

        new Thread(() -> {
            log.debug("执行下载");
            sleep(1);
            guardedObject.complete(null);
        }, "t2").start();
    }
}
```

模拟下载时间1s后，传递null（虚假唤醒），导致线程t1需要继续等待1s后完成，返回null值。输出如下：

```java
18:07:41.223 [t2] c.GuardedObject - 执行下载
18:07:41.223 [t1] c.GuardedObject - 等待结果
18:07:43.233 [t1] c.GuardedObject - 结果：null
```

参考jdk8源码里面对`Thread.join()`方法的设计：

```java
public final synchronized void join(long millis)
  throws InterruptedException {
  long base = System.currentTimeMillis();
  long now = 0;

  if (millis < 0) {
    throw new IllegalArgumentException("timeout value is negative");
  }

  if (millis == 0) {
    while (isAlive()) {
      wait(0);
    }
  } else {
    while (isAlive()) {
      long delay = millis - now;
      if (delay <= 0) {
        break;
      }
      wait(delay);
      now = System.currentTimeMillis() - base;
    }
  }
}
```

### 5.3.1 保护性暂停-扩展（一）

图中Futures就好比居民楼一层的信箱（每个信箱有房间编号），左侧的t0, t2, t4就好比等待邮件的居民，右侧t1, t3, t5就好比邮递员。如果需要在多个类之间使用`GuardedObject`对象，作为参数传递不是很方便，因此设计一个用来解耦的中间类，这样不仅能够解耦“结果等待者”和“结果生产者”，还能够同时支持多个任务的管理

<img src="../images/concurrency/41.png" alt="image-20240302上午93740091" style="zoom:50%;" />

测试代码：

```java
@Slf4j(topic = "c.GuardedObject")
public class TestGuard {
    public static void main(String[] args) {
        for (int i = 0; i < 3; i++) {
            new People().start();
        }
        sleep(1);
        for (Integer id : MailBoxes.getIds()) {
            new Postman(id, "内容 " + id).start();
        }
    }
}

@Slf4j(topic = "c.People")
class People extends Thread {
    @Override
    public void run() {
        GuardedObject guardedObject = MailBoxes.createGuardedObject();
        log.debug("开始收信 {}", guardedObject.getId());
        Object mail = guardedObject.get(5000);
        log.debug("收到信 id: {}, 内容: {}", guardedObject.getId(), mail);
    }
}

@Slf4j(topic = "c.Postman")
class Postman extends Thread {
    private int mailId;
    private String mailContent;

    public Postman(int mailId, String mailContent) {
        this.mailId = mailId;
        this.mailContent = mailContent;
    }

    @Override
    public void run() {
        GuardedObject guardedObject = MailBoxes.getGuardedObject(mailId);
        log.debug("送信 id: {}, 内容: {}", mailId, mailContent);
        guardedObject.complete(mailContent);
    }
}

class MailBoxes {
    private static Map<Integer, GuardedObject> boxes = new ConcurrentHashMap<>();
    private static int id = 1;

    public static synchronized int generateId() {
        return id++;
    }

    public static GuardedObject getGuardedObject(int id) {
        return boxes.remove(id);
    }

    public static GuardedObject createGuardedObject() {
        GuardedObject guardedObject = new GuardedObject(generateId());
        boxes.put(guardedObject.getId(), guardedObject);
        return guardedObject;
    }

    public static Set<Integer> getIds() {
        return boxes.keySet();
    }
}

class GuardedObject {
    // 标识 Guarded Object
    private int id;

    public GuardedObject(int id) {
        this.id = id;
    }

    public int getId() {
        return id;
    }

    private Object response;

    public Object get(long timeout) {
        synchronized (this) {
            long start = System.currentTimeMillis();
            long passed = 0;
            while (response == null) {
                if (passed >= timeout) {
                    break;
                }
                try {
                    this.wait(timeout - passed);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
                passed = System.currentTimeMillis() - start;

            }
            return response;
        }
    }

    public void complete(Object response) {
        synchronized (this) {
            this.response = response;
            this.notifyAll();
        }
    }
}
```

输出：

```java
10:46:23.583 [Thread-0] c.People - 开始收信 3
10:46:23.583 [Thread-2] c.People - 开始收信 2
10:46:23.583 [Thread-1] c.People - 开始收信 1
10:46:24.596 [Thread-4] c.Postman - 送信 id: 2, 内容: 内容 2
10:46:24.596 [Thread-3] c.Postman - 送信 id: 3, 内容: 内容 3
10:46:24.597 [Thread-5] c.Postman - 送信 id: 1, 内容: 内容 1
10:46:24.597 [Thread-2] c.People - 收到信 id: 2, 内容: 内容 2
10:46:24.597 [Thread-0] c.People - 收到信 id: 3, 内容: 内容 3
10:46:24.598 [Thread-1] c.People - 收到信 id: 1, 内容: 内容 1
```

需要注意，这种**保护性暂停模式**，是一一对应的模式，即一个线程产生的结果，需要固定的另一个线程来处理。如果是非一一对应模式，就是下一节的**生产者消费者模式**

## 5.4 异步模式之生产者 / 消费者

> 要点

- 与前面的保护性暂停模式中的`GuardedObject`不同，不需要产生结果和消费结果的线程一一对应
- 消费队列可以用来平衡生产和消费的线程资源
- 生产者仅负责产生结果数据，不关心数据该如何处理；而消费者专心处理结果数据
- 消息队列是有容量限制的，容量满时不会再加入数据，空时不会再消费数据
- JDK中各种阻塞队列，采用的就是这种模式

<img src="../images/concurrency/42.png" alt="image-20240302上午105707664" style="zoom:50%;" />

测试代码：

```java
@Slf4j(topic = "c.Test10")
public class Test10 {
    public static void main(String[] args) {
        MessageQueue queue = new MessageQueue(2);
        for (int i = 0; i < 3; i++) {
            int id = i;
            new Thread(() -> {
                queue.put(new Message(id, "内容: " + id));
            }, "Producer#" + i).start();
        }

        new Thread(() -> {
            while (true) {
                sleep(1);
                queue.take();
            }
        }).start();
    }
}

@Slf4j(topic = "c.MessageQueue")
class MessageQueue {
    private final LinkedList<Message> list = new LinkedList<>();
    private int capacity;

    public MessageQueue(int capacity) {
        this.capacity = capacity;
    }

    public Message take() {
        // 需要先检查队列是否为空
        synchronized (list) {
            while (list.isEmpty()) {
                try {
                    log.debug("队列为空，消费者线程等待");
                    list.wait();
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
            Message message = list.removeFirst();
            log.debug("已消费消息: {}", message);
            // 唤醒等待在list上的生产者线程
            list.notifyAll();
            return message;
        }
    }

    public void put(Message message) {
        // 需要检查队列是否已满
        synchronized (list) {
            while (list.size() == capacity) {
                log.debug("队列已满，生产者线程等待");
                try {
                    list.wait();
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
            log.debug("已生产消息: {}", message);
            list.addLast(message);
            // 唤醒等待在list上的消费者线程
            list.notifyAll();
        }
    }
}

final class Message {
    private int id;
    private Object value;

    public Message(int id, Object value) {
        this.id = id;
        this.value = value;
    }

    public int getId() {
        return id;
    }

    public Object getValue() {
        return value;
    }

    @Override
    public String toString() {
        return "Message{" +
                "id=" + id +
                ", value=" + value +
                '}';
    }
}
```

输出如下：

```java
11:23:54.276 [Producer#0] c.MessageQueue - 已生产消息: Message{id=0, value=内容: 0}
11:23:54.285 [Producer#1] c.MessageQueue - 已生产消息: Message{id=1, value=内容: 1}
11:23:54.285 [Producer#2] c.MessageQueue - 队列已满，生产者线程等待
11:23:55.280 [Thread-0] c.MessageQueue - 已消费消息: Message{id=0, value=内容: 0}
11:23:55.282 [Producer#2] c.MessageQueue - 已生产消息: Message{id=2, value=内容: 2}
11:23:56.282 [Thread-0] c.MessageQueue - 已消费消息: Message{id=1, value=内容: 1}
11:23:57.288 [Thread-0] c.MessageQueue - 已消费消息: Message{id=2, value=内容: 2}
11:23:58.290 [Thread-0] c.MessageQueue - 队列为空，消费者线程等待
```
