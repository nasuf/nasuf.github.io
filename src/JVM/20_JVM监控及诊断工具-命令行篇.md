# 1. jps：查看正在运行的Java进程
## 1.1 基本情况
jps（Java Process Status）：显式指定系统内所有的HotSpot虚拟机进程（查看虚拟机进程信息），可用于查询正在运行的虚拟机进程。  
**说明：对于本地虚拟机进程来说，进程的本地虚拟机ID与操作系统的进程ID是一致的，唯一的**
> 测试
```java
nasuf@songtaodeMacBook-Pro jps
42064 KotlinCompileDaemon
6562
42115 Launcher
42739 Jps
28612 RemoteMavenServer36
```
## 1.2 基本语法
```
jps [options] [hostid]
```
我们还可以通过追加参数，来打印额外的信息。  

### 1.2.1 options参数
- -q：仅仅表示LVMID（local virtual machine id），即本地虚拟机唯一id。不显示主类的名称等
```java
nasuf@songtaodeMacBook-Pro jps -q
42064
6562
42115
28612
42751
```
- -l：输出应用程序主类的全类名，或者，如果进程执行的是jar包，则输出jar完成路径
```java
nasuf@songtaodeMacBook-Pro jps -l
42064 org.jetbrains.kotlin.daemon.KotlinCompileDaemon
6562
42115 org.jetbrains.jps.cmdline.Launcher
42771 sun.tools.jps.Jps
28612 org.jetbrains.idea.maven.server.RemoteMavenServer36
```
- -m：输出虚拟机进程启动时传递给主类的`main()`的参数
```java
nasuf@songtaodeMacBook-Pro jps -m
42064 KotlinCompileDaemon --daemon-runFilesPath /Users/nasuf/Library/Application Support/kotlin/daemon --daemon-autoshutdownIdleSeconds=7200 --daemon-compilerClasspath /Applications/IntelliJ IDEA CE.app/Contents/plugins/Kotlin/kotlinc/lib/kotlin-compiler.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_102.jdk/Contents/Home/lib/tools.jar:/Applications/IntelliJ IDEA CE.app/Contents/plugins/Kotlin/kotlinc/lib/kotlin-daemon.jar
6562
42115 Launcher /Applications/IntelliJ IDEA CE.app/Contents/plugins/java/lib/aether-transport-http-1.1.0.jar:/Applications/IntelliJ IDEA CE.app/Contents/lib/util.jar:/Applications/IntelliJ IDEA CE.app/Contents/plugins/java/lib/maven-repository-metadata-3.3.9.jar:/Applications/IntelliJ IDEA CE.app/Contents/plugins/java/lib/plexus-component-annotations-1.6.jar:/Applications/IntelliJ IDEA CE.app/Contents/lib/jna-platform.jar:/Applications/IntelliJ IDEA CE.app/Contents/plugins/java/lib/aether-dependency-resolver.jar:/Applications/IntelliJ IDEA CE.app/Contents/lib/log4j.jar:/Applications/IntelliJ IDEA CE.app/Contents/plugins/java/lib/aether-spi-1.1.0.jar:/Applications/IntelliJ IDEA CE.app/Contents/plugins/java/lib/jps-builders-6.jar:/Applications/IntelliJ IDEA CE.app/Contents/plugins/java/lib/commons-lang3-3.4.jar:/Applications/IntelliJ IDEA CE.app/Contents/lib/netty-codec-4.1.41.Final.jar:/Applications/IntelliJ IDEA CE.app/Contents/plugins/java/lib/aether-impl-1.1.0.jar:/Applications/IntelliJ I
42788 Jps -m
28612 RemoteMavenServer36
```
- -v：列出虚拟机进程启动时的jvm参数，比如`-Xms20m -Xmx50m`是启动程序指定的jvm参数
```java
nasuf@songtaodeMacBook-Pro jps -v
42064 KotlinCompileDaemon -Djava.awt.headless=true -Djava.rmi.server.hostname=127.0.0.1 -Xmx700m -Dkotlin.incremental.compilation=true -Dkotlin.incremental.compilation.js=true -ea
6562  -Xms128m -Xmx2048m -XX:ReservedCodeCacheSize=240m -XX:+UseConcMarkSweepGC -XX:SoftRefLRUPolicyMSPerMB=50 -ea -XX:CICompilerCount=2 -Dsun.io.useCanonPrefixCache=false -Djava.net.preferIPv4Stack=true -Djdk.http.auth.tunneling.disabledSchemes="" -XX:+HeapDumpOnOutOfMemoryError -XX:-OmitStackTraceInFastThrow -Djdk.attach.allowAttachSelf=true -Dkotlinx.coroutines.debug=off -Djdk.module.illegalAccess.silent=true -XX:+UseCompressedOops -Dfile.encoding=UTF-8 -XX:ErrorFile=/Users/nasuf/java_error_in_idea_%p.log -XX:HeapDumpPath=/Users/nasuf/java_error_in_idea.hprof -Djb.vmOptionsFile=/Users/nasuf/Library/Preferences/IdeaIC2019.3/idea.vmoptions -Didea.paths.selector=IdeaIC2019.3 -Didea.executable=idea -Didea.platform.prefix=Idea -Didea.home.path=/Applications/IntelliJ IDEA CE.app/Contents
42115 Launcher -Xmx700m -Djava.awt.headless=true -Djava.endorsed.dirs="" -Djdt.compiler.useSingleThread=true -Dpreload.project.path=/Users/nasuf/Project/LeetCode -Dpreload.config.path=/Users/nasuf/Library/Preferences/IdeaIC2019.3/options -Dcompile.parallel=false -Drebuild.on.dependency.change=true -Djava.net.preferIPv4Stack=true -Dio.netty.initialSeedUniquifier=-1029266495424193827 -Dfile.encoding=UTF-8 -Duser.language=zh -Duser.country=CN -Didea.paths.selector=IdeaIC2019.3 -Didea.home.path=/Applications/IntelliJ IDEA CE.app/Contents -Didea.config.path=/Users/nasuf/Library/Preferences/IdeaIC2019.3 -Didea.plugins.path=/Users/nasuf/Library/Application Support/IdeaIC2019.3 -Djps.log.dir=/Users/nasuf/Library/Logs/IdeaIC2019.3/build-log -Djps.fallback.jdk.home=/Applications/IntelliJ IDEA CE.app/Contents/jbr/Contents/Home -Djps.fallback.jdk.version=11.0.5 -Dio.netty.noUnsafe=true -Djava.io.tmpdir=/Users/nasuf/Library/Caches/IdeaIC2019.3/compile-server/leetcode_744308d5/_temp_ -Djps.backward.ref.index.builder=true -Dkotlin.increme
42803 Jps -Dapplication.home=/Library/Java/JavaVirtualMachines/jdk1.8.0_202.jdk/Contents/Home -Xms8m
28612 RemoteMavenServer36 -Djava.awt.headless=true -Dmaven.defaultProjectBuilder.disableGlobalModelCache=true -Xmx768m -Didea.maven.embedder.version=3.6.1 -Dmaven.ext.class.path=/Applications/IntelliJ IDEA CE.app/Contents/plugins/maven/lib/maven-event-listener.jar -Dfile.encoding=UTF-8
```
> 补充说明
> 如果某Java进程关闭了默认开启的`UserPerfData`参数（即使用参数`-XX:-UsePerfData`），那么jps命令（以及下面介绍的jstat）将无法探知该Java进程**

### 1.2.2 hostid参数

RMI注册表中注册的主机名。如果想要远程监控主机上的Java程序，需要安装jstatd。对于具有更严格的安全实践的网络场所而言，可能使用一个自定义的策略文件来显示对特定的可信主机或网络的访问，尽管这种技术容易受到IP地址欺诈攻击  

如果安全问题无法使用一个定制的策略文件来处理，那么最安全的操作是不运行jstatd服务器，而是在本地使用jstat和jps工具

# 2. jstat：查看JVM统计信息
## 2.1 基本情况
jstat（JVM Statistics Monitoring Tool）：用于监视虚拟机各种运行状态信息的命令行工具。它可以显示本地或者远程虚拟机进程中的类装载、内存、垃圾收集、JIT编译等运行数据  

在没有GUI图形界面，只提供了纯文本控制台环境的服务器上，它将是运行期定位虚拟机性能问题的首选工具。常用于检测垃圾回收问题以及内存泄漏问题

官方文档：https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jstat.html
## 2.2 基本语法
```java
jstat -<option> [-t] [-h<lines>] <vmid> [<interval> [<count>]]
```
查看命令相关参数：`jstat -h`或`jstat -help`  
针对以下程序进行测试：
```java
package com.nasuf.jdk8;

import java.util.Scanner;

public class ScannerTest {

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        String info = scanner.next();
    }
    
}
```
```java
nasuf@songtaodeMacBook-Pro jps
6562
42115 Launcher
61334 Launcher
61335 ScannerTest
61354 Jps
```
ScannerTest执行进程id为**61335**  

### 2.2.1 option参数
> 类装载相关
- -class：显示ClassLoader的相关信息：类的装载、卸载数量、总空间、类装载所消耗的时间等
```java
nasuf@songtaodeMacBook-Pro jstat -class 61335
Loaded  Bytes  Unloaded  Bytes     Time
   604  1225.6        0     0.0       0.08
```
> 垃圾回收相关
- -gc：显示与GC相关的堆信息。包括Eden区、两个Survivor区、老年代、永久代等的容量，已用空间、GC时间合计等信息
```java
nasuf@songtaodeMacBook-Pro jstat -gc 61335
 S0C    S1C    S0U    S1U      EC       EU        OC         OU       MC     MU    CCSC   CCSU   YGC     YGCT    FGC    FGCT     GCT
    10752.0 10752.0  0.0    0.0   65536.0   5243.4   175104.0     0.0     4480.0 774.3  384.0   75.9       0    0.000   0      0.000    0.000
```
    新生代相关:
        S0C是第一个幸存者区的大小（字节）
        S1C是第二个幸存者区的大小（字节）
        S0U是第一个幸存者区已使用的大小（字节）
        S1U是第二个幸存者区已使用的大小（字节）
        EC是Eden空间的大小（字节）
        EU是Eden空间已使用的大小（字节）
    老年代相关:
        OC是老年代的大小（字节）
        OU是老年代已使用的大小（字节）
    方法区（元空间）相关:
        MC是方法区的大小（字节）
        MU是方法区已使用的大小（字节）
        CCSC是压缩类空间的大小（字节）
        CCSU是压缩类空间已使用的大小（字节）
    其他:
        YGC是从应用程序启动到采样时young gc次数
        YGCT是从应用程序启动到采样时young gc消耗的时间（秒）
        FGC是从应用程序启动到采样时full gc次数
        FGCT是从应用程序启动到采样时full gc消耗的时间（秒）
        GCT是从应用程序启动到采样时gc的总时间
> 代码测试
```java
/**
 * -Xmx60m -Xms60m -XX:SurvivorRatio=8
 */
public class GCDemo {
    public static void main(String[] args) {
        List<byte[]> list = new ArrayList<>();
        for (int i = 0; i < 1000; i++) {
            byte[] arr = new byte[1024 * 100]; // 100 kb
            list.add(arr);
            try {
                Thread.sleep(120);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```
