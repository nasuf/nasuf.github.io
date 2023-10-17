# 1. 概述

## 1.1 字节码文件的跨平台性

### 1.1.1 Java语言：跨平台的语言
- 当Java源代码成功编译成字节码后，如果想在不同的平台上面运行，则无需再次编译
- 这个优势不再那么吸引人了，Python、PHP、Perl、Ruby、Lisp等有强大的解释器
- 跨平台似乎已经快称为一门语言必选的特性

### 1.1.2 Java虚拟机：跨语言的平台
Java虚拟机不和包括Java语言在内的任何语言绑定，它只与**Class文件**这种特定的二进制文件格式所关联。无论使用何种语言进行软件开发，只要能将源文件编译为正确的Class文件，那么这种语言就可以在Java虚拟机上执行。可以说，统一而强大的Class文件结构就是Java虚拟机的基石和桥梁

所有的JVM全部遵守Java虚拟机规范，也就是说所有的JVM环境都是一样的额，这样一来字节码文件可以在各种JVM上运行

![image.png](/images/jvm/15/1.png)
参考官方规范 https://docs.oracle.com/javase/specs/index.html

### 1.1.3 javac
- 想要让一个Java程序正确地运行在JVM中，Java源码就必须要被编译为符合JVM规范的字节码
- 前端编译器的主要任务就是负责将符合Java语法规范的Java代码转换为符合JVM规范的字节码文件
- `javac`是一种能够将Java源码编译为字节码的前端编译器
- `javac`编译器在将Java源码编译为一个有效的字节码文件过程中经历了四个步骤：**词法解析**、**语法解析**、**语义解析**及**生成字节码**

![image.png](/images/jvm/15/2.png)
Oracle JDK软件包括两部分内容：

- 一部分是将Java源代码编译成Java虚拟机的指令集的编译器
- 另一部分是用于实现Java虚拟机的运行时环境

## 1.2 Java的前端编译器

![image.png](/images/jvm/15/3.png)
### 1.2.1 前端编译器 vs 后端编译器
Java源代码的编译结果是字节码，那么肯定需要有一种编译器能够将Java源码编译为字节码，承担这个重要热舞的就是配置在path环境变量中的javac编译器。**javac是一种能够将Java源码编译为字节码的前端编译器**

HotSpot VM并没有强制要求前端编译器只能使用javac来编译字节码，其实只要编辑结果符合JVM规范，可以被JVM所识别即可。在Java的前端编译器领域，除了javac之外，还有一种被大家经常用到的前端编译器，就是内置在Eclipse中的`ECJ（Eclipse Compiler for Java）`。**和javac的全量式编译不同，ECJ是一种增量式编译器**
- 在Eclipse，当开发与人员编写完代码后，使用`Ctrl+S`快捷键时，**ECJ编译器所采取的的编译方案是把未编译部分的源码逐行进行编译，而非每次都是全量编译**。因此ECJ的编译效率会比javac更加迅速和高效，当然编译质量和javac相比大致还是一样的
- ECJ不仅是Eclipse的默认内置前端编译器，在Tomcat中同样也是使用ECJ编译器来编译jsp文件，由于ECJ编译器是采用GPLv2的开源协议进行源代码公开，所以大家可以登录eclipse官网下载ECJ编译器的源码进行二次开发
- 默认情况下，Intellij IDEA使用javac编译器（还可以自己设置为AspectJ编译器`ajc`)

**前端编译器并不会直接涉及编译优化等方面的技术，而是将这些具体优化细节移交给HotSpot的JIT编译器负责**


## 1.3 透过字节码指令看代码细节
> 代码演示 1
```
public class IntegerTest {
    public static void main(String[] args) {

        Integer x = 5;
        int y = 5;
        System.out.println(x == y); // true

        Integer i1 = 10;
        Integer i2 = 10;
        System.out.println(i1 == i2);   // true

        Integer i3 = 128;
        Integer i4 = 128;
        System.out.println(i3 == i4);   // false
    }
}
```
部分字节码解析如下：
```java
 0 iconst_5  // 将常量5放入操作数栈
 1 invokestatic #2 <java/lang/Integer.valueOf>  // 调用Integer.valueOf方法，源码见下文
 4 astore_1  // 将返回值保存在局部变量表索引为1的位置（引用类型）
 5 iconst_5  // 将常量5放入操作数栈
 6 istore_2  // 将返回值保存在局部变量表索引为2的位置
 7 getstatic #3 <java/lang/System.out>
10 aload_1  // 取出局部变量表索引为1位置的值（引用类型）
11 invokevirtual #4 <java/lang/Integer.intValue>  // 拆箱
14 iload_2  // 取出局部变量表索引为2位置的值
15 if_icmpne 22 (+7)  // 比较
...
```
`Integer.valueOf()` 源码:
```java
public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}
```
其中`IntegerCache.low`为`-128`，`IntegerCache.high`为`127`，在此区间内的数值，都会从`Integer.cache[]`数组缓存中取，所以在上述第第一个和第二个判断中为true，而第三个判断，128超出了缓存范围，所以为false

> 代码演示 2
```java
public class StringTest {
    public static void main(String[] args) {
        String str = new String("hello") + new String("world");
        String str1 = "helloworld";
        System.out.println(str == str1);    // false
    }
}
```
部分字节码解析如下：
```java
 0 new #2 <java/lang/StringBuilder>  // 创建StringBuilder实例
 3 dup
 4 invokespecial #3 <java/lang/StringBuilder.<init>>
 7 new #4 <java/lang/String>
10 dup
11 ldc #5 <hello> // 加载hello字符串
13 invokespecial #6 <java/lang/String.<init>>
16 invokevirtual #7 <java/lang/StringBuilder.append>
19 new #4 <java/lang/String>
22 dup
23 ldc #8 <world>
25 invokespecial #6 <java/lang/String.<init>>
28 invokevirtual #7 <java/lang/StringBuilder.append>
31 invokevirtual #9 <java/lang/StringBuilder.toString>
34 astore_1
35 ldc #10 <helloworld>
37 astore_2
38 getstatic #11 <java/lang/System.out>
41 aload_1
42 aload_2
43 if_acmpne 50 (+7)
46 iconst_1
47 goto 51 (+4)
50 iconst_0
51 invokevirtual #12 <java/io/PrintStream.println>
54 return
```
由于字符串拼接使用`StringBuilder`进行操作，且返回值是`String`类型，所以会调用`StringBuilder.toString`方法，其源码如下：
```java
@Override
public String toString() {
    // Create a copy, don't share the array
    return new String(value, 0, count);
}
```
可以看到创建了新的String对象，所以最后的判断为false
> 代码演示 3
```java
package com.nasuf.jdk8;

public class SonTest {
    public static void main(String[] args) {
        Parent p = new Sub();
        System.out.println(p.x);
    }

}


class Parent {
    int x = 10;
    public Parent() {
        this.print();
        x = 20;
    }

    public void print() {
        System.out.println("Parent.x = " + x);
    }
}

class Sub extends Parent {
    int x = 30;
    public Sub() {
        this.print();
        x = 40;
    }
    public void print() {
        System.out.println("Sub.x = " + x);
    }
}
```
输出如下：
```java
Sub.x = 0
Sub.x = 30
20
```
先查看Parent的构造方法字节码如下：
```java
 0 aload_0 
 1 invokespecial #1 <java/lang/Object.<init>>  // 调用父类Object的构造方法
 4 aload_0
 5 bipush 10  // 加载常量10
 7 putfield #2 <com/nasuf/jdk8/Parent.x> // 将10赋值给x变量
10 aload_0 
11 invokevirtual #3 <com/nasuf/jdk8/Parent.print>  // 调用print方法
14 aload_0
15 bipush 20  // 加载常量20
17 putfield #2 <com/nasuf/jdk8/Parent.x> // 将10赋值给x变量
20 return
```
查看Sub的构造方法字节码如下：
```java
 0 aload_0
 // 以下字节码指令：调用父类Parent的构造方法，父类构造方法中调用了this.print()，
 // 而子类Sub中重写了print()方法，所以此处实际调用的是子类的print()方法。
 // 而此时子类的x尚未赋值，为初始值0，所以输出：Sub.x = 0
 1 invokespecial #1 <com/nasuf/jdk8/Parent.<init>>  
 4 aload_0
 5 bipush 30  // 加载常量30
 7 putfield #2 <com/nasuf/jdk8/Sub.x>  // 为子类的Sub.x赋值30
10 aload_0
11 invokevirtual #3 <com/nasuf/jdk8/Sub.print>  // 此时输出：Sub.x = 30
14 aload_0
15 bipush 40  // 加载常量40
17 putfield #2 <com/nasuf/jdk8/Sub.x>  // 为子类的Sub.x赋值40
20 return
```

# 2. 虚拟机的基石：Class文件
> 字节码文件里是什么？
> 源代码经过编译器编译之后会生成一个或多个字节码文件，字节码是一种二进制的类文件，它的内容是JVM指令（或成字节码指令），而不像C、C++经由编译器直接生成机器码

> 什么是字节码指令（byte code）？
> Java虚拟机的指令由一个字节长度的、代表着某种特定操作含义的**操作码（opcode）** 以及跟随其后的零至多个代表此操作所需参数的**操作数（operand）** 所构成。**虚拟机中许多指令并不包含操作数，只有一个操作码**。比如：
```java
4 aload_0  // 只有操作码aload_0
5 bipush 10  // 包含操作码bipush和操作数10
```

