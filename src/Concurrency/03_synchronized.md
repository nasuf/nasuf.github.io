# 3. Synchronized

## 3.1 共享带来的问题

> Java的体现

示例代码：

```java
@Slf4j(topic = "c.Test9")
public class Test9 {
    static int counter = 0;
    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(() -> {
            for (int i = 0; i < 5000; i++) {
                counter++;
            }
        }, "t1");
        Thread t2 = new Thread(() -> {
            for (int i = 0; i < 5000; i++) {
                counter--;
            }
        }, "t2");

        t1.start();
        t2.start();
        t1.join();
        t2.join();
        log.debug("counter: {}", counter);
    }
}
```

输出：

```java
18:52:10.045 [main] c.Test9 - counter: -289
```

> 问题分析

以上的结果可能是正数、负数或者零。因为Java中对静态变量的自增、自减并不是原子操作。要彻底理解必须从字节码开始分析。

例如对于`i++`操作而言（i为静态变量），实际会产生如下的jvm字节码指令：

```java
getstatic i // 获取静态变量i的值
iconst_1    // 准备常量1
iadd        // 自增
putstatic i // 将修改后的值存入静态变量i
```

而对应`i--`操作，也是类似：

```java
getstatic i // 获取静态变量i的值
iconst_1    // 准备常量1
isub        // 自减
putstatic   // 将修改后的值存入静态变量i
```

而java的内存模型如下，完成静态变量的自增，自减需要在主存和工作内存中进行数据交换：

![image-20240128下午70017827](../images/concurrency/8.png)

如果是单线程，以上8行代码是按顺序执行（不会交错），便没有问题：

<img src="../images/concurrency/9.png" alt="image-20240128下午70847219" style="zoom:50%;" />

但如果是多线程，这8行代码可能出现交错运行。

出现负数的情况：

<img src="../images/concurrency/10.png" alt="image-20240128下午71006667" style="zoom:50%;" />

出现正数的情况：

<img src="../images/concurrency/11.png" alt="image-20240128下午71112804" style="zoom:50%;" />

> 临界区 Critical Section

- 一个程序运行多个线程本身是没有问题的

- 问题出现在多个线程访问共享资源

  - 多个线程读取共享资源其实也没有问题
  - 在多个线程对共享资源读写操作发生指令交错，就会出现问题

- 一段代码块内如果存在对共享资源的多线程读写操作，称这块代码为临界区，例如：

  ```java
  static int counter = 0;
  
  static void increment()
  // 临界区
  {
    counter++;
  }
  
  static void decrement()
  // 临界区
  {
    counter--;
  }
  ```

> 竞态条件 Race Condition

多个线程在临界区内执行，由于代码的执行序列不同而导致结果无法预测，称之为发生了竞态条件

## 3.2 synchronized 解决方案

为了避免临界区的竞态条件发生，有多钟手段可以达到目的。

- 阻塞式解决方案：synchronize, Lock
- 非阻塞式解决方案：原子变量

本章节使用阻塞式解决方案synchronized，来解决上述问题，即俗称的“锁对象”。它采用互斥的方式让同一时刻至多只有一个线程能持有“锁对象”，其他线程再想获取这个锁对象时就会阻塞住。这样就能保证有锁的线程可以安全地执行临界区内的代码，不用担心线程上线文切换

注意，虽然java中互斥和同步都可以采用synchronized关键字来完成，但他们还是有区别的：

- 互斥是保证临界区的竞态条件发生，同一时刻只能有一个线程执行临界区代码
- 同步是由于线程执行的先后、顺序不同，需要一个线程等待其他线程运行到某个点

### 3.2.1 synchronized

语法：

```java
synchronized(对象) {
  // 临界区
}
```

示例代码：