此时通过jps查看此程序进程id并输出每隔1秒钟输出一次gc信息，一共10次：
```java
nasuf@songtaodeMacBook-Pro jstat -gc 29513 1000 10
 S0C    S1C    S0U    S1U      EC       EU        OC         OU       MC     MU    CCSC   CCSU   YGC     YGCT    FGC    FGCT     GCT
2048.0 2048.0  0.0    0.0   16384.0   6644.4   40960.0      0.0     4480.0 774.3  384.0   75.9       0    0.000   0      0.000    0.000
2048.0 2048.0  0.0    0.0   16384.0   6724.5   40960.0      0.0     4480.0 774.3  384.0   75.9       0    0.000   0      0.000    0.000
2048.0 2048.0  0.0    0.0   16384.0   6804.6   40960.0      0.0     4480.0 774.3  384.0   75.9       0    0.000   0      0.000    0.000
2048.0 2048.0  0.0    0.0   16384.0   7152.3   40960.0      0.0     4480.0 774.3  384.0   75.9       0    0.000   0      0.000    0.000
2048.0 2048.0  0.0    0.0   16384.0   7152.3   40960.0      0.0     4480.0 774.3  384.0   75.9       0    0.000   0      0.000    0.000
2048.0 2048.0  0.0    0.0   16384.0   7152.3   40960.0      0.0     4480.0 774.3  384.0   75.9       0    0.000   0      0.000    0.000
2048.0 2048.0  0.0    0.0   16384.0   7152.3   40960.0      0.0     4480.0 774.3  384.0   75.9       0    0.000   0      0.000    0.000
2048.0 2048.0  0.0    0.0   16384.0   7222.4   40960.0      0.0     4480.0 774.3  384.0   75.9       0    0.000   0      0.000    0.000
2048.0 2048.0  0.0    0.0   16384.0   7302.6   40960.0      0.0     4480.0 774.3  384.0   75.9       0    0.000   0      0.000    0.000
2048.0 2048.0  0.0    0.0   16384.0   7382.7   40960.0      0.0     4480.0 774.3  384.0   75.9       0    0.000   0      0.000    0.000
```
- -gccapacity：显示内容与-gc基本相同，但输出主要关注Java堆各个区域使用到的最大、最小空间
- -gcutil：显示内容与-gc基本相同，但输出主要关注已使用空间占总空间的百分比
```java
nasuf@songtaodeMacBook-Pro jstat -gcutil 29748 2000 30
S0     S1     E      O      M     CCS    YGC     YGCT    FGC    FGCT     GCT
0.00   0.00  89.09   0.00  17.28  19.76      0    0.000     0    0.000    0.000
0.00   0.00  98.86   0.00  17.28  19.76      0    0.000     0    0.000    0.000
0.00  99.23   9.22  31.66  64.97  68.13      1    0.009     0    0.000    0.009
0.00  99.23  18.99  31.66  64.97  68.13      1    0.009     0    0.000    0.009
0.00  99.23  29.37  31.66  64.97  68.13      1    0.009     0    0.000    0.009
0.00  99.23  39.14  31.66  64.97  68.13      1    0.009     0    0.000    0.009
0.00  99.23  48.90  31.66  64.97  68.13      1    0.009     0    0.000    0.009
0.00  99.23  59.28  31.66  64.97  68.13      1    0.009     0    0.000    0.009
0.00  99.23  69.05  31.66  64.97  68.13      1    0.009     0    0.000    0.009
0.00  99.23  81.33  31.66  64.97  68.13      1    0.009     0    0.000    0.009
0.00  99.23  89.26  31.66  64.97  68.13      1    0.009     0    0.000    0.009
0.00  99.23  99.03  31.66  64.97  68.13      1    0.009     0    0.000    0.009
0.00   0.00  11.25  76.09  64.98  68.13      2    0.018     1    0.009    0.027
0.00   0.00  21.63  76.09  64.98  68.13      2    0.018     1    0.009    0.027
0.00   0.00  31.40  76.09  64.98  68.13      2    0.018     1    0.009    0.027
0.00   0.00  41.16  76.09  64.98  68.13      2    0.018     1    0.009    0.027
0.00   0.00  50.93  76.09  64.98  68.13      2    0.018     1    0.009    0.027
0.00   0.00  61.31  76.09  64.98  68.13      2    0.018     1    0.009    0.027
0.00   0.00  71.08  76.09  64.98  68.13      2    0.018     1    0.009    0.027
0.00   0.00  81.45  76.09  64.98  68.13      2    0.018     1    0.009    0.027
0.00   0.00  91.22  76.09  64.98  68.13      2    0.018     1    0.009    0.027
0.00   0.00  43.43  98.93  64.98  68.13      2    0.018     2    0.022    0.040
0.00   0.00  51.98  98.93  64.98  68.13      2    0.018     2    0.022    0.040
0.00   0.00  61.74  98.93  64.98  68.13      2    0.018     2    0.022    0.040
0.00   0.00  71.51  98.93  64.98  68.13      2    0.018     2    0.022    0.040
0.00   0.00  81.89  98.93  64.98  68.13      2    0.018     2    0.022    0.040
0.00   0.00  91.66  98.93  64.98  68.13      2    0.018     2    0.022    0.040
0.00   0.00  99.52  99.90  64.98  68.13      2    0.018     3    0.026    0.044
```

- -gccause：与-gcutil功能一样，但是会额外输出导致最后一次或当前正在发生的GC产生的原因
```java
nasuf@songtaodeMacBook-Pro jstat -gccause 29803 1000 10
S0     S1     E      O      M     CCS    YGC     YGCT    FGC    FGCT     GCT    LGCC                 GCC
0.00  99.23   5.56  31.79  64.97  68.13      1    0.009     0    0.000    0.009 Allocation Failure   No GC
0.00  99.23  10.45  31.79  64.97  68.13      1    0.009     0    0.000    0.009 Allocation Failure   No GC
0.00  99.23  15.94  31.79  64.97  68.13      1    0.009     0    0.000    0.009 Allocation Failure   No GC
0.00  99.23  20.82  31.79  64.97  68.13      1    0.009     0    0.000    0.009 Allocation Failure   No GC
0.00  99.23  25.71  31.79  64.97  68.13      1    0.009     0    0.000    0.009 Allocation Failure   No GC
0.00  99.23  30.59  31.79  64.97  68.13      1    0.009     0    0.000    0.009 Allocation Failure   No GC
0.00  99.23  35.47  31.79  64.97  68.13      1    0.009     0    0.000    0.009 Allocation Failure   No GC
0.00  99.23  40.97  31.79  64.97  68.13      1    0.009     0    0.000    0.009 Allocation Failure   No GC
0.00  99.23  45.85  31.79  64.97  68.13      1    0.009     0    0.000    0.009 Allocation Failure   No GC
0.00  99.23  50.74  31.79  64.97  68.13      1    0.009     0    0.000    0.009 Allocation Failure   No GC
```
- -gcnew：显示新生代GC状况
- -gcnewcapacity：显示内容与-gcnew基本相同，输出主要关注使用到的最大、最小空间
- -gcold：显示老年代GC状况
- -gcoldcapacity：显示内容与-gcold基本相同，输出主要关注使用到的最大、最小空间
- -gcpermcapacity：显示永久代使用到的最大、最小空间
> JIT相关
- -compiler：显示JIT编译器编译过的方法、耗时等信息
```java
nasuf@songtaodeMacBook-Pro jstat -compiler 61335
Compiled Failed Invalid   Time   FailedType FailedMethod
      89      0       0     0.03          0
```
- -printcompilation：输出已经被JIT编译的方法
```java
nasuf@songtaodeMacBook-Pro jstat -printcompilation 61335
Compiled  Size  Type Method
  89     99    1 java/lang/StringBuffer append
```

### 2.2.2 interval参数
用于指定输出统计数据的周期，单位为毫秒。即查询间隔
```java
# 每隔一秒输出一次类装载相关信息
nasuf@songtaodeMacBook-Pro jstat -class 61335 1000
Loaded  Bytes  Unloaded  Bytes     Time
   604  1225.6        0     0.0       0.08
   604  1225.6        0     0.0       0.08
   604  1225.6        0     0.0       0.08
   604  1225.6        0     0.0       0.08
   604  1225.6        0     0.0       0.0
```

### 2.2.3 count参数
用于指定查询的总次数  
```java
# 每隔一秒输出一次类装载相关信息，累计5次终止
nasuf@songtaodeMacBook-Pro jstat -class 61335 1000 5
Loaded  Bytes  Unloaded  Bytes     Time
   604  1225.6        0     0.0       0.08
   604  1225.6        0     0.0       0.08
   604  1225.6        0     0.0       0.08
   604  1225.6        0     0.0       0.08
   604  1225.6        0     0.0       0.08
```

### 2.2.4 -t 参数
可以在输出信息前加一个timestamp列，显示程序运行时间。单位为秒
```java
# 每隔一秒输出一次类装载相关信息，累计5次终止，并额外输出一个timestamp列，表示从程序开始执行到当前为止的总运行时间
nasuf@songtaodeMacBook-Pro jstat -class -t 61335 1000 5
Timestamp       Loaded  Bytes  Unloaded  Bytes     Time
         1038.0    604  1225.6        0     0.0       0.08
         1039.0    604  1225.6        0     0.0       0.08
         1040.0    604  1225.6        0     0.0       0.08
         1041.1    604  1225.6        0     0.0       0.08
         1042.0    604  1225.6        0     0.0       0.08
```
**经验：** 我们可以比较Java进程的启动时间以及总GC时间（GCT）为例，或者两次测量的间隔时间以及总GC时间的增量，来得出GC时间占运行时间的比例。如果该比例超过20%，则说明目前堆的压力比较大；如果该比例超过90%，则说明堆里几乎没有可用空间，随时都可能抛出OOM异常

 ### 2.2.5 -h\<lines> 参数
 可以在周期性数据输出时，输出多少行数据后输出一个表头信息
 ```java
 # 每隔一秒输出一次类装载相关信息，累计10次终止；同时每隔5行输出一次表头信息
nasuf@songtaodeMacBook-Pro jstat -class -t -h5 61335 1000 10
Timestamp       Loaded  Bytes  Unloaded  Bytes     Time
         1221.4    604  1225.6        0     0.0       0.08
         1222.4    604  1225.6        0     0.0       0.08
         1223.4    604  1225.6        0     0.0       0.08
         1224.4    604  1225.6        0     0.0       0.08
         1225.4    604  1225.6        0     0.0       0.08
Timestamp       Loaded  Bytes  Unloaded  Bytes     Time
         1226.4    604  1225.6        0     0.0       0.08
         1227.4    604  1225.6        0     0.0       0.08
         1228.4    604  1225.6        0     0.0       0.08
         1229.4    604  1225.6        0     0.0       0.08
         1230.4    604  1225.6        0     0.0       0.08
 ```
 ### 2.2.6 补充说明
 jstat还可以用来判断是否出现内存泄漏：
 - 第一步：在长时间运行的Java程序中，我们可以运行jstat命令连续获取多行性能数据，并取这几行数据中OU列（即已占用的老年代内存）的最小值
 - 第二步：然后我们每隔一段较长的时间重复一次上述操作，来获得多组OU最小值。如果这些值呈上涨趋势，则说明该Java程序的老年代内存已使用量在不断上涨，这意味着无法回收的对象在不断增加，因此可能存在内存泄漏