> 如何解读供虚拟机解释执行的二进制字节码
> 方式一：使用binaryViewer类似的软件查看字节码内容。如下是用十六进制查看字节码内容：

![image.png](/images/jvm/15/4.png)

方式二：使用`javap`指令：JDK自带的反解析工具

![image.png](/images/jvm/15/5.png)

**方式三：使用`Jclasslib`插件**

![image.png](/images/jvm/15/6.png)

# 3. Class文件结构


![image.png](/images/jvm/15/7.png)

参考官方文档链接：https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html

> Class 类的本质
> 任何一个Class文件都对应着唯一一个类或接口的定义信息，但反过来说，Class文件实际上它并不一定以磁盘文件的形式存在。**Class文件是一组以8字节为基础单位的二进制流**

> Class 文件格式
> Class的结构不像XML等描述语言，由于它没有任何分隔符号，所以在其中的数据项，无论是字节顺序还是数量，都是被严格限定的，哪个字节代表什么含义，长度是多少，先后顺序如何，都不允许改变

**Class文件格式采用一种类似于C语言结构体的方式进行数据存储。这种结构中只有两种数据类型：无符号数和表**
- **无符号数**属于基本的数据类型，以u1、u2、u4、u8来分别代表1个字节、2个字节、4个字节和8个字节的无符号数，无符号数可以用来描述数字、索引引用，数量值或者按照UTF-8编码构成字符串值
- **表**是由多个无符号数或者其他表作为数据项构成的复合数据类型，所有表都习惯性地以`_info`结尾。表用于描述有层次关系的复合结构的数据。**整个Class文件本质上就是一张表**。由于表没有固定长度，所以通常会在其前面加上个数说明

> Class文件结构概述
> Class文件的结构并不是一成不变的，随着Java虚拟机的不断发展，总是不可避免地会对Class文件结构做出一些调整。但是其基本结构和框架是非常稳定的。结构如下：

| 类型           | 名称                | 说明                    | 长度    | 数量                  |
| -------------- | ------------------- | ----------------------- | ------- | --------------------- |
| u4             | magic               | 魔数，识别Class文件格式 | 4个字节 | 1                     |
| u2             | minor_version       | 副版本号（小版本）      | 2个字节 | 1                     |
| u2             | major_version       | 主版本号（大版本）      | 2个字节 | 1                     |
| u2             | constant_pool_count | 常量池计数器            | 2个字节 | 1                     |
| cp_info        | constant_pool       | 常量池表                | n个字节 | constant_pool_count-1 |
| u2             | access_flags        | 访问标识                | 2个字节 | 1                     |
| u2             | this_class          | 类索引                  | 2个字节 | 1                     |
| u2             | super_class         | 父类索引                | 2个字节 | 1                     |
| u2             | interfaces_count    | 接口计数器              | 2个字节 | 1                     |
| u2             | interfaces          | 接口索引集合            | 2个字节 | interfaces_count      |
| u2             | fields_count        | 字段计数器              | 2个字节 | 1                     |
| field_info     | fields              | 字段表                  | n个字节 | fields_count          |
| u2             | methods_count       | 方法计数器              | 2个字节 | 1                     |
| method_info    | methods             | 方法表                  | n个字节 | methods_count         |
| u2             | attributes_count    | 属性计数器              | 2个字节 | 1                     |
| attribute_info | attributes          | 属性表                  | n个字节 | attributes_count      |

![image.png](/images/jvm/15/8.png)

## 3.1 Magic Number 魔数
- 每个Class文件开头的4个字节的无符号整数称为魔数
- 它的唯一作用是确定这个文件是否为一个能被虚拟机接收的有效合法的Class文件。即：魔数是Class文件的标识符
- 魔数值固定为`0xCAFEBABE`。不会改变
- 如果一个Class文件不以`0xCAFEBABE`开头，虚拟机在进行文件校验的时候会直接抛出以下错误：
```java
Error: A JNI error has occured, please check your installation and try again 
Exception in thread "main" java.lang.ClassFromatError: Incompatible magic value 1885430635 in file StringTest
```
- 使用魔数而不是扩展名来进行识别主要是基于安全方面的考虑。因为文件扩展名可以随意改动

## 3.2 Class文件版本号
- 紧接着魔数的4个字节存储的是Class文件的版本号。同样也是4个字节。第5和第6字节所代表的含义就是编译的副版本号`minor_version`，而第7和第8字节就是编译的主版本号`major_version`
- 它们共同构成了class文件的格式版本号。譬如某个Class文件的主版本号为`M`，副版本号为`m`，那么这个Class文件的格式版本号就确定为`M.m`
- 版本号和Java编译器的对应关系如下表：

    | 主版本（十进制） | 副版本（十进制） | 编译器版本 |
    | ---------------- | ---------------- | ---------- |
    | 45               | 3                | 1.1        |
    | 46               | 0                | 1.2        |
    | 47               | 0                | 1.3        |
    | 48               | 0                | 1.4        |
    | 49               | 0                | 1.5        |
    | 50               | 0                | 1.6        |
    | 51               | 0                | 1.7        |
    | 52               | 0                | 1.8        |
    | 53               | 0                | 1.9        |
    | 54               | 0                | 1.10       |
    | 55               | 0                | 1.11       |
- Java的版本号是从45开始的，JDK 1.1之后的每个JDK大版本发布主版本号向上加1
- 不同版本的Java编译器编译的Class文件对应的版本是不一样的。目前，高版本的Java虚拟机可以执行由低版本编译器生成Class文件，但是低版本的Java虚拟机不能执行由高版本编译器生成的Class文件。否则JVM会抛出`java.lang.UnsupportedClassVersionError`异常
- 在实际应用中，由于开发环境和生产环境的不同，可能会导致该问题的发生。因此，需要我们在开发时，特别注意开发编译的JDK版本和生产环境的JDK版本是否一致
- 虚拟机的JDK版本为`1.k (k≥2)`时，对应的Class文件格式版本号的范围为`45.0 - 44+k.0 (含两端)`

## 3.3 常量池：存放所有常量

![image.png](/images/jvm/15/9.png)
- 常量池时Class文件中内容最为丰富的区域之一。常量池对于Class文件中的字段和方法解析也有着至关重要的作用
- 随着Java虚拟机的不断发展，常量池的内容也日渐丰富。可以说，常量池是整个Class文件的基石
- 在版本号之后，紧接着就是常量池的存放，以及若干个常量池表项
- 常量池中常量的数量是不固定的，所以在常量池的入口需要放置一项`u2`类型的无符号数，代表常量池内容计数值（constant_pool_count）。与Java中语言习惯不一样的是，这个容量计数是从1开始而不是0开始的

    | 类型          | 名称                | 数量                    |
    | ------------- | ------------------- | ----------------------- |
    | u2 (无符号数) | constant_pool_count | 1                       |
    | cp_info (表)  | constant_pool       | constant_pool_count - 1 |

    由上表可见，Class文件使用了一个前置的容量计数器（constant_pool_count）加若干个连续的数据项（constant_pool）的形式来描述常量池内容。我们把这一系列连续常量池数据成为常量池集合
- 常量池表项中，用于存放编译时期生成的各种字面量和符号引用，这部分内容将在类加载后进入方法区的运行时常量池中存放

### 3.3.1 常量池计数器（constant_pool_count）
- 由于常量池的数量不固定，时长时短，所以需要放置两个字节来表示常量池容量计数值
- 常量池容量计数值（u2类型）：从1开始，表示常量池中有多少项常量。**即constant_pool_count=1表示常量池中有0个常量项**
- 在Demo中，其值为`0x0016`，换算成十进制为22.需要注意的是，这里实际上只有21项常量。索引范围是1-21.
    - 通常我们写代码都是从0开始的，但是这里常量池却是从1开始。因为它把第0项常量空出来了。这是为了满足后面某些指向常量池的索引值的数据在特定情况下需要表达”不引用任何一个常量项目“的含义，这种情况可用索引值0来表示