```java
@Slf4j(topic = "c.Test9")
public class Test9 {
    static int counter = 0;
    static final Object room = new Object();
    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(() -> {
            for (int i = 0; i < 5000; i++) {
                synchronized (room) {
                    counter++;
                }
            }
        }, "t1");
        Thread t2 = new Thread(() -> {
            for (int i = 0; i < 5000; i++) {
                synchronized (room) {
                    counter--;
                }
            }
        }, "t2");

        t1.start();
        t2.start();
        t1.join();
        t2.join();
        log.debug("counter: {}", counter);
    }
}
```

输出永远为：

```java
19:23:19.116 [main] c.Test9 - counter: 0
```

如何理解`synchronized`的作用？

- `synchronized(对象)`中的对象，可以想象为一个房间，有唯一的入口，房间只能一次进入一人进行计算。线程`t1`和`t2`想象成两个人
- 当线程`t1`执行到`synchronized(room)`时就好比`t1`进入到了这个房间，并锁住了门，拿走了钥匙，在门内执行`i++`代码
- 这时候如果`t2`也运行到了`synchronized(room)`时，它发现门被锁住了，只能在门外等待，发生了上下文切换，阻塞住了
- 这中间即使`t1`的cpu时间片不幸用完了，被踢出了门外（不要错误地理解为锁住了对象就能一直执行下去），这时门还是锁住的，`t1`仍然拿着钥匙，`t2`线程还在阻塞状态进不来，只有下次轮到`t1`自己再次获得时间片时才能开门进入
- 当`t1`执行完`synchronized{...}`块内的代码，这时候才会从房间出来，并解开锁，唤醒`t2`线程把钥匙给他。`t2`线程这时才能进入房间，锁住门拿上钥匙，执行它的`count--`代码

总结一下，`synchronized`实际是用对象锁保证了临界区内代码的原子性，临界区内的代码对外是不可分割的，不会被线程切换所打断。

为了加深理解，思考如下问题：

- 如果把`synchronized(obj)`放在`for`循环外面，如何理解？
  - 这样其实是把整个`for`循环作为一个整体来进行加锁
- 如果`t1 synchronized(obj_1)`，而`t2 synchronzied(obj_2)`会怎样运行？
  - 相当于两个线程进入了不同的房间，无法保证线程安全
- 如果`t1 synchronized(obj)` 而`t2`没有加锁，如何运行？
  - `t2`线程在想访问共享资源时候，无需尝试获取锁，所以依然存在线程安全问题

### 3.2.2 案例：计数（面向对象设计）

```java
@Slf4j(topic = "c.Test9")
public class Test9 {

    public static void main(String[] args) throws InterruptedException {
        Room room = new Room();
        Thread t1 = new Thread(() -> {
            for (int i = 0; i < 5000; i++) {
                room.increment();
            }
        }, "t1");
        Thread t2 = new Thread(() -> {
            for (int i = 0; i < 5000; i++) {
                room.decrement();
            }
        }, "t2");

        t1.start();
        t2.start();
        t1.join();
        t2.join();
        log.debug("counter: {}", room.getCounter());
    }
}

class Room {
    private int counter = 0;

    public void increment() {
        synchronized (this) {
            counter++;
        }
    }

    public void decrement() {
        synchronized (this) {
            counter--;
        }
    }

    public int getCounter() {
        synchronized (this) {
            return counter;
        }
    }
}
```

## 3.3 方法上的synchronized

语法：

```java
class Test {
  public synchronized void test() {
    
  }
}

// 等价于：
class Test {
  public void test() {
    synchronized(this) {
      
    }
  }
}
```

```java
class Test {
  public synchronized static void test() {
    
  }
}

// 等价于：
class Test {
  public static void test() {
    synchronized(Test.class) {
      
    }
  }
}
```

所以上一节案例中的`Room` class可以修改为：

```java
class Room {
    private int counter = 0;

    public synchronized void increment() {
        counter++;
    }

    public synchronized void decrement() {
        counter--;
    }

    public synchronized int getCounter() {
        return counter;
    }
}
```