# 3. jinfo：实时查看和修改JVM配置参数
**注意：该命令在mac系统下的jdk8版本有bug，故使用jdk9版本进行演示**
## 3.1 基本情况
jinfo（Configuration Info for Java）查看虚拟机配置参数信息，也可以用于调整虚拟机的配置参数

在很多情况下，Java应用程序不会指定所有的Java虚拟机参数。而此时，开发人员可能不知道某一个具体的Java虚拟机参数的默认值。在这种情况下，可能需要通过查找文档获取某个参数的默认值。这个查找过程可能是非常艰难的。但有了jinfo工具，开发人员可以很方便地找到Java虚拟机参数的当前值
## 3.2 基本语法
依然运行下列代码查看jinfo信息：
```java
package com.nasuf.jdk9;

import java.util.Scanner;

public class ScannerTest {

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        String info = scanner.next();
    }
    
}
```
## 3.3 参数查看
> jinfo -syspros PID
> 该命令可以查看由`System.getProperties()`取得的参数  

```java
nasuf@songtaodeMacBook-Pro jinfo -sysprops 1341
Java System Properties:
#Sun Jan 23 20:21:14 CST 2022
java.runtime.name=Java(TM) SE Runtime Environment
sun.boot.library.path=/Library/Java/JavaVirtualMachines/jdk1.8.0_102.jdk/Contents/Home/jre/lib
java.vm.version=25.102-b14
gopherProxySet=false
java.vm.vendor=Oracle Corporation
java.vendor.url=http\://java.oracle.com/
path.separator=\:
java.vm.name=Java HotSpot(TM) 64-Bit Server VM
file.encoding.pkg=sun.io
user.country=CN
sun.java.launcher=SUN_STANDARD
sun.os.patch.level=unknown
java.vm.specification.name=Java Virtual Machine Specification
user.dir=/Users/nasuf/Project/JvmNotesCode
java.runtime.version=1.8.0_102-b14
java.awt.graphicsenv=sun.awt.CGraphicsEnvironment
java.endorsed.dirs=/Library/Java/JavaVirtualMachines/jdk1.8.0_102.jdk/Contents/Home/jre/lib/endorsed
os.arch=x86_64
java.io.tmpdir=/var/folders/n6/mygfwd311mj6lvhjrdbpbct80000gn/T/
line.separator=\n
java.vm.specification.vendor=Oracle Corporation
os.name=Mac OS X
sun.jnu.encoding=UTF-8
java.library.path=/Users/nasuf/Library/Java/Extensions\:/Library/Java/Extensions\:/Network/Library/Java/Extensions\:/System/Library/Java/Extensions\:/usr/lib/java\:.
java.specification.name=Java Platform API Specification
java.class.version=52.0
sun.management.compiler=HotSpot 64-Bit Tiered Compilers
os.version=10.15.7
http.nonProxyHosts=local|*.local|169.254/16|*.169.254/16
user.home=/Users/nasuf
user.timezone=Asia/Shanghai
java.awt.printerjob=sun.lwawt.macosx.CPrinterJob
file.encoding=UTF-8
java.specification.version=1.8
java.class.path=/Library/Java/JavaVirtualMachines/jdk1.8.0_102.jdk/Contents/Home/jre/lib/charsets.jar\:/Library/Java/JavaVirtualMachines/jdk1.8.0_102.jdk/Contents/Home/jre/lib/deploy.jar\:/Library/Java/JavaVirtualMachines/jdk1.8.0_102.jdk/Contents/Home/jre/lib/ext/cldrdata.jar\:/Library/Java/JavaVirtualMachines/jdk1.8.0_102.jdk/Contents/Home/jre/lib/ext/dnsns.jar\:/Library/Java/JavaVirtualMachines/jdk1.8.0_102.jdk/Contents/Home/jre/lib/ext/jaccess.jar\:/Library/Java/JavaVirtualMachines/jdk1.8.0_102.jdk/Contents/Home/jre/lib/ext/jfxrt.jar\:/Library/Java/JavaVirtualMachines/jdk1.8.0_102.jdk/Contents/Home/jre/lib/ext/localedata.jar\:/Library/Java/JavaVirtualMachines/jdk1.8.0_102.jdk/Contents/Home/jre/lib/ext/nashorn.jar\:/Library/Java/JavaVirtualMachines/jdk1.8.0_102.jdk/Contents/Home/jre/lib/ext/sunec.jar\:/Library/Java/JavaVirtualMachines/jdk1.8.0_102.jdk/Contents/Home/jre/lib/ext/sunjce_provider.jar\:/Library/Java/JavaVirtualMachines/jdk1.8.0_102.jdk/Contents/Home/jre/lib/ext/sunpkcs11.jar\:/Library/Java/JavaVirtualMachines/jdk1.8.0_102.jdk/Contents/Home/jre/lib/ext/zipfs.jar\:/Library/Java/JavaVirtualMachines/jdk1.8.0_102.jdk/Contents/Home/jre/lib/javaws.jar\:/Library/Java/JavaVirtualMachines/jdk1.8.0_102.jdk/Contents/Home/jre/lib/jce.jar\:/Library/Java/JavaVirtualMachines/jdk1.8.0_102.jdk/Contents/Home/jre/lib/jfr.jar\:/Library/Java/JavaVirtualMachines/jdk1.8.0_102.jdk/Contents/Home/jre/lib/jfxswt.jar\:/Library/Java/JavaVirtualMachines/jdk1.8.0_102.jdk/Contents/Home/jre/lib/jsse.jar\:/Library/Java/JavaVirtualMachines/jdk1.8.0_102.jdk/Contents/Home/jre/lib/management-agent.jar\:/Library/Java/JavaVirtualMachines/jdk1.8.0_102.jdk/Contents/Home/jre/lib/plugin.jar\:/Library/Java/JavaVirtualMachines/jdk1.8.0_102.jdk/Contents/Home/jre/lib/resources.jar\:/Library/Java/JavaVirtualMachines/jdk1.8.0_102.jdk/Contents/Home/jre/lib/rt.jar\:/Library/Java/JavaVirtualMachines/jdk1.8.0_102.jdk/Contents/Home/lib/ant-javafx.jar\:/Library/Java/JavaVirtualMachines/jdk1.8.0_102.jdk/Contents/Home/lib/dt.jar\:/Library/Java/JavaVirtualMachines/jdk1.8.0_102.jdk/Contents/Home/lib/javafx-mx.jar\:/Library/Java/JavaVirtualMachines/jdk1.8.0_102.jdk/Contents/Home/lib/jconsole.jar\:/Library/Java/JavaVirtualMachines/jdk1.8.0_102.jdk/Contents/Home/lib/packager.jar\:/Library/Java/JavaVirtualMachines/jdk1.8.0_102.jdk/Contents/Home/lib/sa-jdi.jar\:/Library/Java/JavaVirtualMachines/jdk1.8.0_102.jdk/Contents/Home/lib/tools.jar\:/Users/nasuf/Project/JvmNotesCode/out/production/jdk1.8-module
user.name=nasuf
java.vm.specification.version=1.8
sun.java.command=com.nasuf.jdk8.ScannerTest
java.home=/Library/Java/JavaVirtualMachines/jdk1.8.0_102.jdk/Contents/Home/jre
sun.arch.data.model=64
user.language=zh
java.specification.vendor=Oracle Corporation
awt.toolkit=sun.lwawt.macosx.LWCToolkit
java.vm.info=mixed mode
java.version=1.8.0_102
java.ext.dirs=/Users/nasuf/Library/Java/Extensions\:/Library/Java/JavaVirtualMachines/jdk1.8.0_102.jdk/Contents/Home/jre/lib/ext\:/Library/Java/Extensions\:/Network/Library/Java/Extensions\:/System/Library/Java/Extensions\:/usr/lib/java
sun.boot.class.path=/Library/Java/JavaVirtualMachines/jdk1.8.0_102.jdk/Contents/Home/jre/lib/resources.jar\:/Library/Java/JavaVirtualMachines/jdk1.8.0_102.jdk/Contents/Home/jre/lib/rt.jar\:/Library/Java/JavaVirtualMachines/jdk1.8.0_102.jdk/Contents/Home/jre/lib/sunrsasign.jar\:/Library/Java/JavaVirtualMachines/jdk1.8.0_102.jdk/Contents/Home/jre/lib/jsse.jar\:/Library/Java/JavaVirtualMachines/jdk1.8.0_102.jdk/Contents/Home/jre/lib/jce.jar\:/Library/Java/JavaVirtualMachines/jdk1.8.0_102.jdk/Contents/Home/jre/lib/charsets.jar\:/Library/Java/JavaVirtualMachines/jdk1.8.0_102.jdk/Contents/Home/jre/lib/jfr.jar\:/Library/Java/JavaVirtualMachines/jdk1.8.0_102.jdk/Contents/Home/jre/classes
java.vendor=Oracle Corporation
file.separator=/
java.vendor.url.bug=http\://bugreport.sun.com/bugreport/
sun.io.unicode.encoding=UnicodeBig
sun.cpu.endian=little
socksNonProxyHosts=local|*.local|169.254/16|*.169.254/16
ftp.nonProxyHosts=local|*.local|169.254/16|*.169.254/16
sun.cpu.isalist=
```
> jinfo -flags PID
> 该命令可以查看曾经赋过值的一些参数
```java
nasuf@songtaodeMacBook-Pro jinfo -flags 1341
VM Flags:
-XX:CICompilerCount=4 -XX:InitialHeapSize=268435456 -XX:MaxHeapSize=4294967296 -XX:MaxNewSize=1431306240 -XX:MinHeapDeltaBytes=524288 -XX:NewSize=89128960 -XX:OldSize=179306496 -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:+UseFastUnorderedTimeStamps -XX:+UseParallelGC
```
> jinfo -flag 具体参数 PID
> 该命令可以查看某个java进程的具体参数的值
```java
nasuf@songtaodeMacBook-Pro jinfo -flag UseParallelGC 1498
-XX:-UseParallelGC
nasuf@songtaodeMacBook-Pro jinfo -flag UseG1GC 1498
-XX:+UseG1GC
nasuf@songtaodeMacBook-Pro jinfo -flag MaxHeapSize 1498
-XX:MaxHeapSize=4294967296
```
## 3.4 参数修改
jinfo不仅可以查看运行时某一个Java虚拟机参数的实际取值，甚至可以在运行时修改部分参数，并使之立即生效。但是，并非所有的参数都支持动态修改。参数只有被标记为`manageable`的flag可以被实时修改。其实，这个修改能力是极其有限的。使用以下命令查看被标记为`manageable`的参数：
```java
# jdk8环境下：
nasuf@songtaodeMacBook-Pro  ~  java -XX:+PrintFlagsFinal -version | grep manageable
     intx CMSAbortablePrecleanWaitMillis            = 100                                 {manageable}
     intx CMSTriggerInterval                        = -1                                  {manageable}
     intx CMSWaitDuration                           = 2000                                {manageable}
     bool HeapDumpAfterFullGC                       = false                               {manageable}
     bool HeapDumpBeforeFullGC                      = false                               {manageable}
     bool HeapDumpOnOutOfMemoryError                = false                               {manageable}
    ccstr HeapDumpPath                              =                                     {manageable}
    uintx MaxHeapFreeRatio                          = 100                                 {manageable}
    uintx MinHeapFreeRatio                          = 0                                   {manageable}
     bool PrintClassHistogram                       = false                               {manageable}
     bool PrintClassHistogramAfterFullGC            = false                               {manageable}
     bool PrintClassHistogramBeforeFullGC           = false                               {manageable}
     bool PrintConcurrentLocks                      = false                               {manageable}
     bool PrintGC                                   = false                               {manageable}
     bool PrintGCDateStamps                         = false                               {manageable}
     bool PrintGCDetails                            = false                               {manageable}
     bool PrintGCID                                 = false                               {manageable}
     bool PrintGCTimeStamps                         = false                               {manageable}
java version "1.8.0_202"
Java(TM) SE Runtime Environment (build 1.8.0_202-b08)
Java HotSpot(TM) 64-Bit Server VM (build 25.202-b08, mixed mode)
```
```java
# jdk9环境下
nasuf@songtaodeMacBook-Pro java -XX:+PrintFlagsFinal -version | grep manageable
     intx CMSAbortablePrecleanWaitMillis           = 100                                   {manageable} {default}
     intx CMSTriggerInterval                       = -1                                    {manageable} {default}
     intx CMSWaitDuration                          = 2000                                  {manageable} {default}
     bool HeapDumpAfterFullGC                      = false                                 {manageable} {default}
     bool HeapDumpBeforeFullGC                     = false                                 {manageable} {default}
     bool HeapDumpOnOutOfMemoryError               = false                                 {manageable} {default}
    ccstr HeapDumpPath                             =                                       {manageable} {default}
    uintx MaxHeapFreeRatio                         = 70                                    {manageable} {default}
    uintx MinHeapFreeRatio                         = 40                                    {manageable} {default}
     bool PrintClassHistogram                      = false                                 {manageable} {default}
     bool PrintConcurrentLocks                     = false                                 {manageable} {default}
java version "9.0.1"
Java(TM) SE Runtime Environment (build 9.0.1+11)
Java HotSpot(TM) 64-Bit Server VM (build 9.0.1+11, mixed mode)
```
修改方式：  
针对boolean类型：`jinfo -flag [+|-] 具体参数 PID`  
针对非boolean类型：`jinfo -flag 具体参数=具体参数值 PID`
> 扩展
- java -XX:+PrintFlagsInitial 查看所有JVM参数启动的初始值
- java -XX:+PrintFlagsFinal 查看所有JVM参数的最终值
- java -XX:+PrintCommangLineFlags 查看已经被用户或者JVM设置过的详细的XX参数的名称和值

