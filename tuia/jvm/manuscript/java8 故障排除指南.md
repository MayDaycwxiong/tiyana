https://docs.oracle.com/javase/8/docs/technotes/guides/troubleshoot/

# **Java 平台，标准版故障排除指南**

## 第一部分 一般 Java 故障排除

### 1准备 Java 以进行故障排除

#### 1.1设置 Java 进行故障排除

#### 1.2为 JVM 故障排除启用选项/标志

2 **添加-XX：+ HeapDumpOnOutOfMemoryError到JVM标志：**该`-XX:+HeapDumpOnOutOfMemoryError`标志节省了Java堆转储到磁盘，如果应用程序运行到`OutOfMemoryError`。使用[jhat Utility](https://docs.oracle.com/javase/8/docs/technotes/guides/troubleshoot/tooldescr012.html#BABJGJBI)工具检查 Java 堆并找出占用最多空间的对象，然后检查不再使用但保持活动状态的对象。

4 将 -verbosegc 添加到 JVM 命令行：**该标志`-verbosegc`记录有关 Java 垃圾收集器的基本信息。此日志可帮助您找到以下内容：

- 垃圾回收运行时间长吗？
- 空闲内存会随着时间的推移而减少吗？

垃圾收集器日志有助于在应用程序抛出`OutOFMemoryError`或应用程序遇到性能问题时诊断问题；因此，`-verbosegc`默认情况下打开该标志有助于解决问题。

*注意：*使用日志轮换，以便应用程序重新启动不会删除以前的日志。从 JDK7 开始，标志`UseGClogFileRotation`和`NumberOfGCLogFiles`可用于设置日志轮换。有关这些标志的说明，请参阅[Java HotSpot VM 的调试选项](http://www.oracle.com/technetwork/java/javase/tech/vmoptions-jsp-140102.html)。

6 **设置 JMC JMX 以进行远程监控：** JMX 可用于使用 Mission Control 或 Visual VM 等工具远程连接到 Java 应用程序。除非您可以在运行您的应用程序的同一台机器上运行这些工具，否则设置它可能有助于稍后监控应用程序、发送诊断命令、管理飞行记录等。启用 JMX 没有性能开销。

在 Java 应用程序启动后启用 JMX 的另一种替代方法是使用诊断命令`ManagementAgent.start`。运行`jcmd <pid> help ManagementAgent.start`可与命令一起发送的标志列表。

#### 1.3收集相关数据

如果您的应用程序遇到问题并且您想进一步调试问题，请确保在重新启动系统之前收集所有相关数据，特别是如果重新启动将删除以前的文件。

以下是需要收集的重要文件：

崩溃问题的核心文件。`hs_err` Java 崩溃的打印文本文件。

日志文件：Java 和应用程序日志。

Java 堆转储`-XX:+HeapDumpOnOutOfMemoryError`。

Java 飞行记录（如果启用）- 如果问题没有终止应用程序，则转储连续记录。

如果应用程序停止响应，则收集以下文件。

堆栈跟踪：`jcmd <pid> Thread.print`在重新启动系统之前使用多个堆栈跟踪。

转储飞行记录（如果启用）。

强制核心文件：如果应用程序无法正常关闭，则停止应用程序并`kill -6 <pid>`在 Linux 或 Solaris 系统上强制使用核心文件。

1.3.1使 Java 应用程序更易于调试

使用日志框架是启用未来调试的好方法。如果您在特定模块中遇到问题，您应该能够在该模块中启用日志记录。指定不同级别的日志记录也很好，例如信息、调试和跟踪。`java.util.logging`框架的使用方法请参见[Java日志技术](https://docs.oracle.com/javase/8/docs/technotes/guides/logging/index.html)。

#### 2.6 jcmd 实用程序

##### 2.6.1 jcmd 实用程序的有用命令

`jcmd <process id/main class> help`是查看所有可用选项的最佳方式。您始终可以使用`jcmd <process id/main class> help <command>`来获取这些命令的任何其他选项。

**打印完整的 HotSpot 和 JDK 版本 ID**

```
jcmd <process id/main class> VM.version
```

**打印为** VM **设置的所有系统属性**

```
jcmd <process id/main class> VM.system_properties
```

**打印用于** VM **的所有标志**

```
jcmd <process id/main class> VM.flags
```

**以秒为单位打印正常运行时间**

```
jcmd <process id/main class> VM.uptime
```

**创建类直方图**

结果可能相当冗长，因此您可以将输出重定向到文件。内部和特定于应用程序的类都包含在列表中。占用内存最多的类列在顶部，类按降序排列。

```
jcmd <process id/main class> GC.class_histogram
```

sh-4.2# jcmd 1 GC.class_histogram | head -30

**创建堆转储**（**hprof** **dump**）****

`jcmd GC.heap_dump filename=Myheapdump`

这与 using 相同`jmap -dump:file=<file> <pid>`，但`jcmd`它是推荐使用的工具。

**创建堆直方图**

`jcmd <process id/main class> GC.class_histogram filename=Myheaphistogram`

这与 using 相同`jmap -histo <pid>`，但`jcmd`它是推荐使用的工具。

**使用堆栈跟踪打印所有线程**

```
jcmd <process id/main class> Thread.print
```

#### 2.6.2使用 jcmd 实用程序进行故障排除

### 2.7原生内存跟踪

2.7.1使用 NMT 检测内存泄漏

按照以下步骤检测内存泄漏。

使用命令行选项启动带有摘要或详细信息跟踪的 JVM ：`-XX:NativeMemoryTracking=summary`或`-XX:NativeMemoryTracking=detail`。建立早期基线 - 使用 NMT 基线功能通过运行以下命令来获取基线以在开发和维护期间进行比较：`jcmd <pid> VM.native_memory baseline`.

监控内存使用的变化：`jcmd <pid> VM.native_memory detail.diff`。

如果应用程序泄漏少量内存，则需要一段时间才能显示出来。

### 2.8 HPROF

### 2.9控制台

### 2.10 Java VisualVM

### 2.11 jdb 工具

### 2.12 jhat 实用程序

### 2.13 jinfo 实用程序

该`jinfo`命令行实用程序从正在运行的Java进程或崩溃转储获取配置信息，并打印系统属性或用来启动JVM的命令行标志。

JDK 8 的发布引入了 Java Mission Control、Java Flight Recorder 和`jcmd`用于诊断 JVM 和 Java 应用程序问题的实用程序。建议使用最新的实用程序，`jcmd`而不是以前的`jinfo`实用程序，以增强诊断并减少性能开销。

### 2.14 jmap 实用程序

此外，JDK 7 版本引入了`-dump:format=b,file=`filename选项，这会导致`jmap`将二进制 HPROF 格式的 Java 堆转储到指定文件。然后可以使用该`jhat`工具分析该文件。

如果`jmap` pid命令由于进程挂起而没有响应，则`-F`可以使用该选项（仅在 Oracle Solaris 和 Linux 操作系统上）来强制使用 Serviceability Agent。

#### 2.14.1**堆配置和使用**

该`-heap`选项用于获取以下 Java 堆信息：

特定于垃圾回收 (GC) 算法的信息，包括 GC 算法的名称（例如，并行 GC）和特定于算法的详细信息（例如并行 GC 的线程数）。

堆配置可能已指定为命令行选项或由 VM 根据机器配置选择。

堆使用摘要：对于每一代（堆的区域），该工具打印总堆容量、使用中的内存和可用的空闲内存。如果一代被组织为空间的集合（例如，新一代），则包含空间特定的内存大小摘要。

```
jmap -heap 29620
```

#### 2.14.2堆直方图

```
jmap -histo 29620
```

`jmap`带有`-histo`选项的命令可用于获取堆的类特定直方图。根据指定的参数，该`jmap -histo`命令可以打印出正在运行的进程或核心文件的堆直方图。

在正在运行的进程上执行该命令时，该工具会打印每个类的对象数量、内存大小（以字节为单位）和完全限定的类名。Java HotSpot VM 中的内部类用尖括号括起来。直方图有助于理解如何使用堆。要获得对象的大小，您必须将总大小除以该对象类型的计数。

在`jmap -histo`核心文件上执行该命令时，该工具会打印每个类的大小、计数和类名。Java HotSpot VM 中的内部类以星号 (*) 为前缀。

#### 2.14.3类加载器统计

```
jmap -clstats 16624
```

使用`jmap`带有`-clstats`选项的命令打印 Java 堆的类加载器统计信息。该`jmap`命令使用进程 ID 连接到正在运行的进程，并打印有关元空间中加载的类的详细信息：

**class_loader**：运行实用程序时快照处的类加载器对象的地址

**classes** : 加载的类数

**bytes**：这个类加载器加载的所有类的元数据消耗的大约字节数

**parent_loader**：父类加载器的地址（如果有）

**活？**: 加载器对象将来是否会被垃圾收集的存活或死亡指示

**type** : 这个类加载器的类名

#### 2.15 jps 实用程序

#### 2.16 jstack 工具

JDK 8 的发布引入了 Java Mission Control、Java Flight Recorder 和`jcmd`用于诊断 JVM 和 Java 应用程序问题的实用程序。建议使用最新的实用程序，`jcmd`而不是以前的`jstack`实用程序，以增强诊断并减少性能开销。

##### 2.16.1使用 jstack 实用程序进行故障排除

```
jstack -l 8321
```

该`jstack`命令行实用程序附加到指定的进程或核心文件，并打印被附加到虚拟机的所有线程，包括Java线程和VM内螺纹，和任选的本机堆栈帧的堆栈跟踪。该实用程序还执行死锁检测。

该`-l`选项指示实用程序在堆中查找可拥有的同步器并打印有关`java.util.concurrent.locks`. 如果没有此选项，线程转储将仅包含有关监视器的信息。

##### 2.16.2强制堆栈转储

```
jstack -F 8321
```

如果`jstack` pid命令由于进程挂起而没有响应，则`-F`可以使用该选项（仅在 Oracle Solaris 和 Linux 操作系统上）强制进行堆栈转储，如[示例 2-29](https://docs.oracle.com/javase/8/docs/technotes/guides/troubleshoot/tooldescr016.html#BABBBDIG)所示。

##### 2.16.3来自核心转储的堆栈跟踪

##### 2.16.4混合堆栈

该`jstack`实用程序还可用于打印混合堆栈；也就是说，除了 Java 堆栈之外，它还可以打印本机堆栈帧。本机框架是与 VM 代码和 JNI/本机代码相关联的 C/C++ 框架。

```
$ jstack -m 21177 
```

#### 2.17 jstat 实用程序

该`jstat`实用程序使用 Java HotSpot VM 中的内置检测来提供有关正在运行的应用程序的性能和资源消耗的信息。该工具可用于诊断性能问题，尤其是与堆大小和垃圾收集相关的问题。该`jstat`实用程序不需要使用任何特殊选项启动 VM。默认情况下启用 Java HotSpot VM 中的内置检测。此实用程序包含在 Oracle 支持的所有操作系统平台的 JDK 下载中。

`-gcutil`选项的使用，其中`jstat`实用程序附加到 LVMID 编号 2834，以 250 毫秒的间隔采集七个样本。

```
$ jstat -gcutil 2834 250 7 
```

此示例的输出显示在第三个和第四个样本之间发生了年轻代收集。收集耗时0.017秒，将对象从伊甸空间（E）提升到旧空间（O），使得旧空间利用率从46.56%提升至54.60%。

该`-gcnew`选项的使用，其中`jstat`实用程序附加到 LVMID 编号 2834，以 250 毫秒的间隔进行采样，并显示输出。此外，它使用`-h3`选项在每三行数据后显示列标题。

```
$ jstat -gcnew -h3 2834 250 
```

除了显示重复的标头字符串外，这个例子还显示了在第四个和第五个样本之间，发生了年轻代收集，其持续时间为 0.02 秒。收集发现足够的实时数据，幸存者空间 1 利用率 (S1U) 将超过所需的幸存者大小 (DSS)。结果，对象被提升到老年代（在这个输出中不可见），并且任期阈值（TT）从 15 降低到 1。

该`-gcoldcapacity`选项的使用，其中`jstat`实用程序附加到 LVMID 编号 21891 并以 250 毫秒的间隔采集三个样本。该`-t`选项用于为第一列中的每个样本生成时间戳。

```
$ jstat -gcoldcapacity -t 21891 250 3 
```

Timestamp 列报告自目标 JVM 启动以来经过的时间（以秒为单位）。此外，`-gcoldcapacity`输出显示老年代容量（OGC）和老空间容量（OC）随着堆的扩展而增加以满足分配或提升需求。该OGC已经从11696 KB 81之后生长至13820 KB ST充分发电容量（FGC）。该代（和空间）的最大容量为 60544 KB (OGCMX)，因此仍有扩展空间。

#### 2.18 visualgc 工具

该`visualgc`工具相关的`jstat`工具，见[jstat工具](https://docs.oracle.com/javase/8/docs/technotes/guides/troubleshoot/tooldescr017.html#BABCDBEA)。该`visualgc`工具提供了垃圾收集 (GC) 系统的图形视图。与 一样`jstat`，它使用 Java HotSpot VM 的内置检测。

该`visualgc`工具未包含在 JDK 版本中，但可从[`jvmstat`技术](http://www.oracle.com/technetwork/java/jvmstat-142257.html)页面单独下载。

#### 2.21自定义诊断工具

## 3 内存泄漏故障排除

#### 3.1使用 Java Flight Recorder 调试内存泄漏

##### 3.1.1检测内存泄漏

##### 3.1.2查找泄漏类

尤其要注意不属于标准库的类。例如，您经常会看到`Char`数组是最大的种植者之一。这是由于许多`Strings`被分配了；因此，请注意保持这些`Strings`活动的对象。如果你有一个有 10 个`Strings`成员的类，那么对象本身不会使用太多的堆。堆将由 使用`Strings`，它主要包含指向`Char`数组的指针。因此，最好根据实例的数量而不是对象的大小进行排序。如果您的应用程序类之一有许多实例，那么它很可能是那些使其他对象保持活动状态的对象。

##### 3.1.3查找漏洞

查找实际泄漏可能很困难 当您走到这[一步](https://docs.oracle.com/javase/8/docs/technotes/guides/troubleshoot/tooldescr012.html#BABJGJBI)时，要做的一件事是转储[HPROF](https://docs.oracle.com/javase/8/docs/technotes/guides/troubleshoot/tooldescr008.html#BABGIIGB)并使用[jhat 实用程序](https://docs.oracle.com/javase/8/docs/technotes/guides/troubleshoot/tooldescr012.html#BABJGJBI)检查 Java 堆[，](https://docs.oracle.com/javase/8/docs/technotes/guides/troubleshoot/tooldescr012.html#BABJGJBI)以找出可疑对象是如何保持活动状态的。您还可以使用 JMC 插件 JOverflow 来查看 HPROF 转储中的参考链。

#### 3.2理解 OutOfMemoryError 异常

内存泄漏的一个常见迹象是`java.lang.OutOfMemoryError`异常。通常，当 Java 堆中没有足够的空间分配对象时会抛出此错误。在这种情况下，垃圾收集器无法腾出空间来容纳新对象，堆也无法进一步扩展。此外，当本机内存不足以支持 Java 类的加载时，可能会抛出此错误。在极少数情况下，`java.lang.OutOfMemoryError`当花费过多时间进行垃圾收集并且释放的内存很少时，可能会抛出a 。

当`java.lang.OutOfMemoryError`抛出异常时，还会打印堆栈跟踪。

该`java.lang.OutOfMemoryError`异常也可以通过本机库的代码时，抛出本地分配不能满足（例如，如果交换空间不足）。

诊断`OutOfMemoryError`异常的早期步骤是确定异常的原因。是因为Java堆满了还是原生堆满了才抛出？为了帮助您找到原因，异常的文本在末尾包含一条详细消息，如以下异常所示。

https://docs.oracle.com/javase/8/docs/technotes/guides/troubleshoot/memleaks002.html

#### 3.3解决崩溃而不是 OutOfMemoryError

例如，如果没有可用内存，`malloc`系统调用将返回`null`。如果`malloc`未检查return from ，则应用程序在尝试访问无效内存位置时可能会崩溃。根据具体情况，此类问题可能难以定位。

#### 3.4诊断 Java 语言代码中的泄漏

**NetBeans Profiler：** NetBeans Profiler 可以非常快速地定位内存泄漏。商业内存泄漏调试工具可能需要很长时间才能定位大型应用程序中的泄漏。但是，NetBeans Profiler 使用此类对象通常展示的内存分配和回收模式。此过程还包括缺少内存回收。探查器可以检查这些对象的分配位置，这通常足以确定泄漏的根本原因。

**jhat 实用程序：**该`jhat`实用程序在调试无意的对象保留（或内存泄漏）时很有用。它提供了一种浏览对象转储、查看堆中所有可访问对象以及了解哪些引用使对象保持活动状态的方法。

#### 3.4.1创建堆转储

如果 HPROF 与应用程序一起启动，则它可以创建堆转储。[示例 3-1](https://docs.oracle.com/javase/8/docs/technotes/guides/troubleshoot/memleaks004.html#CIHCGHJI)显示了该命令的用法。

```
$ java -agentlib:hprof=file=snapshot.hprof,format=b应用程序 
```

该`jmap`实用程序可用于创建正在运行的进程的堆转储。

```
jcmd <process id/main class> GC.heap_dump filename=Myheapdump 
$ jmap -dump:format=b,file=snapshot.jmap pid
```

- 创建堆转储的另一种方法是使用 JConsole 实用程序。在**MBeans**选项卡中，选择**HotSpotDiagnostic** MBean，然后显示**Operations**，并选择**dumpHeap**操作。

- 如果`-XX:+HeapDumpOnOutOfMemoryError`在运行应用程序时指定命令行选项，则在`OutOfMemoryError`抛出异常时，JVM 将生成堆转储。

  #### 3.4.2获取堆直方图

  如果 Java 进程是使用`-XX:+PrintClassHistogram`命令行选项启动的，那么 Control+Break 处理程序将生成一个堆直方图。

  您可以使用该`jmap`实用程序从正在运行的进程中获取堆直方图：

  ```
  jcmd <process id/main class> GC.class_histogram filename=Myheaphistogram 
  jmap -histo pid	
  ```

您可以使用该`jmap`实用程序从核心文件中获取堆直方图，如[示例 3-5](https://docs.oracle.com/javase/8/docs/technotes/guides/troubleshoot/memleaks004.html#CIHIEFIJ)所示。

```
jmap -histo core_file 
```

- [示例 3-6](https://docs.oracle.com/javase/8/docs/technotes/guides/troubleshoot/memleaks004.html#CIHDDAEA)显示`OutOfMemoryError`异常是由`java.lang.String`对象数量（堆中的 3,611,828 个实例）引起的。没有进一步的分析，目前还不清楚这些字符串的分配位置。但是，这些信息仍然有用。要继续工具，如调查`HPROF`和`jhat`寻找到的字符串分配，以及什么引用保持它们活着，防止它们被垃圾收集。

  #### 3.4.3监控待完成的对象

  当 OutOfMemoryError 异常与“Java 堆空间”详细消息一起抛出时，原因可能是过度使用终结器。要对此进行诊断，您可以使用多种选项来监视待定完成的对象数量：

  - 所述[的JConsole](https://docs.oracle.com/javase/8/docs/technotes/guides/troubleshoot/tooldescr009.html#BABDCICF)管理工具可用于监测所挂起终止的对象的数量。此工具会在“**摘要”**选项卡窗格的内存统计信息中报告挂起的完成计数。该计数是近似值，但可用于表征应用程序并了解它是否在很大程度上依赖于最终确定。
  - 所述[的JConsole](https://docs.oracle.com/javase/8/docs/technotes/guides/troubleshoot/tooldescr009.html#BABDCICF)管理工具可用于监测所挂起终止的对象的数量。此工具会在“**摘要”**选项卡窗格的内存统计信息中报告挂起的完成计数。该计数是近似值，但可用于表征应用程序并了解它是否在很大程度上依赖于最终确定。
  - 应用程序可以使用类的`getObjectPendingFinalizationCount`方法报告等待完成的对象的大致数量`java.lang.management.MemoryMXBean`。可以在[自定义诊断工具中](https://docs.oracle.com/javase/8/docs/technotes/guides/troubleshoot/tooldescr021.html#BABEDEBD)找到 API 文档和示例代码的链接。示例代码可以很容易地扩展到包括挂起完成计数的报告。

## 第二部分 调试 JVM 问题

#### 6 对进程挂起和循环进行故障排除

有时，明显的挂起实际上是一个循环。例如，VM 进程中导致一个或多个线程进入无限循环的错误可能会消耗所有可用的 CPU 周期。

诊断挂起的第一步是确定 VM 进程是否空闲或消耗了所有可用的 CPU 周期。您可以使用本机操作系统 (OS) 实用程序执行此操作。如果进程看起来很忙并且正在消耗所有可用的 CPU 周期，那么问题很可能是循环线程而不是死锁。例如，在 Oracle Solaris 操作系统上，`prstat -L -p` pid命令可用于报告目标进程中所有轻量级进程 (LWP) 的统计信息，从而识别消耗大量 CPU 周期的线程。

##### 6.1诊断循环过程

如果 VM 进程似乎在循环，则第一步是尝试获取线程转储。如果可以获得线程转储，通常很清楚哪个线程在循环。如果可以识别循环线程，则线程转储中的跟踪堆栈可以提供有关线程在何处（以及为什么）循环的方向。

如果可以获得线程转储，那么最好从处于该`RUNNABLE`状态的线程的线程堆栈开始。有关[线程转储](https://docs.oracle.com/javase/8/docs/technotes/guides/troubleshoot/tooldescr019.html#BABBEFFC)格式的信息以及线程转储中可能的线程状态表，请参阅线程转储。在某些情况下，可能需要获取一系列线程转储以确定哪些线程似乎一直处于忙碌状态。

在查看`jstack`实用程序的输出时，首先关注处于该`RUNNABLE`状态的线程。对于繁忙且可能循环的线程来说，这是最可能的状态。可能需要执行`jstack`多次才能更全面地了解哪些线程正在循环。如果线程似乎始终处于该`RUNNABLE`状态，则该`-m`选项可用于打印本机帧并提供有关线程正在执行的操作的进一步提示。如果线程在该`RUNNABLE`状态下似乎持续循环，则这种情况可能表明存在需要进一步调查的潜在 HotSpot VM 错误。

##### 6.2.1检测到死锁

##### 6.2.3 无线程转储

如果某个线程处于该`BLOCKED`状态并且原因不清楚，则使用该`-m`选项获取混合堆栈。使用混合堆栈输出，应该可以确定线程被阻塞的原因。如果线程在尝试进入同步方法或块时被阻塞，那么您将看到诸如`ObjectMonitor::enter`堆栈顶部附近的帧。[示例 6-2](https://docs.oracle.com/javase/8/docs/technotes/guides/troubleshoot/hangloop002.html#CEGHAEHJ)显示了一个示例混合堆栈输出。

## 第三部分调试核心库问题

## ~~第四部分调试客户端问题~~

## 第五部分提交错误报告