### 3.3.2 常量池表
- constant_pool是一种表结构，以1 ~ constant_pool_count - 1 为索引，表明了后面有多少个常量项
- 常量池主要存放两大类常量：**字面量（Literal）** 和 **符号引用（Symbolic References）**
- 它包含了class文件结构及其子结构中引用的所有字符串常量、类或接口名、字段名和其他常量。常量池中的每一项都具备相同的特征。第1个字节作为类型标记,用于确定该项的格式,这个字节称为tag byte (标记字节、标签字节）

    | 类型                             | 标志（或标识） | 描述                   |
    | -------------------------------- | -------------- | ---------------------- |
    | CONSTANT_utf8_info               | 1              | UTF-8编码的字符串      |
    | CONSTANT_Integer_info            | 3              | 整型字面量             |
    | CONSTANT_Float_info              | 4              | 浮点型字面量           |
    | CONSTANT_Long_info               | 5              | 长整型字面量           |
    | CONSTANT_Double_info             | 6              | 双精度浮点型字面量     |
    | CONSTANT_Class_info              | 7              | 类或接口的符号引用     |
    | CONSTANT_String_info             | 8              | 字符串类型字面量       |
    | CONSTANT_Fieldref_info           | 9              | 字段的符号引用         |
    | CONSTANT_Methodref_info          | 10             | 类中方法的符号引用     |
    | CONSTANT_InterfaceMethodref_info | 11             | 接口中方法的符号引用   |
    | CONSTANT_NameAndType_info        | 12             | 字段或方法的符号引用   |
    | CONSTANT_MethodHandle_info       | 15             | 表示方法句柄           |
    | CONSTANT_MethodType_info         | 16             | 标志方法类型           |
    | CONSTANT_InvokeDynamic_info      | 18             | 表示一个动态方法调用点 |

#### 3.3.2.1 字面量和符号引用
常量池主要存放两大类常量：字面量和符号引用

| 常量     | 具体的常量          |
| -------- | ------------------- |
| 字面量   | 文本字符串          |
|          | 声明为final的常量值 |
| 符号引用 | 类和接口的全限定名  |
|          | 字段的名称和描述符  |
|          | 方法的名称和描述符  |

> 全限定名
> `com/nasuf/test/Demo` 这个就是类的全限定名，仅仅是把包名的`.`换成了`/`，为了使连续的多个全限定名之间不产生混淆，在使用时最后一般会加入一个`;`表示全限定名结束

> 简单名称
> 简单名称是指没有类型和参数修饰的方法或者字段名称，例如类中方法`add()`和字段`num`的简单名称为`add`和`num`

> 描述符
> 描述符的作用是用来描述字段的数据类型、方法的参数列表（包括数量、类型以及顺序）和返回值。根据描述符规则，基本数据类型（byte、char、double、float、long、short、boolean）以及代表无返回值的void类型都用一个大写字符来表示，而对象类型则用字符`L`加对象的全限定名来表示

| 标志符 | 含义                                               |
| ------ | -------------------------------------------------- |
| B      | 基本数据类型byte                                   |
| C      | 基本数据类型char                                   |
| D      | 基本数据类型double                                 |
| F      | 基本数据类型float                                  |
| I      | 基本数据类型int                                    |
| J      | 基本数据类型long                                   |
| S      | 基本数据类型short                                  |
| Z      | 基本数据类型boolean                                |
| V      | 代表void类型                                       |
| L      | 对象类型，比如：Ljava/lang/Object;                 |
| [      | 数组类型，代表一维数组。比如: double[][][] is [[[D |

数组类型代码测试
```java
public class ArrayTest {
    public static void main(String[] args) {
        Object[] arr1 = new Object[1];
        System.out.println(arr1);

        String[] arr2 = new String[1];
        System.out.println(arr2);

        long[][] arr3 =  new long[10][];
        System.out.println(arr3);
    }
}
```
输出如下
```java
[Ljava.lang.Object;@2503dbd3
[Ljava.lang.String;@4b67cf4d
[[J@7ea987ac
```
> 补充说明
> 虚拟机在加载Class文件时才会进行动态链接，也就是说，Class文件中不会保存各个方法和字段的最终内存布局信息，因此，这些字段和方法的符号引用不经过转换是无法直接被虚拟机使用的。当虚拟机运行时，需要从常量池中获得对应的符号引用，再在类加载过程中的解析阶段将其替换为直接引用，并翻译到具体的内存地址中
- 符号引用：符号引用以一组符号来描述所引用的目标，符号可以是任何形式的字面量，只要使用时能无歧义地定位到目标即可。符号引用与虚拟机实现的内存布局无关，引用的目标并不一定已经加载到了内存中
- 直接引用：直接引用可以是直接指向目标的指针、相对偏移量或是一个能间接定位到目标的句柄。直接引用是与虚拟机实现的内存布局相关的，同一个符号引用在不同虚拟机实例上翻译出来的直接引用一般不会相同。如果有了直接引用，那说明引用的目标必定已经存在于内存之中了

#### 3.3.2.2 常量类型和结构
常量池中每一项常量都是一个表，JDK1.7之后共有`14`种不同的表结构数据。如下表格所示：

   ![image.png](/images/jvm/15/10.png)
- 根据上图每个类型的描述我们也可以知道每个类型是用来描述常量池中哪些内容（主要是字面量、符号引用）的
- 标志为15、16、18的常量项类型是用来支持动态语言调用的（JDK1.7时才加入）
> 细节说明：
- CONSTANT_Class_info 结构表示类或接口
- CONSTANT_Fieldref_info、CONSTANT_Methodref_info、CONSTANT_InterfaceMethodref_info 结构表示字段、方法和接口方法
- CONSTANT_NameAndType_info 结构表示字段或方法，但是和上面的3个结构不同，CONSTANT_NameAndType_info结构没有指明该字段或方法所属的类或接口
- CONSTANT_String_info 结构表示String类型的常量对象
- CONSTANT_Integer_info、CONSTANT_Float_info 结构表示4字节（int和float）的数值常量
- CONSTANT_Long_info、CONSTANT_Double_info 结构表示8字节（long和double）的数值常量
    - 在class文件的常量池表中，所有的8字节常量均占2个表成员（项）的空间。如果一个CONSTANT_Long_info或CONSTANT_Double_info结构的项在常量池表中索引位为`n`，则常量池中下一个可用项的索引位为`n+2`，此时常量池表中索引为`n+1`的项仍然有效但必须视为不可用的
- CONSTANT_utf8_info 结构表示字符常量的值
- CONSTANT_MethodHandle_info 结构表示方法句柄
- CONSTANT_MethodType_info 结构表示方法类型
- CONSTANT_InvokeDynamic_info 结构表示`invokedynamic`指令所用到的引导方法（bootstrap method）、引导方法所用到的动态调用名称（dynamic invocation name）、参数和返回类型，并可以给引导方法传入一系列称为静态参数（static argument）的常量
> 总结1
- 这14种表（或者常量项结构）的共同点是：表开始的第一位是一个u1类型的标志位（tag），代表当前这个常量项使用的是哪种表结构，即哪种常量类型
- 在常量池列表中，CONSTANT_Utf8_info常量项是一种使用改进过的UTF-8编码格式来存储诸如文字字符串、类或者接口的全限定名、字段或者方法的简单名称以及描述符等常量字符串信息
- 这14种常量项结构还有一个特点是，其中13个常量项占用的字节固定，只有CONSTANT_Utf8_info占用字节不固定。其大小由length决定。因为从常量池存放的内容可知，其存放的是字面量和符号引用，最终这些内容都会是一个字符串，这些字符串的大小是在编写程序时才确定的。比如定义一个类，类名可长可短，所以在没编译前，大小不固定；编译后，通过UTF-8编码，就可以知道其长度
> 总结2
- 常量池：可以理解为Class文件之中的资源仓库，它是Class文件结构中与其他项目关联最多的数据类型（后面的很多数据类型都会指向此处），也是占用Class文件空间最大的数据项目之一
- 常量池中为什么要包含这些内容？Java代码在进行javac编译的时候，并不像C和C++那样有“连接”这一步骤，而是在虚拟机加载Class文件的时候进行动态链接。也就是说，在Class文件中不会保存各个方法、字段的最终内存布局信息。因此这些字段、方法的符号引用不经过运行期转换的话，无法得到真正的内存入口地址，也就无法直接被虚拟机使用。当虚拟机运行时，需要从常量池获得对应的符号引用，再在类创建时或运行时解析、翻译到具体的内存地址中

## 3.4 访问标识
访问标识（access_flag、访问标志、访问标记）参考 https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.1

![image.png](/images/jvm/15/11.png)
Flag Name        | Value  | Interpretation                                                                      |
| ---------------- | ------ | ----------------------------------------------------------------------------------- |
| `ACC_PUBLIC`     | 0x0001 | Declared `public`; may be accessed from outside its package.                        |
| `ACC_FINAL`      | 0x0010 | Declared `final`; no subclasses allowed.                                            |
| `ACC_SUPER`      | 0x0020 | Treat superclass methods specially when invoked by the *invokespecial* instruction. |
| `ACC_INTERFACE`  | 0x0200 | Is an interface, not a class.                                                       |
| `ACC_ABSTRACT`   | 0x0400 | Declared `abstract`; must not be instantiated.                                      |
| `ACC_SYNTHETIC`  | 0x1000 | Declared synthetic; not present in the source code.                                 |
| `ACC_ANNOTATION` | 0x2000 | Declared as an annotation type.                                                     |
| `ACC_ENUM`       | 0x4000 | Declared as an `enum` type.

在常量池后，紧跟着访问标记。该标记使用两个字节表示，用于识别一些类或者接口层次的访问信息，包括：这个Class是类还是接口；是否定义为public类型；是否定义为abstract类型；如果是类的话，是否被声明为final等。各种访问标记解释如下所示：

![image.png](/images/jvm/15/12.png)

- 类的访问权限通常为ACC_开头的常量
- 每一种类型的表示都是通过设置访问标记的32位中的特定位来实现的。比如，若是public final的类，则该标记为 **ACC_PUBLIC | ACC_FINAL**
- 使用ACC_SUPER可以让类更准确地定位到父类的方法super.method()，现代编译器都会设置并且使用这个标记
> 补充说明
- 带有ACC_INTERFACE标志的class文件标识的是接口而不是类，反之则表示的是类而不是接口
    - 如果一个class文件被设置了ACC_INTERFACE标志，那么同时也得设置ACC_ABSTRACT标志。同时它不能再设置ACC_FINAL、ACC_SUPER或ACC_ENUM标志
    - 如果没有设置ACC_INTERFACE标志，那么这个class文件可以具有上表中除ACC_ANNOTATION外的其他所有标志。当然，ACC_FINAL和ACC_ABSTRACT这类互斥的标志除外。这两个标志不得同时设置
- ACC_SUPER标志用于确定类或接口里面的invokespecial指令使用的是哪一种执行语义。**针对Java虚拟机指令集的编译器都应当设置这个标志**。对于Java SE 8及后续版本来说，无论class文件中这个标志的实际值是什么，也不管class文件的版本号是多少，Java虚拟机都认为每个class文件均设置了ACC_SUPER标志
    - ACC_SUPER标志是为了向后兼容由旧Java编译器所编译的代码而设计的。目前的ACC_SUPER标志在由JDK1.0.2之前的编译器所生成的access_flag中是没有确定含义的。如果设置了该标志，那么Oracle的Java虚拟机实现会将其忽略
- ACC_SYNTHETIC标志意味着该类或接口是由编译器生成的，而并不是由源代码生成的
- 注解类型必须设置ACC_ANNOTATION标志。如果设置了ACC_ANNOTATION标志，那么也必须设置ACC_INTERFACE标志
- ACC_ENUM标志标明该类或其父类为枚举类型
- 表中没有使用的access_flag标志是为未来扩充而预留的，这些预留的标志在编译器中应该设置为0，Java虚拟机实现也应该忽略它们

## 3.5 类索引、父类索引、接口索引集合

![image.png](/images/jvm/15/13.png)
- 在访问标志后，会指定该类的类别、父类类别以及实现的接口，格式如下：

    | 长度 | 含义                         |
    | ---- | ---------------------------- |
    | u2   | this_class                   |
    | u2   | super_class                  |
    | u2   | interfaces_count             |
    | u2   | interfaces[interfaces_count] |
- 这三项数据来确定这个类的继承关系
    - 类索引用于确定这个类的全限定名
    - 父类索引用于确定这个类的父类的全限定名。由于Java语言不允许多重继承，所以父类索引只有一个，除了java.lang.Object之外，所有的Java类都有父类，因此除了java.lang.Object外，所有的Java类的父类索引都不为0
    - 接口索引集合就是用来描述这个类实现了哪些接口，这些被实现的接口将按implements语句（如果这个类本身是一个接口，则应当是extends语句）后的接口顺序从左到右排列在接口索引集合中
### 3.5.1 this_class (类索引）
2字节无符号整数，指向常量池的索引。它提供了类的全限定名，如com/nasuf/jvm/Demo。this_class的值必须是对常量池表中某项的一个有效索引。常量池在这个索引处的成员必须为CONSTANT_Class_info类型结构体，该结构体表示这个class文件所定义的类或接口
### 3.5.2 interfaces
- 指向常量池索引集合，它提供了一个符号引用到所有已实现的接口
- 由于一个类可以实现多个接口，因此需要以数组形式保存多个接口的索引，表示接口的每个索引也是一个指向常量池的CONSTANT_Class（当然这里就必须是接口，而不是类）
#### 3.5.2.1 interface_count (接口计数器)
interface_count项的值表示当前类或接口的直接接口数量
#### 3.5.2.2 interface[] (接口索引集合)
interface[]中每个成员的值必须是对常量池表中某项的有效索引值，它的长度为interfaces_count。每个成员interfaces[i]必须为CONSTANT_Class_info结构，其中0 <= i < interfaces_count。在interfaces[]中，各成员所表示的接口顺序和对应的源代码中给定的接口顺序（从左到右）一样，即interfaces[0]对应的是源代码中最左边的接口

## 3.6 字段表集合

![image.png](/images/jvm/15/14.png)
- 用于描述接口或类中声明的变量。字段（field）包括类级变量以及实例级变量，但是不包括方法内部、代码块内部声明的局部变量
- 字段叫什么名字、字段被定义为什么数据类型，这些都是无法固定的，只能引用常量池中的常量来描述
- 它指向常量池索引集合、它描述了每个字段的完整信息。比如字段的标识符、访问修饰符（putlib、private或protected）、是类变量还是实例变量（static修饰符）、是否是常量（final修饰符）等
> 注意
- 字段表集合中不会列出从父类或者实现的接口中继承而来的字段，但有可能列出原本Java代码之中不存在的字段。比如在内部类中为了保持对外部类的访问性，会自动添加指向外部类实例的字段
- **在Java语言中字段是无法重载的。两个字段的数据类型、修饰符不管是否相同，都必须使用不一样的名称，但是对于字节码来讲，如果两个字段的描述符不一致，那字段重名是合法的**

### 3.6.1 字段计数器
fields_count（字段计数器）的值表示当前class文件fields表的成员个数。使用两个字节表示。fields表中每个成员都是一个field_info结构，用于表示该类或接口所声明的所有类字段或者实例字段，不包括方法内部声明的变量，也不包括从父类或父接口继承的那些字段

### 3.6.2 字段表
- fields字段表中的每个成员都必须是一个fields_info结构的数据项，用于表示当前类或接口中某个字段的完整描述
- 一个字段的信息包括如下这些信息。这些信息中，各个修饰符都是布尔值，要么有，要么没有
    - 作用域（public、private、protected修饰符）
    - 是实例变量还是类变量（static修饰符）
    - 可变性（final修饰符）
    - 并发可见性（volatile修饰符，是否强制从主内存读写）
    - 可否序列化（transient修饰符）
    - 字段数据类型（基本数据类型、对象、数组）
    - 字段名称
- 字段表结构

    | 类型           | 名称             | 含义       | 数量             |
    | -------------- | ---------------- | ---------- | ---------------- |
    | u2             | access_flags     | 访问标志   | 1                |
    | u2             | name_index       | 字段名索引 | 1                |
    | u2             | descriptor_index | 描述符索引 | 1                |
    | u2             | attributes_count | 属性计数器 | 1                |
    | attribute_info | attributes       | 属性集合   | attributes_count |
#### 3.6.2.1 字段访问标识
我们知道，一个字段可以被各种关键字去修饰，比如：作用域修饰符（public、private、protected）、static修饰符、final修饰符、volatile修饰符等。因此，其可像类的访问标志那样，使用一些标志来标记字段：

| 标志名称      | 标志值 | 含义                       |
| ------------- | ------ | -------------------------- |
| ACC_PUBLIC    | 0x0001 | 字段是否为public           |
| ACC_PRIVATE   | 0x0002 | 字段是否为private          |
| ACC_PROTECTED | 0x0004 | 字段是否为protected        |
| ACC_STATIC    | 0x0008 | 字段是否为static           |
| ACC_FINAL     | 0x0010 | 字段是否为final            |
| ACC_VOLATILE  | 0x0040 | 字段是否为volatile         |
| ACC_TRANSIENT | 0x0080 | 字段是否为transient        |
| ACC_SYNCHETIC | 0x1000 | 字段是否为由编译器自动生成 |
| ACC_ENUM      | 0x4000 | 字段是否为enum             |

#### 3.6.2.2 字段名索引
根据字段名索引的值，查询常量池中的指定索引项即可

#### 3.6.2.3 描述符索引
描述符的作用是描述字段的数据类型、方法的参数列表（包括数量、类型以及顺序）和返回值。根据描述符规则，基本数据类型（byte、char、double、float、int、long、short、boolean）及代表无返回值的void类型都用一个大写字符来表示，而对象则用字符L加对象的全限定名来表示

| 标志符 | 含义                                               |
| ------ | -------------------------------------------------- |
| B      | 基本数据类型byte                                   |
| C      | 基本数据类型char                                   |
| D      | 基本数据类型double                                 |
| F      | 基本数据类型float                                  |
| I      | 基本数据类型int                                    |
| J      | 基本数据类型long                                   |
| S      | 基本数据类型short                                  |
| Z      | 基本数据类型boolean                                |
| V      | 代表void类型                                       |
| L      | 对象类型，比如：Ljava/lang/Object;                 |
| [      | 数组类型，代表一维数组。比如: double[][][] is [[[D |

#### 3.6.2.4 属性表集合
- 一个字段还可能拥有一些属性，用于存储更多的额外信息。比如初始化信息，一些注释信息等。属性个数存放在attribute_count中，属性具体内容存放在attributes数组中
- 以常量属性为例，结构为：
    ```java
    ConstantValue_attribute {
        u2 attribute_name_index;
        u4 attribute_length;
        u2 constantvalue_index;
    }
    ```
    **说明：对于常量属性而言，attribute_length值恒为2**

假如代码修改如下：
```java
public class Demo {
    private final int num = 1;

    public int add() {
//        num = num + 2;
        return num;
    }
}
```
重新编译后查看字段的属性：

![image.png](/images/jvm/15/15.png)

## 3.7 方法表集合
方法表集合（methods）指向常量池索引集合，它完整描述了每个方法的签名
- 在字节码文件中，每一个method_info项都对应着一个类或者接口中的方法信息。比如方法的访问修饰符（public、private或protected），方法的返回值类型以及方法的参数信息等
- 如果这个方法不是抽象的或者不是native的，那么字节码中会体现出来
- 一方面，methods表只描述当前类或接口中声明的方法，不包括从父类或父接口继承的方法。另一方面，methods表有可能会出现由编译器自动添加的方法。最典型的便是编译器产生的方法信息（比如类或接口初始化方法\<clinit>()和实例初始化方法\<init>())
> 注意
> 在Java语言中，要重载（Overload）一个方法，除了要与原方法具有相同的简单名称之外，还要求必须拥有一个与原方法不同的特征签名，特征签名就是一个方法中各个参数在常量池中的字段符号引用的集合，也就是因为返回值不会包含在特征签名之中，因此Java语言里无法仅仅依靠返回值的不同来对一个已有方法进行重载。但在Class文件格式中，特征签名的范围更大一些，只要描述符不是完全一致的两个方法就可以共存。也就是说，如果两个方法有相同的名称和特征签名，但返回值不同，那么也是可以合法共存于同一个class文件中的，尽管Java语法规范并不允许在一个类或者接口中声明多个方法签名相同的方法，但是和Java语法规范相反，**字节码文件中却恰恰允许存放多个方法签名相同的方法，唯一的条件就是这些方法之间的返回值不能相同**

### 3.7.1 方法计数器
方法计数器（methods_count）的值表示当前class文件methods表的成员个数，使用两个字节表示。methods表中每个成员都是一个method_info结构

### 3.7.2 方法表
- 方法表中每个成员都必须是一个method_info结构，用于表示当前类或接口某个方法的完整描述。如果某个method_info结构的access_flags项既没有设置ACC_NATIVE标志也没有设置ACC_ABSTRACT标志，那么该结构中也应包含实现这个方法所用的Java虚拟机指令
- method_info结构可以表示类和接口中定义的所有方法，包括实例方法、类方法、实例初始化方法和类或接口初始化方法
- 方发表的结构实际跟字段表是一样的。结构如下：

    | 类型           | 名称             | 含义       | 数量             |
    | -------------- | ---------------- | ---------- | ---------------- |
    | u2             | access_flags     | 访问标志   | 1                |
    | u2             | name_index       | 字段名索引 | 1                |
    | u2             | descriptor_index | 描述符索引 | 1                |
    | u2             | attributes_count | 属性计数器 | 1                |
    | attribute_info | attributes       | 属性集合   | attributes_count |
    
## 3.8 属性表集合
- 方法表集合之后的属性表集合（attributes），指的是class文件所携带的辅助信息，比如该class文件的源文件的名称，以及任何带有RetentionPolicy.CLASS或者RetentionPolicy.RUNTIME的注解。这类信息通常被用于Java虚拟机的验证和运行，以及Java程序的调整，一般无需深入了解
- 此外，字段表、方法表都可以有自己的属性表，用于描述某些场景专有的信息
- 属性表集合的限制没有那么严格，不再要求各个属性表具有严格的顺序，并且只要不与已有的属性名重复，任何人实现的编译器都可以向属性表中写入自己定义的属性信息，但Java虚拟机运行时会忽略掉它不认识的属性

### 3.8.1 属性计数器
属性计数器（attributes_count）的值表示当前class文件属性表的成员个数。属性表中每一项都是一个attribute_info结构

### 3.8.2 属性表
属性表（attributes[])的每个项的值必须是attribute_info的结构。属性表的结构比较灵活，各种不同的属性只要满足以下结构即可：

#### 3.8.2.1 属性的通用格式
参考 https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.7 
```
attribute_info {
    u2 attribute_name_index;
    u4 attribute_length;
    u1 info[attribute_length];
}
```

| 类型 | 名称                 | 数量             | 含义       |
| ---- | -------------------- | ---------------- | ---------- |
| u2   | attribute_name_index | 1                | 属性名索引 |
| u4   | attribute_length     | 1                | 属性长度   |
| u1   | info                 | attribute_length | 属性表     |

即只需说明属性的名称以及占用位数的长度即可，属性表具体的结构可以去自定义

#### 3.8.2.2 属性类型
属性表实际上可以有很多类型，上面看到的Code属性只是其中一种，Java8中定义了23种属性，下面这些是虚拟机中预定义的属性 

| 属性名称                            | 使用位置           | 含义                                                         |
| ----------------------------------- | ------------------ | ------------------------------------------------------------ |
| Code                                | 方法表             | Java代码编译成的字节码指令                                   |
| ConstantValue                       | 字段表             | final关键字定义的常量池                                      |
| Deprecated                          | 类、方法、字段表   | 被声明为deprecated的方法和字段                               |
| Exceptions                          | 方法表             | 方法抛出的异常                                               |
| EnclosingMethod                     | 类文件             | 仅当一个类为局部类或者匿名类时才拥有这个属性，这个属性用于标识这个类所在的外围方法 |
| InnerClass                          | 类文件             | 内部类列表                                                   |
| LineNumberTable                     | Code属性           | Java源码的行号与字节码指令的对应关系                         |
| LocalVariableTable                  | Code属性           | 方法的局部变量描述                                           |
| StackMapTable                       | Code属性           | JDK1.6中新增的属性，供新的类型检查检验器检查和处理目标方法的局部变量和操作数有所需要的类是否匹配 |
| Signature                           | 类、方法表、字段表 | 用于支持泛型情况下的方法签名                                 |
| SourceFile                          | 类文件             | 记录源文件名称                                               |
| SourceDebugExtension                | 类文件             | 用于存储额外的调试信息                                       |
| Synthetic                           | 类、方法表、字段表 | 标志方法或字段为编译器自动生成的                             |
| LocalVariableTypeTable              | 类                 | 使用特征签名代替描述符，是为了引入泛型语法之后能描述泛型参数化类型而添加 |
| RuntimeVisibleAnnotations           | 类、方法表、字段表 | 为动态注解提供支持                                           |
| RuntimeInvisibleAnnotations         | 类、方法表、字段表 | 用于指明哪些注解是运行时不可见的                             |
| RuntimeVisibleParameterAnnotation   | 方法表             | 作用与RuntimeVisibleAnnotation属性类似，只不过作用对象为方法 |
| RuntimeInvisibleParameterAnnotation | 方法表             | 作用与RuntimeInvisibleAnnotation属性类型，作用对象为方法参数 |
| AnnotationDefault                   | 方法表             | 用于记录注解类元素的默认值                                   |
| BootstrapMethods                    | 类文件             | 用于保存invokeddynamic指令引用的引导方法限定符               |

#### 3.8.2.3 部分属性详解
> ConstantValue 属性
> ConstantValue属性表示一个常量字段的值。位于field_info结构的属性表中
```
ConstantValue_attribute {
    u2 attribute_name_index;
    u4 attribute_length;
    u2 constantvalue_index; // 字段值在常量池中的索引，常量池在该索引处的项给出该属性表示的常量值（例如，值是long型的，在常量池中便是CONSTANT_Long）
}
```
> Deprecated 属性
> Deprecated属性是在JDK1.1为了支持注释中的关键词@deprecated而引入的
```
Deprecated_attribute {
    u2 attribute_name_index;
    u4 attribute_length;
}
```
> Code属性
> Code属性就是存放方法体里面的代码。但是，并非所有方法表都有Code属性。像接口或者抽象方法，他们没有具体的方法体，因此也就不会有Code属性了。其具体结构如下：

| 类型           | 名称                   | 数量                   | 含义                     |
| -------------- | ---------------------- | ---------------------- | ------------------------ |
| u2             | attribute_name_index   | 1                      | 属性名索引               |
| u4             | attribute_length       | 1                      | 属性长度                 |
| u2             | max_stack              | 1                      | 操作数栈深度的最大值     |
| u2             | max_locals             | 1                      | 局部变量表所需的存续空间 |
| u4             | code_length            | 1                      | 字节码指令的长度         |
| u1             | code                   | code_length            | 存储字节码指令           |
| u2             | exception_table_length | 1                      | 异常表长度               |
| exception_info | exception_table        | exception_table_length | 异常表                   |
| u2             | attributes_count       | 1                      | 属性集合计数器           |
| attribute_info | attributes             | attributes_count       | 属性集合                 |

可以看到：Code属性表的前两项跟属性表是一致的，即Code属性表遵循属性表的结构，后面那些则是自定义的结构

> InnerClasses 属性
> 为了方便说明，特别定义一个表示类或接口的Class格式为C。如果C的常量池中包含某个CONSTANT_Class_info成员，且这个成员所表示的类或接口不属于任何一个包，那么C的ClassFile结构的属性表中就必须包含对应的InnerClasses属性。InnerClasses属性是在JDK1.1中为了支持内部类和内部接口而引入的，位于ClassFile结构的属性表

> LineNumberTable 属性
> LineNumberTable属性是可选变长属性，位于Code结构的属性表。该属性为用来描述Java源码行号与字节码行号之间的对应关系。这个属性可以用来在调试的时候定位代码执行的行数
- start_pc，即字节码行号；line_number，即Java源代码行号
在Code属性的属性表中，LineNumberTable属性可以按照任意顺序出现，此外，多个LineNumberTable属性可以共同表示一个行号在源文件中表示的内容，即LineNumberTable属性不需要与源文件的行一一对应。其属性表结构如下：
```
LineNumberTable_attribute {
    u2 attribute_name_index;
    u4 attribute_length;
    u2 line_number_table_length;
    {
        u2 start_pc;
        u2 line_number;
    } line_number_table[line_number_table_length];
}
```
> LocalVariableTable 属性
> 该属性是可选变长属性，位于Code属性的属性表中。它被调试器用于确定方法在执行过程中局部变量的信息。在Code属性的属性表中，LocalVariableTable属性可以按照任意顺序出现。Code属性中的每个局部变量最多只能有一个LocalVariableTable属性
- start pc + length 表示这个变量在字节码中的生命周期其实和结束的偏移位置（this生命周期从头0到结尾10）
- index就是这个变量在局部变量表中的槽位（槽位可复用）
- name就是变量名称
- Descriptor表示局部变量类型描述
该属性表结构如下：
```
LocalVariableTable_attribute {
    u2 attribute_name_index;
    u4 attribute_length;
    u2 local_variable_table_length;
    {
        u2 start_pc;
        u2 length;
        u2 name_index;
        u2 descriptor_index;
        u2 index;
    } local_variable_table[local_variable_table_length];
}
```
> Signature 属性
> 该属性是可选的定长属性，位于ClassFile.field_info或method_info结构的属性表中。在Java语言中，任何类、接口、初始化方法或成员的泛型签名如果包含了类型变量（Type Variables）或参数化类型（Parameterized Types），则Signature属性会为它记录泛型签名信息
> SourceFile 属性
> 该属性结构如下：

| 类型 | 名称                 | 数量 | 含义         |
| ---- | -------------------- | ---- | ------------ |
| u2   | attribute_name_index | 1    | 属性名索引   |
| u4   | attribute_length     | 1    | 属性长度     |
| u2   | sourcefile_index     | 1    | 源码文件索引 |

可以看到，其长度总是固定8个字节

# 4. javap指令解析Class文件
`javap`是JDK自带的反解析工具。它的作用就是根据class字节码文件，反解析出当前类对应的code区（字节码指令）、局部变量表、异常表和代码行偏移量映射表、常量池等信息。通过局部变量表，我们可以查看局部变量的作用域范围、所在槽位等信息，甚至可以看到槽位复用等信息

## 4.1 javac -g 操作
解析字节码文件得到的信息中，有些信息（如局部变量表、指令和代码行偏移量映射表、常量池中方法的参数名称等等）需要在使用javac编译成class文件时，指定参数才能输出

比如，直接执行`javac xx.java`，不会生成对应的局部变量表（**LocalVariableTable**）等信息，如果使用`javac -g xx.java`就可以生成所有相关信息了。如果使用eclipse或IDEA，则默认情况下，在编译时会帮你生成局部变量表、指令和代码行偏移映射表等信息

## 4.2 javap的用法
`javap`的用法格式：`javap <options> <classes>`，其中`classes`就是要反编译的class文件。在命令行中直接输入`javap`或`javap -help`可以看到javap的options有如下选项
```java
$ javap -help
用法: javap <options> <classes>
其中, 可能的选项包括:
  -help  --help  -?                输出此用法消息
  -version                         版本信息（其实是当前javap所在jdk的版本信息，不是class在哪个jdk下生成的版本信息）
  -v  -verbose                     输出附加信息（不包括私有信息）
  -l                               输出行号和本地变量表
  -public                          仅显示公共类和成员
  -protected                       显示受保护的/公共类和成员
  -package                         显示程序包/受保护的/公共类
                                   和成员 (默认)
  -p  -private                     显示所有类和成员
  -c                               对代码进行反汇编
  -s                               输出内部类型签名
  -sysinfo                         显示正在处理的类的
                                   系统信息 (路径, 大小, 日期, MD5 散列)
  -constants                       显示最终常量
  --module <模块>, -m <模块>       指定包含要反汇编的类的模块
  --module-path <路径>             指定查找应用程序模块的位置
  --system <jdk>                   指定查找系统模块的位置
  --class-path <路径>              指定查找用户类文件的位置
  -classpath <路径>                指定查找用户类文件的位置
  -cp <路径>                       指定查找用户类文件的位置
  -bootclasspath <路径>            覆盖引导类文件的位置

GNU 样式的选项可使用 = (而非空白) 来分隔选项名称
及其值。

每个类可由其文件名, URL 或其
全限定类名指定。示例:
   path/to/MyClass.class
   jar:file:///path/to/MyJar.jar!/mypkg/MyClass.class
   java.lang.Object
```
> javap -public
```java
$ javap -public JavapTest.class
Compiled from "JavapTest.java"
public class com.nasuf.jdk8.JavapTest {
  public java.lang.String info;
  public static final int COUNT;
  public com.nasuf.jdk8.JavapTest();
  public void showInfo();
}
```
> javap -protected
```java
$ javap -protected JavapTest.class
Compiled from "JavapTest.java"
public class com.nasuf.jdk8.JavapTest {
  protected char gender;
  public java.lang.String info;
  public static final int COUNT;
  public com.nasuf.jdk8.JavapTest();
  protected char showGender();
  public void showInfo();
}
```
> javap -private
```java
$ javap -private JavapTest.class
Compiled from "JavapTest.java"
public class com.nasuf.jdk8.JavapTest {
  private int num;
  boolean flag;
  protected char gender;
  public java.lang.String info;
  public static final int COUNT;
  public com.nasuf.jdk8.JavapTest();
  private com.nasuf.jdk8.JavapTest(boolean);
  int getNum(int);
  protected char showGender();
  public void showInfo();
  static {};
}
```
> javap -package
```java
$ javap -package JavapTest.class
Compiled from "JavapTest.java"
public class com.nasuf.jdk8.JavapTest {
  boolean flag;
  protected char gender;
  public java.lang.String info;
  public static final int COUNT;
  public com.nasuf.jdk8.JavapTest();
  int getNum(int);
  protected char showGender();
  public void showInfo();
  static {};
}
```
> javap -sysinfo
```java
$ javap -sysinfo JavapTest.class
Classfile /Users/nasuf/Project/JvmNotesCode/out/production/jdk1.8-module/com/nasuf/jdk8/JavapTest.class
  Last modified 2021年8月10日; size 1278 bytes
  MD5 checksum 8b7eba93745a5d4f9a3a8dc972d1e8f4
  Compiled from "JavapTest.java"
public class com.nasuf.jdk8.JavapTest {
  boolean flag;
  protected char gender;
  public java.lang.String info;
  public static final int COUNT;
  public com.nasuf.jdk8.JavapTest();
  int getNum(int);
  protected char showGender();
  public void showInfo();
  static {};
}
```
> javap -constants
```java
$ javap -constants JavapTest.class
Compiled from "JavapTest.java"
public class com.nasuf.jdk8.JavapTest {
  boolean flag;
  protected char gender;
  public java.lang.String info;
  public static final int COUNT = 1;
  public com.nasuf.jdk8.JavapTest();
  int getNum(int);
  protected char showGender();
  public void showInfo();
  static {};
}
```
> javap -s
```java
$ javap -s JavapTest.class 
Compiled from "JavapTest.java"
public class com.nasuf.jdk8.JavapTest {
  boolean flag;
    descriptor: Z
  protected char gender;
    descriptor: C
  public java.lang.String info;
    descriptor: Ljava/lang/String;
  public static final int COUNT;
    descriptor: I
  public com.nasuf.jdk8.JavapTest();
    descriptor: ()V

  int getNum(int);
    descriptor: (I)I

  protected char showGender();
    descriptor: ()C

  public void showInfo();
    descriptor: ()V

  static {};
    descriptor: ()V
}
```
> javap -l
```java
$ javap -l JavapTest.class
Compiled from "JavapTest.java"
public class com.nasuf.jdk8.JavapTest {
  boolean flag;

  protected char gender;

  public java.lang.String info;

  public static final int COUNT;

  public com.nasuf.jdk8.JavapTest();
    LineNumberTable:
      line 18: 0
      line 15: 4
      line 18: 10
    LocalVariableTable:
      Start  Length  Slot  Name   Signature
          0      11     0  this   Lcom/nasuf/jdk8/JavapTest;

  int getNum(int);
    LineNumberTable:
      line 25: 0
    LocalVariableTable:
      Start  Length  Slot  Name   Signature
          0       7     0  this   Lcom/nasuf/jdk8/JavapTest;
          0       7     1     i   I

  protected char showGender();
    LineNumberTable:
      line 29: 0
    LocalVariableTable:
      Start  Length  Slot  Name   Signature
          0       5     0  this   Lcom/nasuf/jdk8/JavapTest;

  public void showInfo();
    LineNumberTable:
      line 33: 0
      line 34: 3
      line 35: 30
    LocalVariableTable:
      Start  Length  Slot  Name   Signature
          0      31     0  this   Lcom/nasuf/jdk8/JavapTest;
          3      28     1     i   I

  static {};
    LineNumberTable:
      line 11: 0
      line 12: 3
    LocalVariableTable:
      Start  Length  Slot  Name   Signature
}
```
> javap -c
```java
$ javap -c JavapTest.class
Compiled from "JavapTest.java"
public class com.nasuf.jdk8.JavapTest {
  boolean flag;

  protected char gender;

  public java.lang.String info;

  public static final int COUNT;

  public com.nasuf.jdk8.JavapTest();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: aload_0
       5: ldc           #2                  // String java
       7: putfield      #3                  // Field info:Ljava/lang/String;
      10: return

  int getNum(int);
    Code:
       0: aload_0
       1: getfield      #5                  // Field num:I
       4: iload_1
       5: iadd
       6: ireturn

  protected char showGender();
    Code:
       0: aload_0
       1: getfield      #6                  // Field gender:C
       4: ireturn

  public void showInfo();
    Code:
       0: bipush        10
       2: istore_1
       3: getstatic     #7                  // Field java/lang/System.out:Ljava/io/PrintStream;
       6: new           #8                  // class java/lang/StringBuilder
       9: dup
      10: invokespecial #9                  // Method java/lang/StringBuilder."<init>":()V
      13: aload_0
      14: getfield      #3                  // Field info:Ljava/lang/String;
      17: invokevirtual #10                 // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
      20: iload_1
      21: invokevirtual #11                 // Method java/lang/StringBuilder.append:(I)Ljava/lang/StringBuilder;
      24: invokevirtual #12                 // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
      27: invokevirtual #13                 // Method java/io/PrintStream.println:(Ljava/lang/String;)V
      30: return

  static {};
    Code:
       0: ldc           #14                 // String www.atguigu.com
       2: astore_0
       3: return
}
```
> javap -v
```java
$ javap -v JavapTest.class
Classfile /Users/nasuf/Project/JvmNotesCode/out/production/jdk1.8-module/com/nasuf/jdk8/JavapTest.class
  Last modified 2021年8月10日; size 1278 bytes
  MD5 checksum 8b7eba93745a5d4f9a3a8dc972d1e8f4
  Compiled from "JavapTest.java"
public class com.nasuf.jdk8.JavapTest
  minor version: 0
  major version: 52
  flags: (0x0021) ACC_PUBLIC, ACC_SUPER
  this_class: #15                         // com/nasuf/jdk8/JavapTest
  super_class: #16                        // java/lang/Object
  interfaces: 0, fields: 5, methods: 6, attributes: 1
Constant pool:
   #1 = Methodref          #16.#45        // java/lang/Object."<init>":()V
   #2 = String             #46            // java
   #3 = Fieldref           #15.#47        // com/nasuf/jdk8/JavapTest.info:Ljava/lang/String;
   #4 = Fieldref           #15.#48        // com/nasuf/jdk8/JavapTest.flag:Z
   #5 = Fieldref           #15.#49        // com/nasuf/jdk8/JavapTest.num:I
   #6 = Fieldref           #15.#50        // com/nasuf/jdk8/JavapTest.gender:C
   #7 = Fieldref           #51.#52        // java/lang/System.out:Ljava/io/PrintStream;
   #8 = Class              #53            // java/lang/StringBuilder
   #9 = Methodref          #8.#45         // java/lang/StringBuilder."<init>":()V
  #10 = Methodref          #8.#54         // java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
  #11 = Methodref          #8.#55         // java/lang/StringBuilder.append:(I)Ljava/lang/StringBuilder;
  #12 = Methodref          #8.#56         // java/lang/StringBuilder.toString:()Ljava/lang/String;
  #13 = Methodref          #57.#58        // java/io/PrintStream.println:(Ljava/lang/String;)V
  #14 = String             #59            // www.atguigu.com
  #15 = Class              #60            // com/nasuf/jdk8/JavapTest
  #16 = Class              #61            // java/lang/Object
  #17 = Utf8               num
  #18 = Utf8               I
  #19 = Utf8               flag
  #20 = Utf8               Z
  #21 = Utf8               gender
  #22 = Utf8               C
  #23 = Utf8               info
  #24 = Utf8               Ljava/lang/String;
  #25 = Utf8               COUNT
  #26 = Utf8               ConstantValue
  #27 = Integer            1
  #28 = Utf8               <init>
  #29 = Utf8               ()V
  #30 = Utf8               Code
  #31 = Utf8               LineNumberTable
  #32 = Utf8               LocalVariableTable
  #33 = Utf8               this
  #34 = Utf8               Lcom/nasuf/jdk8/JavapTest;
  #35 = Utf8               (Z)V
  #36 = Utf8               getNum
  #37 = Utf8               (I)I
  #38 = Utf8               i
  #39 = Utf8               showGender
  #40 = Utf8               ()C
  #41 = Utf8               showInfo
  #42 = Utf8               <clinit>
  #43 = Utf8               SourceFile
  #44 = Utf8               JavapTest.java
  #45 = NameAndType        #28:#29        // "<init>":()V
  #46 = Utf8               java
  #47 = NameAndType        #23:#24        // info:Ljava/lang/String;
  #48 = NameAndType        #19:#20        // flag:Z
  #49 = NameAndType        #17:#18        // num:I
  #50 = NameAndType        #21:#22        // gender:C
  #51 = Class              #62            // java/lang/System
  #52 = NameAndType        #63:#64        // out:Ljava/io/PrintStream;
  #53 = Utf8               java/lang/StringBuilder
  #54 = NameAndType        #65:#66        // append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
  #55 = NameAndType        #65:#67        // append:(I)Ljava/lang/StringBuilder;
  #56 = NameAndType        #68:#69        // toString:()Ljava/lang/String;
  #57 = Class              #70            // java/io/PrintStream
  #58 = NameAndType        #71:#72        // println:(Ljava/lang/String;)V
  #59 = Utf8               www.atguigu.com
  #60 = Utf8               com/nasuf/jdk8/JavapTest
  #61 = Utf8               java/lang/Object
  #62 = Utf8               java/lang/System
  #63 = Utf8               out
  #64 = Utf8               Ljava/io/PrintStream;
  #65 = Utf8               append
  #66 = Utf8               (Ljava/lang/String;)Ljava/lang/StringBuilder;
  #67 = Utf8               (I)Ljava/lang/StringBuilder;
  #68 = Utf8               toString
  #69 = Utf8               ()Ljava/lang/String;
  #70 = Utf8               java/io/PrintStream
  #71 = Utf8               println
  #72 = Utf8               (Ljava/lang/String;)V
{
  boolean flag;
    descriptor: Z
    flags: (0x0000)

  protected char gender;
    descriptor: C
    flags: (0x0004) ACC_PROTECTED

  public java.lang.String info;
    descriptor: Ljava/lang/String;
    flags: (0x0001) ACC_PUBLIC

  public static final int COUNT;
    descriptor: I
    flags: (0x0019) ACC_PUBLIC, ACC_STATIC, ACC_FINAL
    ConstantValue: int 1

  public com.nasuf.jdk8.JavapTest();
    descriptor: ()V
    flags: (0x0001) ACC_PUBLIC
    Code:
      stack=2, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: aload_0
         5: ldc           #2                  // String java
         7: putfield      #3                  // Field info:Ljava/lang/String;
        10: return
      LineNumberTable:
        line 18: 0
        line 15: 4
        line 18: 10
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      11     0  this   Lcom/nasuf/jdk8/JavapTest;

  int getNum(int);
    descriptor: (I)I
    flags: (0x0000)
    Code:
      stack=2, locals=2, args_size=2
         0: aload_0
         1: getfield      #5                  // Field num:I
         4: iload_1
         5: iadd
         6: ireturn
      LineNumberTable:
        line 25: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       7     0  this   Lcom/nasuf/jdk8/JavapTest;
            0       7     1     i   I

  protected char showGender();
    descriptor: ()C
    flags: (0x0004) ACC_PROTECTED
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: getfield      #6                  // Field gender:C
         4: ireturn
      LineNumberTable:
        line 29: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       5     0  this   Lcom/nasuf/jdk8/JavapTest;

  public void showInfo();
    descriptor: ()V
    flags: (0x0001) ACC_PUBLIC
    Code:
      stack=3, locals=2, args_size=1
         0: bipush        10
         2: istore_1
         3: getstatic     #7                  // Field java/lang/System.out:Ljava/io/PrintStream;
         6: new           #8                  // class java/lang/StringBuilder
         9: dup
        10: invokespecial #9                  // Method java/lang/StringBuilder."<init>":()V
        13: aload_0
        14: getfield      #3                  // Field info:Ljava/lang/String;
        17: invokevirtual #10                 // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
        20: iload_1
        21: invokevirtual #11                 // Method java/lang/StringBuilder.append:(I)Ljava/lang/StringBuilder;
        24: invokevirtual #12                 // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
        27: invokevirtual #13                 // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        30: return
      LineNumberTable:
        line 33: 0
        line 34: 3
        line 35: 30
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      31     0  this   Lcom/nasuf/jdk8/JavapTest;
            3      28     1     i   I

  static {};
    descriptor: ()V
    flags: (0x0008) ACC_STATIC
    Code:
      stack=1, locals=1, args_size=0
         0: ldc           #14                 // String www.atguigu.com
         2: astore_0
         3: return
      LineNumberTable:
        line 11: 0
        line 12: 3
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
}
SourceFile: "JavapTest.java"
```
## 4.3 javap -v -p 字节码解析详解
```java
$ javap -v -p JavapTest.class
Classfile /Users/nasuf/Project/JvmNotesCode/out/production/jdk1.8-module/com/nasuf/jdk8/JavapTest.class  // 字节码文件所属的路径
  Last modified 2021年8月10日; size 1278 bytes  // 最后修改时间，字节码问价大小
  MD5 checksum 8b7eba93745a5d4f9a3a8dc972d1e8f4   // MD5散列值
  Compiled from "JavapTest.java"
public class com.nasuf.jdk8.JavapTest
  minor version: 0  // 副版本
  major version: 52 // 主版本
  flags: (0x0021) ACC_PUBLIC, ACC_SUPER  // 访问标识
  this_class: #15                         // com/nasuf/jdk8/JavapTest
  super_class: #16                        // java/lang/Object
  interfaces: 0, fields: 5, methods: 6, attributes: 1
Constant pool:  // 常量池信息
   #1 = Methodref          #16.#45        // java/lang/Object."<init>":()V
   #2 = String             #46            // java
   #3 = Fieldref           #15.#47        // com/nasuf/jdk8/JavapTest.info:Ljava/lang/String;
   #4 = Fieldref           #15.#48        // com/nasuf/jdk8/JavapTest.flag:Z
   #5 = Fieldref           #15.#49        // com/nasuf/jdk8/JavapTest.num:I
   #6 = Fieldref           #15.#50        // com/nasuf/jdk8/JavapTest.gender:C
   #7 = Fieldref           #51.#52        // java/lang/System.out:Ljava/io/PrintStream;
   #8 = Class              #53            // java/lang/StringBuilder
   #9 = Methodref          #8.#45         // java/lang/StringBuilder."<init>":()V
  #10 = Methodref          #8.#54         // java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
  #11 = Methodref          #8.#55         // java/lang/StringBuilder.append:(I)Ljava/lang/StringBuilder;
  #12 = Methodref          #8.#56         // java/lang/StringBuilder.toString:()Ljava/lang/String;
  #13 = Methodref          #57.#58        // java/io/PrintStream.println:(Ljava/lang/String;)V
  #14 = String             #59            // www.atguigu.com
  #15 = Class              #60            // com/nasuf/jdk8/JavapTest
  #16 = Class              #61            // java/lang/Object
  #17 = Utf8               num
  #18 = Utf8               I
  #19 = Utf8               flag
  #20 = Utf8               Z
  #21 = Utf8               gender
  #22 = Utf8               C
  #23 = Utf8               info
  #24 = Utf8               Ljava/lang/String;
  #25 = Utf8               COUNT
  #26 = Utf8               ConstantValue
  #27 = Integer            1
  #28 = Utf8               <init>
  #29 = Utf8               ()V
  #30 = Utf8               Code
  #31 = Utf8               LineNumberTable
  #32 = Utf8               LocalVariableTable
  #33 = Utf8               this
  #34 = Utf8               Lcom/nasuf/jdk8/JavapTest;
  #35 = Utf8               (Z)V
  #36 = Utf8               getNum
  #37 = Utf8               (I)I
  #38 = Utf8               i
  #39 = Utf8               showGender
  #40 = Utf8               ()C
  #41 = Utf8               showInfo
  #42 = Utf8               <clinit>
  #43 = Utf8               SourceFile
  #44 = Utf8               JavapTest.java
  #45 = NameAndType        #28:#29        // "<init>":()V
  #46 = Utf8               java
  #47 = NameAndType        #23:#24        // info:Ljava/lang/String;
  #48 = NameAndType        #19:#20        // flag:Z
  #49 = NameAndType        #17:#18        // num:I
  #50 = NameAndType        #21:#22        // gender:C
  #51 = Class              #62            // java/lang/System
  #52 = NameAndType        #63:#64        // out:Ljava/io/PrintStream;
  #53 = Utf8               java/lang/StringBuilder
  #54 = NameAndType        #65:#66        // append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
  #55 = NameAndType        #65:#67        // append:(I)Ljava/lang/StringBuilder;
  #56 = NameAndType        #68:#69        // toString:()Ljava/lang/String;
  #57 = Class              #70            // java/io/PrintStream
  #58 = NameAndType        #71:#72        // println:(Ljava/lang/String;)V
  #59 = Utf8               www.atguigu.com
  #60 = Utf8               com/nasuf/jdk8/JavapTest
  #61 = Utf8               java/lang/Object
  #62 = Utf8               java/lang/System
  #63 = Utf8               out
  #64 = Utf8               Ljava/io/PrintStream;
  #65 = Utf8               append
  #66 = Utf8               (Ljava/lang/String;)Ljava/lang/StringBuilder;
  #67 = Utf8               (I)Ljava/lang/StringBuilder;
  #68 = Utf8               toString
  #69 = Utf8               ()Ljava/lang/String;
  #70 = Utf8               java/io/PrintStream
  #71 = Utf8               println
  #72 = Utf8               (Ljava/lang/String;)V
{
  /////////////////////// 以下为字段表集合信息 //////////////////
  private int num;  // 字段名
    descriptor: I  // 字段描述符：字段类型
    flags: (0x0002) ACC_PRIVATE // 字段的访问标识

  boolean flag;
    descriptor: Z
    flags: (0x0000)

  protected char gender;
    descriptor: C
    flags: (0x0004) ACC_PROTECTED

  public java.lang.String info;
    descriptor: Ljava/lang/String;
    flags: (0x0001) ACC_PUBLIC

  public static final int COUNT;
    descriptor: I
    flags: (0x0019) ACC_PUBLIC, ACC_STATIC, ACC_FINAL
    ConstantValue: int 1  // 常量字段的属性：ConstantValue

 //////////////////// 以下为方法表集合信息 ////////////////////
  public com.nasuf.jdk8.JavapTest();  // 构造器1的信息
    descriptor: ()V
    flags: (0x0001) ACC_PUBLIC
    Code:
      stack=2, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: aload_0
         5: ldc           #2                  // String java
         7: putfield      #3                  // Field info:Ljava/lang/String;
        10: return
      LineNumberTable:
        line 18: 0
        line 15: 4
        line 18: 10
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      11     0  this   Lcom/nasuf/jdk8/JavapTest;

  private com.nasuf.jdk8.JavapTest(boolean);  // 构造器2的信息
    descriptor: (Z)V
    flags: (0x0002) ACC_PRIVATE
    Code:
      stack=2, locals=2, args_size=2
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: aload_0
         5: ldc           #2                  // String java
         7: putfield      #3                  // Field info:Ljava/lang/String;
        10: aload_0
        11: iload_1
        12: putfield      #4                  // Field flag:Z
        15: return
      LineNumberTable:
        line 20: 0
        line 15: 4
        line 21: 10
        line 22: 15
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      16     0  this   Lcom/nasuf/jdk8/JavapTest;
            0      16     1  flag   Z

  int getNum(int);
    descriptor: (I)I
    flags: (0x0000)
    Code:
      stack=2, locals=2, args_size=2
         0: aload_0
         1: getfield      #5                  // Field num:I
         4: iload_1
         5: iadd
         6: ireturn
      LineNumberTable:
        line 25: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       7     0  this   Lcom/nasuf/jdk8/JavapTest;
            0       7     1     i   I

  protected char showGender();
    descriptor: ()C
    flags: (0x0004) ACC_PROTECTED
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: getfield      #6                  // Field gender:C
         4: ireturn
      LineNumberTable:
        line 29: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       5     0  this   Lcom/nasuf/jdk8/JavapTest;

  public void showInfo();
    descriptor: ()V  // 方法描述符：方法的形参列表、返回值类型
    flags: (0x0001) ACC_PUBLIC  // 方法的访问标识
    Code:  // 方法的Code属性
      stack=3, locals=2, args_size=1  // stack：操作数栈的最大深度；locals：局部变量表的长度；args_size：方法接收参数的个数
   // 偏移量:操作码      操作数   
         0: bipush        10
         2: istore_1
         3: getstatic     #7                  // Field java/lang/System.out:Ljava/io/PrintStream;
         6: new           #8                  // class java/lang/StringBuilder
         9: dup
        10: invokespecial #9                  // Method java/lang/StringBuilder."<init>":()V
        13: aload_0
        14: getfield      #3                  // Field info:Ljava/lang/String;
        17: invokevirtual #10                 // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
        20: iload_1
        21: invokevirtual #11                 // Method java/lang/StringBuilder.append:(I)Ljava/lang/StringBuilder;
        24: invokevirtual #12                 // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
        27: invokevirtual #13                 // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        30: return
      // 行号表：指明字节码指令的偏移量与Java源程序中代码的行号的一一对应关系
      LineNumberTable:
        line 33: 0
        line 34: 3
        line 35: 30
      // 局部变量表：描述内部局部变量的相关信息
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      31     0  this   Lcom/nasuf/jdk8/JavapTest;
            3      28     1     i   I

  static {};
    descriptor: ()V
    flags: (0x0008) ACC_STATIC
    Code:
      stack=1, locals=1, args_size=0
         0: ldc           #14                 // String www.atguigu.com
         2: astore_0
         3: return
      LineNumberTable:
        line 11: 0
        line 12: 3
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
}
SourceFile: "JavapTest.java"  // 附加属性：当前字节码文件对应的源程序文件名
```
## 4.4 总结
- 通过javap指令可以查看一个java类反汇编得到的Class文件版本号、常量池、访问标识、变量表、指令代码行号表等信息。不显示类索引、父类索引、接口索引集合、\<clinit>()、\<init>()等结构
- 通过对前面例子代码反汇编文件的简单分析，可以发现一个方法的执行通常会设计下面几块内存的操作：
    - Java栈：局部变量表、操作数栈
    - Java堆：通过对象的地址引用去操作
    - 常量池
    - 其他如桢数据区、方法区的剩余部分等情况，测试中没有显示出来，这里说明一下
- 通常，我们比较关注的是Java类中每个方法的反汇编中的指令操作过程，这些指令都是顺序执行的，可以参考官方文档查看每个指令的含义 https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-6.html