# 4. jmap：导出内存映像文件 & 内存使用情况
## 4.1 基本情况
jmap（JVM Memory Map）：作用一方面是获取dump文件（堆转储快照文件，二进制文件），它还可以获取目标Java进程的内存相关信息，包括Java堆各区域的使用情况、堆中对象的统计信息、类加载信息等。开发人员可以在控制台中输入命令`jmap -help`查询jmap工具的具体使用方式和一些标准选项配置  
官方帮助文档：https://docs.oracle.com/en/java/javase/11/tools/jmap.html
## 4.2 基本语法
它的基本使用语法为：
- jmap [option] \<pid>
- jmap [option] \<executable \<core>
- jmap [option] \<server_id@]\<remote server IP or hostname>
其中option选项包括：

| 选项           | 作用                                                         |
| -------------- | ------------------------------------------------------------ |
| -dump          | 生成Java堆转储快照dump文件 <br> 特别的：-dump:live只保存堆中的存活对象 |
| -finalizerinfo | 显示在F-Queue中等待Finalizer线程执行finalize方法的对象<br>仅linux/solaris平台有效 |
| -heap          | 输出整个堆空间的详细信息，包括GC的使用、堆配置信息，以及内存的使用信息等 |
| -histo         | 输出对空间中对象的统计信息，包括类、实例数量和合计容量<br>特别的：-histo:live只统计堆中的存活对象 |
| -permstat      | 以ClassLoader为统计口径输出永久代的内存状态信息<br>仅linux/solaris平台有效 |
| -F             | 当虚拟机进程对-dump选项没有任何响应时，强制执行生成dump文件<br>仅linux/solaris平台有效 |
| -h (-help)     | jmap工具使用的帮助命令                                       |
| -J \<flag>     | 传递参数给jmap启动的jvm                                      |

说明：这些参数和linux下输入显示的命令多少会有些不同，包括也受jdk版本的影响
## 4.3 使用方式
### 4.3.1 导出内存映像文件
一般来说，使用jmap指令生成dump文件的操作算得上是最常用的jmap指令之一，将堆中所有存活对象导出至一个文件之中

Heap Dump又叫做堆存储文件，指一个Java进程在某个时间点的内存快照。Heap Dump在出发内存快照的时候会保存此刻如下的信息：
- All Objects: Class, fields, primitive values, references
- All Classes: ClassLoader, name, super class, static fields
- Garbage Collection Roots: Objects defined to be reachable by the JVM
- Thread Stacks and Local Variables: The call-stacks of threads at the moment of the snapshot, and per-frame information about local objects  
说明：
- 通常在写Heap Dump文件前会触发一次Full GC，所以Heap Dump文件里保存的都是Full GC后留下的对象信息
- 由于生成dump文件比较耗时，因此大家需要耐心等待，尤其是大内存镜像生成dump文件则需要耗费更长的时间来完成

> 手动方式

- jmap -dump:format=b,file=<filename_hprof> <pid>
- jmap -dump:live,format=b,file=<filename_hprof> <pid>
```java
/**
 * -Xmx60m -Xms60m -XX:SurvivorRatio=8
 */
public class GCDemo {
    public static void main(String[] args) {
        List<byte[]> list = new ArrayList<>();
        for (int i = 0; i < 1000; i++) {
            byte[] arr = new byte[1024 * 100]; // 100 kb
            list.add(arr);
            try {
                Thread.sleep(120);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```
运行以上代码。在代码执行过程中生成hprof文件如下：
```java
 nasuf@nasufdeMacBook-Pro  ~/Project/JvmNotesCode   master  jmap -dump:format=b,file=1.hprof 36946
Dumping heap to /Users/nasuf/Project/JvmNotesCode/1.hprof ...
Heap dump file created
 nasuf@nasufdeMacBook-Pro  ~/Project/JvmNotesCode   master  jmap -dump:format=b,file=2.hprof 36946
Dumping heap to /Users/nasuf/Project/JvmNotesCode/2.hprof ...
Heap dump file created
 nasuf@nasufdeMacBook-Pro  ~/Project/JvmNotesCode   master  jmap -dump:format=b,file=3.hprof 36946
Dumping heap to /Users/nasuf/Project/JvmNotesCode/3.hprof ...
Heap dump file created
 nasuf@nasufdeMacBook-Pro  ~/Project/JvmNotesCode   master  jmap -dump:live,format=b,file=4.hprof 36946
Dumping heap to /Users/nasuf/Project/JvmNotesCode/4.hprof ...
Heap dump file created
```
可以看到随着程序的执行，生成的hprof文件大小在增加：

![image.png](/images/jvm/20/1.png)
而4.hprof文件实际在生产环境中会变得更小，因为live参数表明只保存堆中得存活对象  

值得注意的一点是，由于jmap将访问堆中的所有对象，为了保证在此过程中不被应用县城干扰，jmap需要借助安全点机制，让所有线程停留在不改变堆中数据的状态。也就是说，由jmap导出的堆快照必定是安全点位置的。这可能导致基于该堆快照的分析结果存在偏差。  

举个例子，假设在编译生成的机器码中，某些对象的生命周期在两个安全点之间，那么`:live`选项将无法探知到这些对象。  

另外，如果某个线程长时间无法跑到安全点，jmap将一直等待下去。与前面讲的jstat则不同，垃圾回收器会主动将jstat所需要的摘要数据保存至固定位置中，而jstat只需直接读取即可

> 自动方式

当程序发生OOM退出系统时，一些瞬时信息都随着程序的终止而消失，而重现OOM问题往往比较困难或者耗时。此时若能在OOM时，自动导出dump文件就显得非常迫切  
这里介绍一种比较常用的取得堆快照文件的方法，即使用`-XX:+HeapDumpOnOutOfMemoryError`参数，在程序发生OOM时，导出应用程序的当前堆快照，`-XX:HeapDumpPath`可以指定堆快照的保存位置  
依然执行上述代码，修改运行参数为：`-Xmx100m -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/Users/nasuf/Project/JvmNotesCode/auto_dump.hprof`，程序运行后输出如下：
```java
/Library/Java/JavaVirtualMachines/jdk1.8.0_311.jdk/Contents/Home/bin/java -Xmx100m -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/Users/nasuf/Project/JvmNotesCode/auto_dump.hprof -javaagent:/Applications/IntelliJ IDEA CE.app/Contents/lib/idea_rt.jar=49275:/Applications/IntelliJ IDEA CE.app/Contents/bin -Dfile.encoding=UTF-8 -classpath /Library/Java/JavaVirtualMachines/jdk1.8.0_311.jdk/Contents/Home/lib/ant-javafx.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_311.jdk/Contents/Home/lib/dt.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_311.jdk/Contents/Home/lib/javafx-mx.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_311.jdk/Contents/Home/lib/jconsole.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_311.jdk/Contents/Home/lib/packager.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_311.jdk/Contents/Home/lib/sa-jdi.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_311.jdk/Contents/Home/lib/tools.jar:/Users/nasuf/Project/JvmNotesCode/out/production/jdk1.8-module com.nasuf.jdk8.GCDemo
java.lang.OutOfMemoryError: Java heap space
Dumping heap to /Users/nasuf/Project/JvmNotesCode/auto_dump.hprof ...
Heap dump file created [97156453 bytes in 0.368 secs]
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
	at com.nasuf.jdk8.GCDemo.main(GCDemo.java:13)

Process finished with exit code 1
```

### 4.3.2 显示堆内存相关信息
- jmap -heap pid 显示当前时间节点的堆内存情况 
- jmap -histo pid 显示当前时间节点的堆内存情况，包括每个类包括的对象

### 4.3.3 其他使用方式
- jmap -permstat pid 查看系统的ClassLoader信息
- jmap -finalizerinfo 查看堆积在finalizer队列中的对象

# 5. jhat：JDK自带堆分析工具
## 5.1 基本情况
**jhat（JVM Heap Analysis Tool）**：SUN JDK提供的jhat命令与jmap命令搭配使用，用于分析jmap生成的heap dump文件（堆转储快照）。jhat内置了一个微型的HTTP/HTML服务器，生成dump文件的分析结果后，用户可以在浏览器中查看分析结果（分析虚拟机转储快照信息）  
使用了jhat命令，就启动了一个http服务，端口是7000，即 http://localhost:7000/， 就可以在浏览器里分析  
**说明：jhat命令在jdk9、jdk10中已被移除，官方建议用VisualVM代替**