### 3.3.1 案例：“线程八锁”

> Case 1

```java
@Slf4j(topic = "c.TestLocks")
public class TestLocks {
    public static void main(String[] args) {
        Number n1 = new Number();
        new Thread(n1::a).start();
        new Thread(n1::b).start();
    }
}

@Slf4j(topic = "c.Number")
class Number {
    public synchronized void a() {
        log.debug("1");
    }

    public synchronized void b() {
        log.debug("2");
    }
}
```

说明：两个线程锁住的是同一个对象`n1`，所以输出可能是`1 -> 2`或者`2 -> 1`

> Case 2

```java
@Slf4j(topic = "c.TestLocks")
public class TestLocks {
    public static void main(String[] args) {
        Number n1 = new Number();
        new Thread(n1::a).start();
        new Thread(n1::b).start();
    }
}

@Slf4j(topic = "c.Number")
class Number {
    public synchronized void a() {
        sleep(1); // sleep 1s
        log.debug("1");
    }

    public synchronized void b() {
        log.debug("2");
    }
}
```

说明：输出可能是`(1s) -> 1 -> 2`或者`2 -> (1s) -> 1`

> Case 3

```java
@Slf4j(topic = "c.TestLocks")
public class TestLocks {
    public static void main(String[] args) {
        Number n1 = new Number();
        new Thread(n1::a).start();
        new Thread(n1::b).start();
        new Thread(n1::c).start();
    }
}

@Slf4j(topic = "c.Number")
class Number {
    public synchronized void a() {
        sleep(1);
        log.debug("1");
    }

    public synchronized void b() {
        log.debug("2");
    }

    public void c() {
        log.debug("3");
    }
}
```

说明：输出可能是`3 -> (1s) -> 1 -> 2`或者`2 -> 3 -> (1s) -> 1`或者`3 -> 2 -> (1s) -> 1`

> Case 4

```java
@Slf4j(topic = "c.TestLocks")
public class TestLocks {
    public static void main(String[] args) {
        Number n1 = new Number();
        Number n2 = new Number();
        new Thread(n1::a).start();
        new Thread(n2::b).start();
    }
}

@Slf4j(topic = "c.Number")
class Number {
    public synchronized void a() {
        sleep(1);
        log.debug("1");
    }

    public synchronized void b() {
        log.debug("2");
    }
}
```

说明：两个线程执行时候锁住的对象不相同，不存在互斥情况，所以只会有一种输出情况：`2 -> (1s) -> 1`

> Case 5

```java
@Slf4j(topic = "c.TestLocks")
public class TestLocks {
    public static void main(String[] args) {
        Number n1 = new Number();
        new Thread(Number::a).start();
        new Thread(n1::b).start();
    }
}

@Slf4j(topic = "c.Number")
class Number {
    public static synchronized void a() {
        sleep(1);
        log.debug("1");
    }

    public synchronized void b() {
        log.debug("2");
    }
}
```

说明：由于`a()`方法为`static`，加锁的对象是`Number.class`，与方法`b()`锁住的`this`对象不同，不存在互斥现象，所以只会有一种输出情况：`2 -> (1s) -> 1`

> Case 6

```java
@Slf4j(topic = "c.TestLocks")
public class TestLocks {
    public static void main(String[] args) {
        Number n1 = new Number();
        new Thread(Number::a).start();
        new Thread(Number::b).start();
    }
}

@Slf4j(topic = "c.Number")
class Number {
    public static synchronized void a() {
        sleep(1);
        log.debug("1");
    }

    public static synchronized void b() {
        log.debug("2");
    }
}
```

说明：两个方法锁住的是同一个`Number.class`对象，存在互斥现象，所以输出可能是`(1s) -> 1 -> 2`或者`2 -> (1s) -> 1`

> Case 7

