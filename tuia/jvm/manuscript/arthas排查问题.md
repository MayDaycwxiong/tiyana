## 启动

java -Xmx200M -Xms200M -XX:+PrintGC -XX:+PrintGCDetails -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/Users/cuiwx/Downloads/heap -jar XXX.jar

#### 查看arthas 命令

help

## 数据面板

dashboard --内存增长，gc日志查看

# 源码

jad --source-only com.example.springdemo.Hello

# 内存

#### 查看jvm 的信息

jvm

#### jvm诊断相关：查看所有option

vmoption

#### 查看指定的option

vmoption HeapDumpOnOutOfMemory

#### 更新指定的option

vmoption PrintGCDateStamps true

#### GC前后打印类直方图，统计每个类的加载数量占用内存大小

- 打开`PrintClassHistogramBeforeFullGC`开关，可以在GC前打印类直方图
- 打开`PrintClassHistogramAfterFullGC`开关，可以在GC结束后打印类直方图
- `vmtool --action forceGc`强制GC

#### dump 到指定文件

heapdump /Users/cuiwx/Downloads/heap/arthas.hprof

# 线程

### 按cpu增量时间排序，只显示页码

thead

### 显示所有匹配的线程

thead -all

#### 查看指定状态的线程

thread --state waiting

### 最繁忙的前3个线程

thread -n 3

### 显示指定的线程运行堆栈

thead id 

### 寻找阻塞其他线程的线程，支持是synchronized组塞住的线程

thead -b

#### 采样：1000ms内的线程占用CPU时间

thead -i 1000

#### 1000ms 内最忙的3个线程

thread -n 3 -i 1000

# Mat 分析

思路

1. 内存占用过大的对象是什么？--MAT-histogram 来进行查询
2. 这个对象被谁引用？MAT-dominator_tree,用来分析对象的调用链
3. 定位到具体代码？MAT-thread_overView，线程简介图，这里面有方法的调用链信息和堆栈信息

# jdk工具

### 启动jar

java -Xmx300M -Xms300M -XX:PrintGC -XX:PrintGCDetails

### jmap 定位内存溢出

top -pid <pid>

jcmd <pid> help

##### 确认jvm用的垃圾回收器

jcmd VM.flags <pid>

#### 查看进程详情

top -pid <pid> 

jmap -histo 1/jcmd <pid> GC.class_histogram

### 线程问题排查

1 top 命令查看cpu占用最多的进程--->找到<pid>

2 查看进程中有哪些线程--->top -Hp <pid>

3 线程<pid>转换为16进制-->printf '%x' <pid>

4 jstack -l <pid> >x.txt(jcmd <pid> Thread.print)