## 5.2 基本用法
```java
 nasuf@nasufdeMacBook-Pro  ~/Project/JvmNotesCode   master ±  jhat 1.hprof
Reading from 1.hprof...
Dump file created Mon Mar 28 19:58:02 CST 2022
Snapshot read, resolving...
Resolving 20043 objects...
Chasing references, expect 4 dots....
Eliminating duplicate references....
Snapshot resolved.
Started HTTP server on port 7000
Server is ready.
```

![image.png](/images/jvm/20/2.png)
- option参数：-stack false|true 关闭|打开对象分配调用栈跟踪
- option参数：-refs false|true 关闭|打开对象引用跟踪
- option参数：-port port-number 设置jhap HTTP Server的端口号，默认为70000
- option参数：-exclude exclude-file 执行对象查询时需要排除的数据成员
- option参数：-baseline exclude-file 制定一个基准堆转储
- option参数：-debug int 设置debug级别
- option参数：-version 启动后显示版本信息就退出
- option参数：-J\<flag> 传入启动参数，比如-J -Xmx512m

# 6. jstak：打印JVM中线程快照
jstak（JVM Stack Trace）：用于生成虚拟机指定进程当前时刻的线程快照（虚拟机堆栈跟踪）。线程快照就是当前虚拟机内指定进程的每一条线程正在执行的方法堆栈的集合  
生成线程快照的作用：可用于定位线程出现长时间停顿的原因，如果线程间死锁、死循环、请求外部资源导致的长时间等待等问题。这些都是导致线程长时间停顿的常见原因。当线程出现停顿时，就可以用jstak显示各个线程调用的堆栈情况  
在thread dump中，要留意下面几种状态：
- 死锁，Deadlock
- 等待资源，Waiting on condition
- 等待获取监视器，Waiting on monitor entry
- 阻塞，Blocked
- 执行中，Runnable
- 暂停，Suspended

## 6.1 基本使用

### 6.1.1 测试线程死锁状态
> 测试代码
```java
package com.nasuf.jdk8;

public class ThreadDeadLock {
    public static void main(String[] args) {
        StringBuilder s1 = new StringBuilder();
        StringBuilder s2 = new StringBuilder();

        new Thread(() -> {
            synchronized (s1) {
                s1.append("a");
                s2.append("1");

                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }

                synchronized (s2) {
                    s1.append("b");
                    s2.append("2");
                    System.out.println(s1);
                    System.out.println(s2);
                }
            }
        }).start();

        new Thread(() -> {
            synchronized (s2) {
                s1.append("c");
                s2.append("3");

                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }

                synchronized (s1) {
                    s1.append("d");
                    s2.append("4");
                    System.out.println(s1);
                    System.out.println(s2);
                }
            }
        }).start();
    }
}
```
使用jstack命令查看线程信息如下：
```java
nasuf@nasufdeMacBook-Pro  ~/Project/JvmNotesCode   master ±  jps
41632 ThreadDeadLock
41749 Jps
83802
41631 Launcher
 nasuf@nasufdeMacBook-Pro  ~/Project/JvmNotesCode   master ±  jstack 41632
2022-03-31 20:23:14
Full thread dump Java HotSpot(TM) 64-Bit Server VM (25.311-b11 mixed mode):

"Attach Listener" #14 daemon prio=9 os_prio=31 tid=0x00007fd9a100a800 nid=0x5703 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"DestroyJavaVM" #13 prio=5 os_prio=31 tid=0x00007fd9a2859800 nid=0x1003 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Thread-1" #12 prio=5 os_prio=31 tid=0x00007fd9a1058800 nid=0x5603 waiting for monitor entry [0x0000700004a93000]
   java.lang.Thread.State: BLOCKED (on object monitor)
	at com.nasuf.jdk8.ThreadDeadLock.lambda$main$1(ThreadDeadLock.java:40)
	- waiting to lock <0x000000076ac0ffb0> (a java.lang.StringBuilder)
	- locked <0x000000076ac0fff8> (a java.lang.StringBuilder)
	at com.nasuf.jdk8.ThreadDeadLock$$Lambda$2/1747585824.run(Unknown Source)
	at java.lang.Thread.run(Thread.java:748)

"Thread-0" #11 prio=5 os_prio=31 tid=0x00007fd9a1058000 nid=0x5503 waiting for monitor entry [0x0000700004990000]
   java.lang.Thread.State: BLOCKED (on object monitor)
	at com.nasuf.jdk8.ThreadDeadLock.lambda$main$0(ThreadDeadLock.java:20)
	- waiting to lock <0x000000076ac0fff8> (a java.lang.StringBuilder)
	- locked <0x000000076ac0ffb0> (a java.lang.StringBuilder)
	at com.nasuf.jdk8.ThreadDeadLock$$Lambda$1/1078694789.run(Unknown Source)
	at java.lang.Thread.run(Thread.java:748)

"Service Thread" #10 daemon prio=9 os_prio=31 tid=0x00007fd9a0835000 nid=0x3903 runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"C1 CompilerThread3" #9 daemon prio=9 os_prio=31 tid=0x00007fd9a3828000 nid=0x3d03 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"C2 CompilerThread2" #8 daemon prio=9 os_prio=31 tid=0x00007fd9a3827000 nid=0x3603 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"C2 CompilerThread1" #7 daemon prio=9 os_prio=31 tid=0x00007fd9a200c000 nid=0x3503 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"C2 CompilerThread0" #6 daemon prio=9 os_prio=31 tid=0x00007fd9a103f000 nid=0x3f03 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Monitor Ctrl-Break" #5 daemon prio=5 os_prio=31 tid=0x00007fd9a102f000 nid=0x4003 runnable [0x000070000427b000]
   java.lang.Thread.State: RUNNABLE
	at java.net.SocketInputStream.socketRead0(Native Method)
	at java.net.SocketInputStream.socketRead(SocketInputStream.java:116)
	at java.net.SocketInputStream.read(SocketInputStream.java:171)
	at java.net.SocketInputStream.read(SocketInputStream.java:141)
	at sun.nio.cs.StreamDecoder.readBytes(StreamDecoder.java:284)
	at sun.nio.cs.StreamDecoder.implRead(StreamDecoder.java:326)
	at sun.nio.cs.StreamDecoder.read(StreamDecoder.java:178)
	- locked <0x000000076ac8e6d8> (a java.io.InputStreamReader)
	at java.io.InputStreamReader.read(InputStreamReader.java:184)
	at java.io.BufferedReader.fill(BufferedReader.java:161)
	at java.io.BufferedReader.readLine(BufferedReader.java:324)
	- locked <0x000000076ac8e6d8> (a java.io.InputStreamReader)
	at java.io.BufferedReader.readLine(BufferedReader.java:389)
	at com.intellij.rt.execution.application.AppMainV2$1.run(AppMainV2.java:49)

"Signal Dispatcher" #4 daemon prio=9 os_prio=31 tid=0x00007fd9a381c800 nid=0x4203 runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Finalizer" #3 daemon prio=8 os_prio=31 tid=0x00007fd9a2019800 nid=0x4a03 in Object.wait() [0x0000700003f6f000]
   java.lang.Thread.State: WAITING (on object monitor)
	at java.lang.Object.wait(Native Method)
	- waiting on <0x000000076ab08ee0> (a java.lang.ref.ReferenceQueue$Lock)
	at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:144)
	- locked <0x000000076ab08ee0> (a java.lang.ref.ReferenceQueue$Lock)
	at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:165)
	at java.lang.ref.Finalizer$FinalizerThread.run(Finalizer.java:216)

"Reference Handler" #2 daemon prio=10 os_prio=31 tid=0x00007fd9a1815800 nid=0x4c03 in Object.wait() [0x0000700003e6c000]
   java.lang.Thread.State: WAITING (on object monitor)
	at java.lang.Object.wait(Native Method)
	- waiting on <0x000000076ab06c00> (a java.lang.ref.Reference$Lock)
	at java.lang.Object.wait(Object.java:502)
	at java.lang.ref.Reference.tryHandlePending(Reference.java:191)
	- locked <0x000000076ab06c00> (a java.lang.ref.Reference$Lock)
	at java.lang.ref.Reference$ReferenceHandler.run(Reference.java:153)

"VM Thread" os_prio=31 tid=0x00007fd9a1024000 nid=0x4e03 runnable

"GC task thread#0 (ParallelGC)" os_prio=31 tid=0x00007fd99f00f800 nid=0x1e07 runnable

"GC task thread#1 (ParallelGC)" os_prio=31 tid=0x00007fd9a0008800 nid=0x2303 runnable

"GC task thread#2 (ParallelGC)" os_prio=31 tid=0x00007fd99f010000 nid=0x2103 runnable

"GC task thread#3 (ParallelGC)" os_prio=31 tid=0x00007fd9a1809800 nid=0x2a03 runnable

"GC task thread#4 (ParallelGC)" os_prio=31 tid=0x00007fd9a180a000 nid=0x2c03 runnable

"GC task thread#5 (ParallelGC)" os_prio=31 tid=0x00007fd9a100d000 nid=0x5303 runnable

"GC task thread#6 (ParallelGC)" os_prio=31 tid=0x00007fd9a100d800 nid=0x5103 runnable

"GC task thread#7 (ParallelGC)" os_prio=31 tid=0x00007fd9a2809000 nid=0x5003 runnable

"VM Periodic Task Thread" os_prio=31 tid=0x00007fd9a3829000 nid=0x3b03 waiting on condition

JNI global references: 319


Found one Java-level deadlock:
=============================
"Thread-1":
  waiting to lock monitor 0x00007fd9a102e158 (object 0x000000076ac0ffb0, a java.lang.StringBuilder),
  which is held by "Thread-0"
"Thread-0":
  waiting to lock monitor 0x00007fd9a102ccb8 (object 0x000000076ac0fff8, a java.lang.StringBuilder),
  which is held by "Thread-1"

Java stack information for the threads listed above:
===================================================
"Thread-1":
	at com.nasuf.jdk8.ThreadDeadLock.lambda$main$1(ThreadDeadLock.java:40)
	- waiting to lock <0x000000076ac0ffb0> (a java.lang.StringBuilder)
	- locked <0x000000076ac0fff8> (a java.lang.StringBuilder)
	at com.nasuf.jdk8.ThreadDeadLock$$Lambda$2/1747585824.run(Unknown Source)
	at java.lang.Thread.run(Thread.java:748)
"Thread-0":
	at com.nasuf.jdk8.ThreadDeadLock.lambda$main$0(ThreadDeadLock.java:20)
	- waiting to lock <0x000000076ac0fff8> (a java.lang.StringBuilder)
	- locked <0x000000076ac0ffb0> (a java.lang.StringBuilder)
	at com.nasuf.jdk8.ThreadDeadLock$$Lambda$1/1078694789.run(Unknown Source)
	at java.lang.Thread.run(Thread.java:748)

Found 1 deadlock.
```