```java
@Slf4j(topic = "c.TestLocks")
public class TestLocks {
    public static void main(String[] args) {
        Number n2 = new Number();
        new Thread(Number::a).start();
        new Thread(n2::b).start();
    }
}

@Slf4j(topic = "c.Number")
class Number {
    public static synchronized void a() {
        sleep(1);
        log.debug("1");
    }

    public synchronized void b() {
        log.debug("2");
    }
}
```

说明：两个方法锁住的不是同一个对象，不存在互斥现象，输出只能是`2 -> (1s) -> 1`

> Case 8

```java
@Slf4j(topic = "c.TestLocks")
public class TestLocks {
    public static void main(String[] args) {
        Number n1 = new Number();
        Number n2 = new Number();
        new Thread(() -> n1.a()).start();
        new Thread(() -> n2.b()).start();
    }
}

@Slf4j(topic = "c.Number")
class Number {
    public static synchronized void a() {
        sleep(1);
        log.debug("1");
    }

    public static synchronized void b() {
        log.debug("2");
    }
}
```

说明：两个方法锁住的是同一个`Number.class`对象，所以输出可能是`(1s) -> 1 -> 2`或者`2 -> (1s) -> 1`

## 3.4 变量的线程安全分析

### 3.4.1 成员变量和静态变量是否线程安全

- 如果它们没有共享，则线程安全
- 如果它们被共享了，根据它们的状态是否能够改变，又分为两种情况
  - 如果只有读操作，则线程安全
  - 如果有读写操作，则这段代码是临界区，需要考虑线程安全问题

### 3.4.2 局部变量是否线程安全

- 局部变量是线程安全的
- 但是局部变量引用的对象则未必安全
  - 如果该对象没有逃离方法的作用域，它是线程安全的
  - 如果该对象逃离方法的作用域，则需要考虑线程安全问题

考虑如下代码：

```java
public static void test() {
  int i = 10;
  i++;
}
```

每个线程调用`test()`方法时局部变量`i`会在每个线程的栈帧内存中被创建多份，因此不存在共享情况：

```java
public static void test();
	descriptor: ()V
  flags: ACC_PUBLIC, ACC_STATIC
  Code:
    stack=1, locals=1, args_size=0
      0: bipush   10
      2: istore_0
      3: iine     0, 1
      6: return
    LineNumberTable:
      line 10: 0
      line 11: 3
      line 12: 6
    LocalVariableTable:
			Start  Length  Slot  Name  Signature
          3       4     0     i  I
```

<img src="../images/concurrency/12.png" alt="image-20240209下午41238494" style="zoom:50%;" />

> 局部变量的引用稍有不同

先看一个成员变量的例子：

```java
public class TestThreadUnsafe {

    static final int THREAD_NUMBER = 2;
    static final int LOOP_NUMBER = 200;

    public static void main(String[] args) {
        ThreadUnsafe threadUnsafe = new ThreadUnsafe();
        for (int i = 0; i < THREAD_NUMBER; i++) {
            new Thread(() -> {
                threadUnsafe.method1(LOOP_NUMBER);
            }, "Thread" + (i + 1)).start();
        }
    }
}

class ThreadUnsafe {
    List<String> list = new ArrayList<>();

    public void method1(int loopNumber) {
        for (int i = 0; i < loopNumber; i++) {
            method2();
            method3();
        }
    }

    private void method2() {
        list.add("1");
    }

    private void method3() {
        list.remove(0);
    }
}
```

输出很容易报错：

```java
Exception in thread "Thread1" java.lang.IndexOutOfBoundsException: Index: 0, Size: 0
	at java.util.ArrayList.rangeCheck(ArrayList.java:659)
	at java.util.ArrayList.remove(ArrayList.java:498)
	at com.nasuf.concurrency.ThreadUnsafe.method3(TestThreadUnsafe.java:36)
	at com.nasuf.concurrency.ThreadUnsafe.method1(TestThreadUnsafe.java:27)
	at com.nasuf.concurrency.TestThreadUnsafe.lambda$main$0(TestThreadUnsafe.java:15)
	at java.lang.Thread.run(Thread.java:750)
```

报错原因是：如果线程2还未执行`add`操作，而线程1就开始执行`remove`操作。而无论哪个线程中的`method2`和`method3`引用的都是同一个对象中的`list`对象。

<img src="../images/concurrency/14.png" alt="image-20240209下午43032316" style="zoom:50%;" />

而如果将`list`作为`method1()`的局部变量，则不会出现上述问题：

```java
public class TestThreadUnsafe {

    static final int THREAD_NUMBER = 2;
    static final int LOOP_NUMBER = 200;

    public static void main(String[] args) {
        ThreadSafe threadSafe = new ThreadSafe();
        for (int i = 0; i < THREAD_NUMBER; i++) {
            new Thread(() -> {
                threadSafe.method1(LOOP_NUMBER);
            }, "Thread" + (i + 1)).start();
        }
    }
}

class ThreadSafe {

    public void method1(int loopNumber) {
        List<String> list = new ArrayList<>();
        for (int i = 0; i < loopNumber; i++) {
            method2(list);
            method3(list);
        }
    }

    private void method2(List<String> list) {
        list.add("1");
    }

    private void method3(List<String> list) {
        list.remove(0);
    }
}
```

分析：

- `list`是局部变量，每个线程调用时都会创建其不同实例，不会共享
- 而`method2()`的参数是从`method1()`中传递过来的，与`method1()`中引用的是同一个对象
- `method3()`的参数分析与`method2()`相同

<img src="../images/concurrency/15.png" alt="image-20240209下午43715308" style="zoom:50%;" />

注意，上述代码`method2`和`method3`都是`private`修饰；如果改成`public`的，是否会存在线程安全问题？

- 情况一：如果有其他线程调用`method2`和`method3` -> 这种情况不会存在线程安全问题。因为其他线程调用的时候传递的参数是不同`list`对象的引用

- 情况二：在情况一的基础上，为`ThreadSafe`添加子类，子类覆盖`method2`或`method3`方法 ，而该方法另外开启线程执行操作-> 这种情况会出现线程安全问题。因为子类执行的`method3`方法跟父类执行的`method2`方法引用的是同一个`list`对象
  ```java
  public class TestThreadUnsafe {
  
      static final int THREAD_NUMBER = 2;
      static final int LOOP_NUMBER = 200;
  
      public static void main(String[] args) {
          ThreadSafeSubClass threadSafeSubClass = new ThreadSafeSubClass();
          for (int i = 0; i < THREAD_NUMBER; i++) {
              new Thread(() -> {
                  threadSafeSubClass.method1(LOOP_NUMBER);
              }, "Thread" + (i + 1)).start();
          }
      }
  }
  
  class ThreadSafe {
  
      public void method1(int loopNumber) {
          List<String> list = new ArrayList<>();
          for (int i = 0; i < loopNumber; i++) {
              method2(list);
              method3(list);
          }
      }
  
      public void method2(List<String> list) {
          list.add("1");
      }
  
      public void method3(List<String> list) {
          list.remove(0);
      }
  }
  
  class ThreadSafeSubClass extends ThreadSafe {
      @Override
      public void method3(List<String> list) {
          new Thread(() -> list.remove(0)).start();
      }
  }
  ```

可见访问修饰符会影响线程安全问题。`private`保证了子类不会重写父类的方法，从而避免了引用的共享导致的线程安全问题。另外也可以使用`final`来达到防止子类对父类方法重写的效果。

### 3.4.3 常见线程安全类

- String
- Integer
- StringBuffer
- Random
- Vector
- Hashtable
- java.util.concurrent包下的类

这里说它们是线程安全的，指的是多个线程调用它们同一个实例的某个方法时，是线程安全的。也可以理解为：