### 6.1.2 测试线程等待状态
> 测试代码
```java
package com.nasuf.jdk8;

public class ThreadSleepTest {

    public static void main(String[] args) {
        System.out.println("hello - 1");
        try {
            Thread.sleep(1000 * 60 * 10);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("hello - 2");
    }

}
```
使用jstack查看线程信息如下：
```java
nasuf@nasufdeMacBook-Pro  ~/Project/JvmNotesCode   master ±  jps
46184 Jps
83802
46107 Launcher
46108 ThreadSleepTest
 nasuf@nasufdeMacBook-Pro  ~/Project/JvmNotesCode   master ±  jstack 46108
2022-03-31 20:33:58
Full thread dump Java HotSpot(TM) 64-Bit Server VM (25.311-b11 mixed mode):

"Attach Listener" #11 daemon prio=9 os_prio=31 tid=0x00007f9db0008800 nid=0x5503 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Service Thread" #10 daemon prio=9 os_prio=31 tid=0x00007f9db103d800 nid=0x4103 runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"C1 CompilerThread3" #9 daemon prio=9 os_prio=31 tid=0x00007f9db1809000 nid=0x3d03 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"C2 CompilerThread2" #8 daemon prio=9 os_prio=31 tid=0x00007f9db304b000 nid=0x4403 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"C2 CompilerThread1" #7 daemon prio=9 os_prio=31 tid=0x00007f9db0019000 nid=0x4603 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"C2 CompilerThread0" #6 daemon prio=9 os_prio=31 tid=0x00007f9daf81b000 nid=0x3c03 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Monitor Ctrl-Break" #5 daemon prio=5 os_prio=31 tid=0x00007f9db304a800 nid=0x4903 runnable [0x0000700004391000]
   java.lang.Thread.State: RUNNABLE
	at java.net.SocketInputStream.socketRead0(Native Method)
	at java.net.SocketInputStream.socketRead(SocketInputStream.java:116)
	at java.net.SocketInputStream.read(SocketInputStream.java:171)
	at java.net.SocketInputStream.read(SocketInputStream.java:141)
	at sun.nio.cs.StreamDecoder.readBytes(StreamDecoder.java:284)
	at sun.nio.cs.StreamDecoder.implRead(StreamDecoder.java:326)
	at sun.nio.cs.StreamDecoder.read(StreamDecoder.java:178)
	- locked <0x000000076ac8e708> (a java.io.InputStreamReader)
	at java.io.InputStreamReader.read(InputStreamReader.java:184)
	at java.io.BufferedReader.fill(BufferedReader.java:161)
	at java.io.BufferedReader.readLine(BufferedReader.java:324)
	- locked <0x000000076ac8e708> (a java.io.InputStreamReader)
	at java.io.BufferedReader.readLine(BufferedReader.java:389)
	at com.intellij.rt.execution.application.AppMainV2$1.run(AppMainV2.java:49)

"Signal Dispatcher" #4 daemon prio=9 os_prio=31 tid=0x00007f9db0818000 nid=0x4a03 runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Finalizer" #3 daemon prio=8 os_prio=31 tid=0x00007f9db1011000 nid=0x3403 in Object.wait() [0x0000700004085000]
   java.lang.Thread.State: WAITING (on object monitor)
	at java.lang.Object.wait(Native Method)
	- waiting on <0x000000076ab08ee0> (a java.lang.ref.ReferenceQueue$Lock)
	at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:144)
	- locked <0x000000076ab08ee0> (a java.lang.ref.ReferenceQueue$Lock)
	at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:165)
	at java.lang.ref.Finalizer$FinalizerThread.run(Finalizer.java:216)

"Reference Handler" #2 daemon prio=10 os_prio=31 tid=0x00007f9db300e800 nid=0x3203 in Object.wait() [0x0000700003f82000]
   java.lang.Thread.State: WAITING (on object monitor)
	at java.lang.Object.wait(Native Method)
	- waiting on <0x000000076ab06c00> (a java.lang.ref.Reference$Lock)
	at java.lang.Object.wait(Object.java:502)
	at java.lang.ref.Reference.tryHandlePending(Reference.java:191)
	- locked <0x000000076ab06c00> (a java.lang.ref.Reference$Lock)
	at java.lang.ref.Reference$ReferenceHandler.run(Reference.java:153)

"main" #1 prio=5 os_prio=31 tid=0x00007f9db200b800 nid=0xe03 waiting on condition [0x0000700003564000]
   java.lang.Thread.State: TIMED_WAITING (sleeping)
	at java.lang.Thread.sleep(Native Method)
	at com.nasuf.jdk8.ThreadSleepTest.main(ThreadSleepTest.java:8)

"VM Thread" os_prio=31 tid=0x00007f9db100e800 nid=0x5203 runnable

"GC task thread#0 (ParallelGC)" os_prio=31 tid=0x00007f9daf808800 nid=0x2007 runnable

"GC task thread#1 (ParallelGC)" os_prio=31 tid=0x00007f9db2014000 nid=0x1f03 runnable

"GC task thread#2 (ParallelGC)" os_prio=31 tid=0x00007f9db180a000 nid=0x1d03 runnable

"GC task thread#3 (ParallelGC)" os_prio=31 tid=0x00007f9daf80e000 nid=0x2a03 runnable

"GC task thread#4 (ParallelGC)" os_prio=31 tid=0x00007f9db2015000 nid=0x2c03 runnable

"GC task thread#5 (ParallelGC)" os_prio=31 tid=0x00007f9db2015800 nid=0x2d03 runnable

"GC task thread#6 (ParallelGC)" os_prio=31 tid=0x00007f9db180b000 nid=0x2f03 runnable

"GC task thread#7 (ParallelGC)" os_prio=31 tid=0x00007f9daf80f000 nid=0x5303 runnable

"VM Periodic Task Thread" os_prio=31 tid=0x00007f9daf81c000 nid=0x4003 waiting on condition

JNI global references: 15
```

### 6.1.3 测试线程同步问题
> 测试代码
```java
package com.nasuf.jdk8;

public class ThreadSyncTest {

    public static void main(String[] args) {
        Number number = new Number();
        Thread t1 = new Thread(number);
        Thread t2 = new Thread(number);

        t1.setName("Thread 1");
        t2.setName("Thread 2");

        t1.start();
        t2.start();
    }

}

class Number implements Runnable {

    private int number = 1;

    @Override
    public void run() {
        while (true) {
            synchronized (this) {
                if (number <= 100) {
                    try {
                        Thread.sleep(500);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }

                    System.out.println(Thread.currentThread().getName() + ":" + number);
                    number++;
                } else {
                    break;
                }
            }
        }
    }
}
```
此时使用jstack查看线程状态如下。由于每个线程睡眠500ms，所以某个线程状态是TIMED_WAITING，另一个就是BLOCKED
```java
nasuf@nasufdeMacBook-Pro  ~/Project/GoDemos   master  jps
64841 Jps
83802
64828 Launcher
64829 ThreadSyncTest
 nasuf@nasufdeMacBook-Pro  ~/Project/GoDemos   master  jstack 64829
2022-04-02 09:58:31
Full thread dump Java HotSpot(TM) 64-Bit Server VM (25.311-b11 mixed mode):

"Attach Listener" #14 daemon prio=9 os_prio=31 tid=0x00007f8783039800 nid=0x5903 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"DestroyJavaVM" #13 prio=5 os_prio=31 tid=0x00007f8786042000 nid=0xc03 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Thread 2" #12 prio=5 os_prio=31 tid=0x00007f878204d000 nid=0x5703 waiting for monitor entry [0x0000700006cb9000]
   java.lang.Thread.State: BLOCKED (on object monitor)
	at com.nasuf.jdk8.Number.run(ThreadSyncTest.java:27)
	- waiting to lock <0x000000076ac12360> (a com.nasuf.jdk8.Number)
	at java.lang.Thread.run(Thread.java:748)

"Thread 1" #11 prio=5 os_prio=31 tid=0x00007f8780827800 nid=0x5503 waiting on condition [0x0000700006bb6000]
   java.lang.Thread.State: TIMED_WAITING (sleeping)
	at java.lang.Thread.sleep(Native Method)
	at com.nasuf.jdk8.Number.run(ThreadSyncTest.java:29)
	- locked <0x000000076ac12360> (a com.nasuf.jdk8.Number)
	at java.lang.Thread.run(Thread.java:748)

"Service Thread" #10 daemon prio=9 os_prio=31 tid=0x00007f8786816800 nid=0x3e03 runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"C1 CompilerThread3" #9 daemon prio=9 os_prio=31 tid=0x00007f878382e000 nid=0x4003 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"C2 CompilerThread2" #8 daemon prio=9 os_prio=31 tid=0x00007f8783016800 nid=0x4203 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"C2 CompilerThread1" #7 daemon prio=9 os_prio=31 tid=0x00007f878680d800 nid=0x4403 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"C2 CompilerThread0" #6 daemon prio=9 os_prio=31 tid=0x00007f878104b000 nid=0x3a03 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Monitor Ctrl-Break" #5 daemon prio=5 os_prio=31 tid=0x00007f878283f000 nid=0x3903 runnable [0x00007000064a1000]
   java.lang.Thread.State: RUNNABLE
	at java.net.SocketInputStream.socketRead0(Native Method)
	at java.net.SocketInputStream.socketRead(SocketInputStream.java:116)
	at java.net.SocketInputStream.read(SocketInputStream.java:171)
	at java.net.SocketInputStream.read(SocketInputStream.java:141)
	at sun.nio.cs.StreamDecoder.readBytes(StreamDecoder.java:284)
	at sun.nio.cs.StreamDecoder.implRead(StreamDecoder.java:326)
	at sun.nio.cs.StreamDecoder.read(StreamDecoder.java:178)
	- locked <0x000000076ac8def0> (a java.io.InputStreamReader)
	at java.io.InputStreamReader.read(InputStreamReader.java:184)
	at java.io.BufferedReader.fill(BufferedReader.java:161)
	at java.io.BufferedReader.readLine(BufferedReader.java:324)
	- locked <0x000000076ac8def0> (a java.io.InputStreamReader)
	at java.io.BufferedReader.readLine(BufferedReader.java:389)
	at com.intellij.rt.execution.application.AppMainV2$1.run(AppMainV2.java:49)

"Signal Dispatcher" #4 daemon prio=9 os_prio=31 tid=0x00007f8783816800 nid=0x4703 runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Finalizer" #3 daemon prio=8 os_prio=31 tid=0x00007f8782023800 nid=0x4f03 in Object.wait() [0x0000700006195000]
   java.lang.Thread.State: WAITING (on object monitor)
	at java.lang.Object.wait(Native Method)
	- waiting on <0x000000076ab08ee0> (a java.lang.ref.ReferenceQueue$Lock)
	at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:144)
	- locked <0x000000076ab08ee0> (a java.lang.ref.ReferenceQueue$Lock)
	at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:165)
	at java.lang.ref.Finalizer$FinalizerThread.run(Finalizer.java:216)

"Reference Handler" #2 daemon prio=10 os_prio=31 tid=0x00007f8781048000 nid=0x3203 in Object.wait() [0x0000700006092000]
   java.lang.Thread.State: WAITING (on object monitor)
	at java.lang.Object.wait(Native Method)
	- waiting on <0x000000076ab06c00> (a java.lang.ref.Reference$Lock)
	at java.lang.Object.wait(Object.java:502)
	at java.lang.ref.Reference.tryHandlePending(Reference.java:191)
	- locked <0x000000076ab06c00> (a java.lang.ref.Reference$Lock)
	at java.lang.ref.Reference$ReferenceHandler.run(Reference.java:153)

"VM Thread" os_prio=31 tid=0x00007f8781045000 nid=0x3103 runnable

"GC task thread#0 (ParallelGC)" os_prio=31 tid=0x00007f8782015800 nid=0x1a07 runnable

"GC task thread#1 (ParallelGC)" os_prio=31 tid=0x00007f8782016000 nid=0x1e03 runnable

"GC task thread#2 (ParallelGC)" os_prio=31 tid=0x00007f8781008800 nid=0x1c03 runnable

"GC task thread#3 (ParallelGC)" os_prio=31 tid=0x00007f8782016800 nid=0x2a03 runnable

"GC task thread#4 (ParallelGC)" os_prio=31 tid=0x00007f8782017000 nid=0x2c03 runnable

"GC task thread#5 (ParallelGC)" os_prio=31 tid=0x00007f8782018000 nid=0x2d03 runnable

"GC task thread#6 (ParallelGC)" os_prio=31 tid=0x00007f8782018800 nid=0x2f03 runnable

"GC task thread#7 (ParallelGC)" os_prio=31 tid=0x00007f8782019000 nid=0x5303 runnable

"VM Periodic Task Thread" os_prio=31 tid=0x00007f878080a000 nid=0x3c03 waiting on condition

JNI global references: 15
```