> 它们的每个方法是原子的。以下操作是线程安全的

```java
HashTable table = new HashTable();
new Thread(() -> table.put("key", "value1")).start();
new Thread(() -> table.put("key", "value2")).start();
```

> 但是它们多个方法的组合调用不是线程安全的

考虑下面代码：

```java
HashTable table = new HashTable();
// Thead1, Thread2同时调用下面代码：
if (table.get("key") == null) {
  table.put("key", "value");
}
```

<img src="../images/concurrency/16.png" alt="image-20240209下午52647836" style="zoom:50%;" />

方法`put`和`get`分别都是线程安全的，但是组合一起使用，无法保证线程安全。

### 3.4.4 不可变类线程安全性

`String`、`Integer`等都是不可变类，因为其内部的状态不可以改变，因此它们的方法都是线程安全的。

> 测试1

```java
// Servlet只有一个实例，内部资源是共享的
public class MyServlet extends HttpServlet {
  Map<String, Object> map = new HashMap<>(); // 非线程安全
  String s1 = "..."; // 线程安全
  final String s2 = "..."; // 线程安全
  Date d1 = new Date(); // 非线程安全
  final Date d2 = new Date(); // 非线程安全
  
  public void doGet(HttpServletRequest request, HttpServletResponse response) {
    // 使用上述变量
  }
}
```

> 测试2

```java
// Servlet只有一个实例，内部资源是共享的
public class MyServlet extends HttpServlet {
  private UserService userService = new UserServiceImpl(); // 非线程安全
  
  public void doGet(HttpServletRequest request, HttpServletResponse response) {
    userService.update();
  }
}

public class UserServiceImpl implements UserService {
  // 记录调用次数
  private int count = 0;
  
  public void update() {
    count++; // count被多个线程共享，无法保证线程安全性
  }
}
```

> 测试3

```java
@Aspect
@Component // 默认为单例，内部资源在多个线程之间共享
public class MyAspect {
  private long start = 0L; // 非线程安全
  
  @Before("execution(* *(..))")
  public void before() {
    start = System.nanoTime();
  }
  
  @After("execution(* *(..))")
  public void after() {
    long end = System.nanoTime();
    System.out.println("cost time: " + (end - start));
  }
}

// 要解决上述代码线程安全问题，最好是使用环绕通知
```

> 测试4

```java
public class MyServlet extends HttpServlet {
  // 线程安全。userService内部线程安全
  private UserService userService = new UserServiceImpl(); 
  
  public void doGet(HttpServletRequest request, HttpServletResponse response) {
    userService.update();
  }
}

public class UserServiceImpl implements UserService {
  // 线程安全。因为虽然userDao被多个线程共享，但是userDao内部操作是线程安全的
  private UserDao userDao = new UserDaoImpl();
  
  public void update() {
    userDao.update();
  }
}

public class UserDaoImpl implements UserDao {
  // 线程安全。因为都是方法内的局部变量，不存在UserDaoImpl的实例变量，不存在资源共享问题
  public void update() {
    String sql = "update user set password = ? where username = ?";
    try (Connection conn = new DriverManager.getConnection(..)) {
      // ...
    } catch (Exception e) {
      // ...
    }
  }
}
```

> 测试5

```java
public class MyServlet extends HttpServlet {
  // 非线程安全。userServiceImpl内部不是线程安全
  private UserService userService = new UserServiceImpl(); 
  
  public void doGet(HttpServletRequest request, HttpServletResponse response) {
    userService.update();
  }
}

public class UserServiceImpl implements UserService {
  // 非线程安全。userDao内部不是线程安全的
  private UserDao userDao = new UserDaoImpl();
  
  public void update() {
    userDao.update();
  }
}

public class UserDaoImpl implements UserDao {
  // 非线程安全。conn为UserDaoImpl类的实例变量，会导致多个线程共享资源问题
  private Connection conn = null;
  public void update() {
    String sql = "update user set password = ? where username = ?";
    conn = DriverManager.getConnection(...);
    // ...
    conn.close();
  }
}
```

> 测试6

```java
public class MyServlet extends HttpServlet {
  // 线程安全。userService内部线程安全
  private UserService userService = new UserServiceImpl(); 
  
  public void doGet(HttpServletRequest request, HttpServletResponse response) {
    userService.update();
  }
}

public class UserServiceImpl implements UserService {
  // 线程安全。userDao是方法内局部变量
  public void update() {
    UserDao userDao = new UserDaoImpl();
    userDao.update();
  }
}

public class UserDaoImpl implements UserDao {
  // 非线程安全。conn为UserDaoImpl类的实例变量，会导致多个线程共享资源问题
  private Connection conn = null;
  public void update() {
    String sql = "update user set password = ? where username = ?";
    conn = DriverManager.getConnection(...);
    // ...
    conn.close();
  }
}
```

> 测试7

```java
public abstract class Test {
  public void bar() {
    SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
    foo(sdf);
  }
  
  public abstract foo(SimpleDateFormat sdf);
  
  public static void main(String[] args) {
    new Test().bar();
  }
}
```

其中`foo`方法要被子类重写，行为是不确定的，可能导致线程安全问题发生，被称之为“外星方法”。例如：

```java
public void foo(SimpleDateFormat sdf) {
  String dateStr = "1999-10-11 00:00:00";
  for (int i = 0; i < 20; i++) {
    new Thread(() -> {
      try {
        sdf.parse(dateStr);
      } catch (ParseException e) {
        e.printStackTrace();
      }
    }).start();
  }
}
```

## 3.5 案例

### 3.5.1 卖票测试

```java
@Slf4j(topic = "c.TicketSelling")
public class TicketsSelling {
    public static void main(String[] args) throws InterruptedException {
        // 模拟多人买票
        TicketWindow window = new TicketWindow(1000);
        // 卖出的票数统计
        List<Integer> amountList = new Vector<>();
        // 所有线程集合
        List<Thread> threadList = new ArrayList<>();

        for (int i = 0; i < 4000; i++) {
            Thread thread = new Thread(() -> {
                int amount = window.sell(randomAmount());
                try {
                    Thread.sleep(randomAmount());
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
                amountList.add(amount);
            });
            threadList.add(thread);
            thread.start();
        }
        for (Thread thread : threadList) {
            thread.join();
        }

        // 统计卖出票数和剩余票数
        log.debug("余票：{}", window.getRemaining());
        log.debug("卖出：{}", amountList.stream().mapToInt(i -> i).sum());
    }

    static Random random = new Random();

    public static int randomAmount() {
        return random.nextInt(5) + 1;
    }
}

class TicketWindow {
    private int remaining; // 实例变量，存在资源共享，导致线程安全问题

    public TicketWindow(int remaining) {
        this.remaining = remaining;
    }

    public int getRemaining() {
        return remaining;
    }

    public int sell(int amount) {
        if (this.remaining >= amount) {
            this.remaining -= amount;
            return amount;
        } else {
            return 0;
        }
    }
}
```

上述代码存在线程安全问题，容易得到如下不合理输出：

```java
14:01:17.916 [main] c.TicketSelling - 余票：0
14:01:17.931 [main] c.TicketSelling - 卖出：1002
```

修改代码如下：

```java
public synchronized int sell(int amount) {
  if (this.remaining >= amount) {
    this.remaining -= amount;
    return amount;
  } else {
    return 0;
  }
}
```

### 3.5.2 转账测试