## 6.2 基本语法
- option参数：-F 当正常输出的请求不被响应时，强制输出线程堆栈
- option参数：-l 除堆栈外，显示关于锁的附加信息  
依然使用上述程序测试此参数：
```java
nasuf@nasufdeMacBook-Pro  ~/Project/GoDemos   master  jps
66688 ThreadSyncTest
66725 Jps
83802
66687 Launcher
 nasuf@nasufdeMacBook-Pro  ~/Project/GoDemos   master  jstack -l 66688
2022-04-02 10:07:03
Full thread dump Java HotSpot(TM) 64-Bit Server VM (25.311-b11 mixed mode):

"Attach Listener" #14 daemon prio=9 os_prio=31 tid=0x00007fcb40087000 nid=0x5703 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
	- None

"DestroyJavaVM" #13 prio=5 os_prio=31 tid=0x00007fcb40083800 nid=0xf03 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
	- None

"Thread 2" #12 prio=5 os_prio=31 tid=0x00007fcb40843800 nid=0xa803 waiting for monitor entry [0x000070000bd9b000]
   java.lang.Thread.State: BLOCKED (on object monitor)
	at com.nasuf.jdk8.Number.run(ThreadSyncTest.java:27)
	- waiting to lock <0x000000076ac12360> (a com.nasuf.jdk8.Number)
	at java.lang.Thread.run(Thread.java:748)

   Locked ownable synchronizers:
	- None

"Thread 1" #11 prio=5 os_prio=31 tid=0x00007fcb40843000 nid=0x5503 waiting on condition [0x000070000bc98000]
   java.lang.Thread.State: TIMED_WAITING (sleeping)
	at java.lang.Thread.sleep(Native Method)
	at com.nasuf.jdk8.Number.run(ThreadSyncTest.java:29)
	- locked <0x000000076ac12360> (a com.nasuf.jdk8.Number)
	at java.lang.Thread.run(Thread.java:748)

   Locked ownable synchronizers:
	- None

"Service Thread" #10 daemon prio=9 os_prio=31 tid=0x00007fcb41066000 nid=0x4303 runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
	- None

"C1 CompilerThread3" #9 daemon prio=9 os_prio=31 tid=0x00007fcb4005b000 nid=0x4103 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
	- None

"C2 CompilerThread2" #8 daemon prio=9 os_prio=31 tid=0x00007fcb4005a000 nid=0x3f03 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
	- None

"C2 CompilerThread1" #7 daemon prio=9 os_prio=31 tid=0x00007fcb4101d000 nid=0x4703 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
	- None

"C2 CompilerThread0" #6 daemon prio=9 os_prio=31 tid=0x00007fcb4101c000 nid=0x3c03 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
	- None

"Monitor Ctrl-Break" #5 daemon prio=5 os_prio=31 tid=0x00007fcb40059000 nid=0x3b03 runnable [0x000070000b583000]
   java.lang.Thread.State: RUNNABLE
	at java.net.SocketInputStream.socketRead0(Native Method)
	at java.net.SocketInputStream.socketRead(SocketInputStream.java:116)
	at java.net.SocketInputStream.read(SocketInputStream.java:171)
	at java.net.SocketInputStream.read(SocketInputStream.java:141)
	at sun.nio.cs.StreamDecoder.readBytes(StreamDecoder.java:284)
	at sun.nio.cs.StreamDecoder.implRead(StreamDecoder.java:326)
	at sun.nio.cs.StreamDecoder.read(StreamDecoder.java:178)
	- locked <0x000000076ac8def0> (a java.io.InputStreamReader)
	at java.io.InputStreamReader.read(InputStreamReader.java:184)
	at java.io.BufferedReader.fill(BufferedReader.java:161)
	at java.io.BufferedReader.readLine(BufferedReader.java:324)
	- locked <0x000000076ac8def0> (a java.io.InputStreamReader)
	at java.io.BufferedReader.readLine(BufferedReader.java:389)
	at com.intellij.rt.execution.application.AppMainV2$1.run(AppMainV2.java:49)

   Locked ownable synchronizers:
	- None

"Signal Dispatcher" #4 daemon prio=9 os_prio=31 tid=0x00007fcb43009000 nid=0x4903 runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
	- None

"Finalizer" #3 daemon prio=8 os_prio=31 tid=0x00007fcb4001c000 nid=0x4f03 in Object.wait() [0x000070000b277000]
   java.lang.Thread.State: WAITING (on object monitor)
	at java.lang.Object.wait(Native Method)
	- waiting on <0x000000076ab08ee0> (a java.lang.ref.ReferenceQueue$Lock)
	at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:144)
	- locked <0x000000076ab08ee0> (a java.lang.ref.ReferenceQueue$Lock)
	at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:165)
	at java.lang.ref.Finalizer$FinalizerThread.run(Finalizer.java:216)

   Locked ownable synchronizers:
	- None

"Reference Handler" #2 daemon prio=10 os_prio=31 tid=0x00007fcb4100e000 nid=0x3103 in Object.wait() [0x000070000b174000]
   java.lang.Thread.State: WAITING (on object monitor)
	at java.lang.Object.wait(Native Method)
	- waiting on <0x000000076ab06c00> (a java.lang.ref.Reference$Lock)
	at java.lang.Object.wait(Object.java:502)
	at java.lang.ref.Reference.tryHandlePending(Reference.java:191)
	- locked <0x000000076ab06c00> (a java.lang.ref.Reference$Lock)
	at java.lang.ref.Reference$ReferenceHandler.run(Reference.java:153)

   Locked ownable synchronizers:
	- None

"VM Thread" os_prio=31 tid=0x00007fcb4081b800 nid=0x3003 runnable

"GC task thread#0 (ParallelGC)" os_prio=31 tid=0x00007fcb40809800 nid=0x1d07 runnable

"GC task thread#1 (ParallelGC)" os_prio=31 tid=0x00007fcb4080f800 nid=0x2203 runnable

"GC task thread#2 (ParallelGC)" os_prio=31 tid=0x00007fcb3f009000 nid=0x2003 runnable

"GC task thread#3 (ParallelGC)" os_prio=31 tid=0x00007fcb42009000 nid=0x2a03 runnable

"GC task thread#4 (ParallelGC)" os_prio=31 tid=0x00007fcb3f00a000 nid=0x5403 runnable

"GC task thread#5 (ParallelGC)" os_prio=31 tid=0x00007fcb40810800 nid=0x2d03 runnable

"GC task thread#6 (ParallelGC)" os_prio=31 tid=0x00007fcb42808800 nid=0x5203 runnable

"GC task thread#7 (ParallelGC)" os_prio=31 tid=0x00007fcb43008800 nid=0x2e03 runnable

"VM Periodic Task Thread" os_prio=31 tid=0x00007fcb3f016800 nid=0x4503 waiting on condition

JNI global references: 15
```
- option参数：-m 如果调用到本地方法的话，可以显示C/C++的堆栈
- option参数：-h 帮助操作

# 7. jcmd：多功能命令行
在JDK1.7以后，新增了一个命令行工具jcmd，它是一个多功能工具，可以用来实现前面除了jstat之外所有命令的功能。比如，用它来导出堆、内存使用、查看Java进程、导出线程信息、执行GC、JVM运行时间等  

官方帮助文档：https://docs.oracle.com/en/java/javase/11/tools/jcmd.html  

jcmd拥有jmap的大部分功能，并且在Oracle的官方网站上也推荐使用jcmd命令代替jmap命令

## 7.1 基本语法
- jcmd -l 列出所有的jvm进程
    ```java
    asuf@nasufdeMacBook-Pro  ~/Project/GoDemos   master  jcmd -l
    83802
    70733 org.jetbrains.jps.cmdline.Launcher /Applications/IntelliJ IDEA CE.app/Contents/plugins/java/lib/jps-builders.jar:/Applications/IntelliJ IDEA CE.app/Contents/plugins/java/lib/jps-builders-6.jar:/Applications/IntelliJ IDEA CE.app/Contents/plugins/java/lib/jps-javac-extension-1.jar:/Applications/IntelliJ IDEA CE.app/Contents/lib/util.jar:/Applications/IntelliJ IDEA CE.app/Contents/lib/annotations.jar:/Applications/IntelliJ IDEA CE.app/Contents/lib/3rd-party-rt.jar:/Applications/IntelliJ IDEA CE.app/Contents/lib/jna.jar:/Applications/IntelliJ IDEA CE.app/Contents/lib/kotlin-stdlib-jdk8.jar:/Applications/IntelliJ IDEA CE.app/Contents/lib/protobuf.jar:/Applications/IntelliJ IDEA CE.app/Contents/lib/platform-api.jar:/Applications/IntelliJ IDEA CE.app/Contents/lib/jps-model.jar:/Applications/IntelliJ IDEA CE.app/Contents/plugins/java/lib/javac2.jar:/Applications/IntelliJ IDEA CE.app/Contents/lib/forms_rt.jar:/Applications/IntelliJ IDEA CE.app/Contents/plugins/java/lib/qdox.jar:/Applications/IntelliJ IDEA CE.app/Conte
    70734 com.nasuf.jdk8.ThreadDeadLock
    70766 sun.tools.jcmd.JCmd -l
    ```
    对比jps命令输出如下
    ```java
    nasuf@nasufdeMacBook-Pro  ~/Project/GoDemos   master  jps -m
    71044 Jps -m
    83802
    70733 Launcher /Applications/IntelliJ IDEA CE.app/Contents/plugins/java/lib/jps-builders.jar:/Applications/IntelliJ IDEA CE.app/Contents/plugins/java/lib/jps-builders-6.jar:/Applications/IntelliJ IDEA CE.app/Contents/plugins/java/lib/jps-javac-extension-1.jar:/Applications/IntelliJ IDEA CE.app/Contents/lib/util.jar:/Applications/IntelliJ IDEA CE.app/Contents/lib/annotations.jar:/Applications/IntelliJ IDEA CE.app/Contents/lib/3rd-party-rt.jar:/Applications/IntelliJ IDEA CE.app/Contents/lib/jna.jar:/Applications/IntelliJ IDEA CE.app/Contents/lib/kotlin-stdlib-jdk8.jar:/Applications/IntelliJ IDEA CE.app/Contents/lib/protobuf.jar:/Applications/IntelliJ IDEA CE.app/Contents/lib/platform-api.jar:/Applications/IntelliJ IDEA CE.app/Contents/lib/jps-model.jar:/Applications/IntelliJ IDEA CE.app/Contents/plugins/java/lib/javac2.jar:/Applications/IntelliJ IDEA CE.app/Contents/lib/forms_rt.jar:/Applications/IntelliJ IDEA CE.app/Contents/plugins/java/lib/qdox.jar:/Applications/IntelliJ IDEA CE.app/Conte
    70734 ThreadDeadLock
    ```