```java
@Slf4j(topic = "c.TransferTest")
public class TransferTest {
    public static void main(String[] args) throws InterruptedException {
        Account a = new Account(1000);
        Account b = new Account(1000);
        Thread t1 = new Thread(() -> {
            for (int i = 0; i < 1000; i++) {
                a.transfer(b, randomAmount());
            }
        }, "t1");
        Thread t2 = new Thread(() -> {
            for (int i = 0; i < 1000; i++) {
                b.transfer(a, randomAmount());
            }
        }, "t2");
        t1.start();
        t2.start();
        t1.join();
        t2.join();
        log.debug("total: {}", a.getMoney() + b.getMoney());
    }

    static Random random = new Random();

    public static int randomAmount() {
        return random.nextInt(5) + 1;
    }
}

class Account {
    private int money;

    public Account(int money) {
        this.money = money;
    }

    public int getMoney() {
        return money;
    }

    public void setMoney(int money) {
        this.money = money;
    }

    public void transfer(Account target, int amount) {
        if (this.money > amount) {
            this.setMoney(this.getMoney() - amount);
            target.setMoney(target.getMoney() + amount);
        }
    }
}
```

上述代码依然存在线程安全问题，容易导致错误输出：

```java
14:23:03.339 [main] c.TransferTest - total: 3480
```

导致出问题的还是`transfer`方法。但是即使加上方法上的`synchronized`也不能保证线程安全：

```java
public synchronized void transfer(Account target, int amount) {
  if (this.money > amount) {
    this.setMoney(this.getMoney() - amount);
    target.setMoney(target.getMoney() + amount);
  }
}
```

相当于：

```java
public void transfer(Account target, int amount) {
  synchronized(this) {
      if (this.money > amount) {
        this.setMoney(this.getMoney() - amount);
        target.setMoney(target.getMoney() + amount);
      }
  }
}
```

加锁的对象是`this`，但是该方法会有另外的target也存在线程安全问题，没有被锁住。解决方法就是对`Account.class`进行加锁，这样可以同时锁住所有`Account`对象：

```java
public void transfer(Account target, int amount) {
  synchronized (Account.class) {
    if (this.money > amount) {
      this.setMoney(this.getMoney() - amount);
      target.setMoney(target.getMoney() + amount);
    }
  }
}
```

但是要注意，这种方式效率很低。后续我们会有更有效的解决方案。

## 3.6 Monitor概念

### 3.6.1 Java对象头

以32位虚拟机为例：

> 普通对象

<img src="../images/concurrency/17.png" alt="image-20240211上午84306045" style="zoom:50%;" />

> 数组对象

<img src="../images/concurrency/18.png" alt="image-20240211上午84346719" style="zoom:50%;" />

其中Mark Word结构为：

<img src="../images/concurrency/19.png" alt="image-20240211上午84444495" style="zoom:50%;" />

### 3.6.2 Monitor

Monitor被翻译为监视器或管程，每个Java对象都可以关联一个Monitor对象，如果使用`synchronized`给对象上锁（重量级）之后，改对象的Mark Word中就被设置指向Monitor对象的指针。

Monitor结构如下：

<img src="../images/concurrency/20.png" alt="image-20240211上午84938305" style="zoom:50%;" />

- 刚开始Monitor中`Owner`为`null`
- 当Thread-2执行`synchronized(obj)`，就会将Monitor的所有者`Owner`设置为`Thread-2`。Monitor中只能有一个Owner
- 在Thread-2上锁的过程中，如果Thread-3，Thread-4，Thread-5也来执行`synchronized(obj)`，就会进入`EntryList`(BLOCKED)
- Thread-2执行完同步代码块的内容，然后唤醒EntryList中等待的线程来竞争锁，竞争是非公平的，不会按照先进先出顺序，而是JDK实际的底层实现来进行
- 途中WaitSet中的Thread-0，Thread-1是之前获得过锁，但是条件不满足进入WAITING状态的线程。后面讲`wait-notify`时会分析

注意：

- `synchronized`必须是进入同一个对象的monitor才有上述效果
- 不加`synchronized`的对象不会关联监视器，不遵循以上规则