- jcmd pid help 针对指定的进程，列出支持的所有命令
    ```java
    nasuf@nasufdeMacBook-Pro  ~/Project/GoDemos   master  jcmd 70734 help
    70734:
    The following commands are available:
    JFR.stop
    JFR.start
    JFR.dump
    JFR.check
    VM.native_memory
    VM.check_commercial_features
    VM.unlock_commercial_features
    ManagementAgent.stop
    ManagementAgent.start_local
    ManagementAgent.start
    VM.classloader_stats
    GC.rotate_log
    Thread.print
    GC.class_stats
    GC.class_histogram
    GC.heap_dump
    GC.finalizer_info
    GC.heap_info
    GC.run_finalization
    GC.run
    VM.uptime
    VM.dynlibs
    VM.flags
    VM.system_properties
    VM.command_line
    VM.version
    help
    
    For more information about a specific command use 'help <command>'.
    ```
- jcmd pid <具体命令> 显示指定进程的指令命令的数据
    ```java
    nasuf@nasufdeMacBook-Pro  ~/Project/GoDemos   master  jcmd 70734 Thread.print
    70734:
    2022-04-02 10:27:37
    Full thread dump Java HotSpot(TM) 64-Bit Server VM (25.311-b11 mixed mode):
    
    "Attach Listener" #14 daemon prio=9 os_prio=31 tid=0x00007fed1c03a800 nid=0x5703 waiting on condition [0x0000000000000000]
       java.lang.Thread.State: RUNNABLE
    
    "DestroyJavaVM" #13 prio=5 os_prio=31 tid=0x00007fed16816000 nid=0x1003 waiting on condition [0x0000000000000000]
       java.lang.Thread.State: RUNNABLE
    
    "Thread-1" #12 prio=5 os_prio=31 tid=0x00007fed16815000 nid=0x5603 waiting for monitor entry [0x0000700006515000]
       java.lang.Thread.State: BLOCKED (on object monitor)
            at com.nasuf.jdk8.ThreadDeadLock.lambda$main$1(ThreadDeadLock.java:40)
            - waiting to lock <0x000000076ac0ffb0> (a java.lang.StringBuilder)
            - locked <0x000000076ac0fff8> (a java.lang.StringBuilder)
            at com.nasuf.jdk8.ThreadDeadLock$$Lambda$2/1747585824.run(Unknown Source)
            at java.lang.Thread.run(Thread.java:748)
    
    "Thread-0" #11 prio=5 os_prio=31 tid=0x00007fed188ce800 nid=0x5503 waiting for monitor entry [0x0000700006412000]
       java.lang.Thread.State: BLOCKED (on object monitor)
            at com.nasuf.jdk8.ThreadDeadLock.lambda$main$0(ThreadDeadLock.java:20)
            - waiting to lock <0x000000076ac0fff8> (a java.lang.StringBuilder)
            - locked <0x000000076ac0ffb0> (a java.lang.StringBuilder)
            at com.nasuf.jdk8.ThreadDeadLock$$Lambda$1/1078694789.run(Unknown Source)
            at java.lang.Thread.run(Thread.java:748)
    
    "Service Thread" #10 daemon prio=9 os_prio=31 tid=0x00007fed19041000 nid=0x4103 runnable [0x0000000000000000]
       java.lang.Thread.State: RUNNABLE
    
    "C1 CompilerThread3" #9 daemon prio=9 os_prio=31 tid=0x00007fed18877800 nid=0x4003 waiting on condition [0x0000000000000000]
       java.lang.Thread.State: RUNNABLE
    
    "C2 CompilerThread2" #8 daemon prio=9 os_prio=31 tid=0x00007fed18876800 nid=0x4603 waiting on condition [0x0000000000000000]
       java.lang.Thread.State: RUNNABLE
    
    "C2 CompilerThread1" #7 daemon prio=9 os_prio=31 tid=0x00007fed18875800 nid=0x4703 waiting on condition [0x0000000000000000]
       java.lang.Thread.State: RUNNABLE
    
    "C2 CompilerThread0" #6 daemon prio=9 os_prio=31 tid=0x00007fed18875000 nid=0x3c03 waiting on condition [0x0000000000000000]
       java.lang.Thread.State: RUNNABLE
    
    "Monitor Ctrl-Break" #5 daemon prio=5 os_prio=31 tid=0x00007fed1980a000 nid=0x3a03 runnable [0x0000700005cfd000]
       java.lang.Thread.State: RUNNABLE
            at java.net.SocketInputStream.socketRead0(Native Method)
            at java.net.SocketInputStream.socketRead(SocketInputStream.java:116)
            at java.net.SocketInputStream.read(SocketInputStream.java:171)
            at java.net.SocketInputStream.read(SocketInputStream.java:141)
            at sun.nio.cs.StreamDecoder.readBytes(StreamDecoder.java:284)
            at sun.nio.cs.StreamDecoder.implRead(StreamDecoder.java:326)
            at sun.nio.cs.StreamDecoder.read(StreamDecoder.java:178)
            - locked <0x000000076ac8def0> (a java.io.InputStreamReader)
            at java.io.InputStreamReader.read(InputStreamReader.java:184)
            at java.io.BufferedReader.fill(BufferedReader.java:161)
            at java.io.BufferedReader.readLine(BufferedReader.java:324)
            - locked <0x000000076ac8def0> (a java.io.InputStreamReader)
            at java.io.BufferedReader.readLine(BufferedReader.java:389)
            at com.intellij.rt.execution.application.AppMainV2$1.run(AppMainV2.java:49)
    
    "Signal Dispatcher" #4 daemon prio=9 os_prio=31 tid=0x00007fed17037000 nid=0x3903 runnable [0x0000000000000000]
       java.lang.Thread.State: RUNNABLE
    
    "Finalizer" #3 daemon prio=8 os_prio=31 tid=0x00007fed16013800 nid=0x3203 in Object.wait() [0x00007000059f1000]
       java.lang.Thread.State: WAITING (on object monitor)
            at java.lang.Object.wait(Native Method)
            - waiting on <0x000000076ab08ee0> (a java.lang.ref.ReferenceQueue$Lock)
            at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:144)
            - locked <0x000000076ab08ee0> (a java.lang.ref.ReferenceQueue$Lock)
            at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:165)
            at java.lang.ref.Finalizer$FinalizerThread.run(Finalizer.java:216)
    
    "Reference Handler" #2 daemon prio=10 os_prio=31 tid=0x00007fed17023000 nid=0x4f03 in Object.wait() [0x00007000058ee000]
       java.lang.Thread.State: WAITING (on object monitor)
            at java.lang.Object.wait(Native Method)
            - waiting on <0x000000076ab06c00> (a java.lang.ref.Reference$Lock)
            at java.lang.Object.wait(Object.java:502)
            at java.lang.ref.Reference.tryHandlePending(Reference.java:191)
            - locked <0x000000076ab06c00> (a java.lang.ref.Reference$Lock)
            at java.lang.ref.Reference$ReferenceHandler.run(Reference.java:153)
    
    "VM Thread" os_prio=31 tid=0x00007fed1881b000 nid=0x3003 runnable
    
    "GC task thread#0 (ParallelGC)" os_prio=31 tid=0x00007fed16010000 nid=0x2307 runnable
    
    "GC task thread#1 (ParallelGC)" os_prio=31 tid=0x00007fed16010800 nid=0x2103 runnable
    
    "GC task thread#2 (ParallelGC)" os_prio=31 tid=0x00007fed16809000 nid=0x2003 runnable
    
    "GC task thread#3 (ParallelGC)" os_prio=31 tid=0x00007fed16809800 nid=0x2a03 runnable
    
    "GC task thread#4 (ParallelGC)" os_prio=31 tid=0x00007fed1800e800 nid=0x2c03 runnable
    
    "GC task thread#5 (ParallelGC)" os_prio=31 tid=0x00007fed19808800 nid=0x2d03 runnable
    
    "GC task thread#6 (ParallelGC)" os_prio=31 tid=0x00007fed19010800 nid=0x2f03 runnable
    
    "GC task thread#7 (ParallelGC)" os_prio=31 tid=0x00007fed1880a800 nid=0x5203 runnable
    
    "VM Periodic Task Thread" os_prio=31 tid=0x00007fed16016800 nid=0x4303 waiting on condition
    
    JNI global references: 319


    Found one Java-level deadlock:
    =============================
    "Thread-1":
      waiting to lock monitor 0x00007fed16813758 (object 0x000000076ac0ffb0, a java.lang.StringBuilder),
      which is held by "Thread-0"
    "Thread-0":
      waiting to lock monitor 0x00007fed168122b8 (object 0x000000076ac0fff8, a java.lang.StringBuilder),
      which is held by "Thread-1"
    
    Java stack information for the threads listed above:
    ===================================================
    "Thread-1":
            at com.nasuf.jdk8.ThreadDeadLock.lambda$main$1(ThreadDeadLock.java:40)
            - waiting to lock <0x000000076ac0ffb0> (a java.lang.StringBuilder)
            - locked <0x000000076ac0fff8> (a java.lang.StringBuilder)
            at com.nasuf.jdk8.ThreadDeadLock$$Lambda$2/1747585824.run(Unknown Source)
            at java.lang.Thread.run(Thread.java:748)
    "Thread-0":
            at com.nasuf.jdk8.ThreadDeadLock.lambda$main$0(ThreadDeadLock.java:20)
            - waiting to lock <0x000000076ac0fff8> (a java.lang.StringBuilder)
            - locked <0x000000076ac0ffb0> (a java.lang.StringBuilder)
            at com.nasuf.jdk8.ThreadDeadLock$$Lambda$1/1078694789.run(Unknown Source)
            at java.lang.Thread.run(Thread.java:748)
    
    Found 1 deadlock
    ```
# 8. jstatd：远程主机信息收集
之前的命令只涉及到监控本机的Java应用程序，而在这些工具中，一些监控工具也支持对远程计算机的监控（如jps、jstat）。为了启用远程监控，则需要配合使用jstatd工具  
命令jstatd是一个RMI服务端程序，它的作用相当于代理服务器，建立本地计算机与远程监控工具的通信。jstatd服务器将本机的Java应用程序信息传递到远程计算机

![image.png](/images/jvm/20/3.png)