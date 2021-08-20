# 安装`arthas`

```sh
sh-4.2# curl -O https://arthas.aliyun.com/arthas-boot.jar
sh-4.2# java -jar arthas-boot.jar
```

# `logger`

动态更新日志级别

```sh
[arthas@1]$ logger --name ROOT --level debug
```

# `jvm`相关

## `dashboard`-当前系统的实时数据面板

> eg:https://arthas.aliyun.com/doc/dashboard.html

```sh
[arthas@1]$ dashboard
```

![image-20210819192508877](/Users/cuiwx/Library/Application Support/typora-user-images/image-20210819192508877.png)

## `jvm`-查看当前`JVM`信息

> eg:https://arthas.aliyun.com/doc/jvm.html

启动参数、环境变量、类加载信息、编译器、垃圾收集器信息、内存使用情况、操作系统、线程信息

```sh
[arthas@1]$ jvm
```

![image-20210819192758984](/Users/cuiwx/Library/Application Support/typora-user-images/image-20210819192758984.png)

## `vmoption`-查看，更新`VM`诊断相关的参数

> eg:https://arthas.aliyun.com/doc/vmoption.html
>

#### 查看所有的`option`

```sh
[arthas@1]$ vmoption
```

![image-20210819193750326](/Users/cuiwx/Library/Application Support/typora-user-images/image-20210819193750326.png)

#### 查看指定`option`

```sh
[arthas@1]$ vmoption PrintGC
```

![image-20210819194052970](/Users/cuiwx/Library/Application Support/typora-user-images/image-20210819194052970.png)

#### 修改指定的`option`

```sh
[arthas@1]$ vmoption PrintGCDateStamps true
[arthas@1]$ vmoption PrintClassHistogramBeforeFullGC true
```

![image-20210819194232879](/Users/cuiwx/Library/Application Support/typora-user-images/image-20210819194232879.png)

## `vmtool`-利用`JVMTI`接口，实现查询内存对象，强制`GC`等功能

> eg:https://arthas.aliyun.com/doc/vmtool.html
>

#### 强制`GC`

```sh
[arthas@1]$ vmtool --action forceGc
```

![image-20210819195742703](/Users/cuiwx/Library/Application Support/typora-user-images/image-20210819195742703.png)

## `heapdump-dump java heap`, 类似`jmap`命令的`heap dump`功能

> eg:https://arthas.aliyun.com/doc/heapdump.html

#### `dump` 到指定文件

```sh
[arthas@1]$ heapdump /Users/cuiwx/Downloads/heap/arthas.hprof
```

![image-20210819195029562](/Users/cuiwx/Library/Application Support/typora-user-images/image-20210819195029562.png)

## `thread`-查看当前线程信息，查看线程的堆栈

> eg:https://arthas.aliyun.com/doc/thread.html
>

#### 显示第一页线程的信息

```sh
[arthas@1]$ thread
```

![image-20210819200500895](/Users/cuiwx/Library/Application Support/typora-user-images/image-20210819200500895.png)

#### 显示所有匹配的线程

```sh
[arthas@1]$ thread -all
```

![image-20210819200556522](/Users/cuiwx/Library/Application Support/typora-user-images/image-20210819200556522.png)

#### 查看指定状态的线程

```sh
[arthas@1]$ thread --state WAITING
```

![image-20210819200633029](/Users/cuiwx/Library/Application Support/typora-user-images/image-20210819200633029.png)

#### 展示当前最忙的前5个线程并打印堆栈

```sh
[arthas@1]$ thread -n 5
```

![image-20210819200706234](/Users/cuiwx/Library/Application Support/typora-user-images/image-20210819200706234.png)

#### 显示指定线程的运行堆栈

```sh
[arthas@1]$ thread 1
```

![image-20210819200754036](/Users/cuiwx/Library/Application Support/typora-user-images/image-20210819200754036.png)

#### 找出当前阻塞其他线程的线程

```sh
[arthas@1]$ thread -b
```

![image-20210819200904216](/Users/cuiwx/Library/Application Support/typora-user-images/image-20210819200904216.png)

#### 统计最近`1000ms`内的线程`CPU`时间

```sh
[arthas@1]$ thread -i 1000
```

![image-20210819200941797](/Users/cuiwx/Library/Application Support/typora-user-images/image-20210819200941797.png)

#### 列出1000ms内最忙的3个线程栈

```sh
[arthas@1]$ thread -i 1000 -n 3
```

![image-20210819201012924](/Users/cuiwx/Library/Application Support/typora-user-images/image-20210819201012924.png)

# `class/classloader`相关

## `jad`-反编译指定已加载类的源码

> eg:https://arthas.aliyun.com/doc/jad.html
>

#### 反编译`MaterialUnitDataService`

```sh
[arthas@1]$ jad cn.com.duiba.tuia.media.manager.service.adx.MaterialUnitDataService
explain:默认情况下会显示ClassLoader的信息
```

![image-20210818111214696](/Users/cuiwx/Library/Application Support/typora-user-images/image-20210818111214696.png)

#### 反编译`MaterialUnitDataService`仅显示源代码

```sh
[arthas@1]$ jad --source-only cn.com.duiba.tuia.media.manager.service.adx.MaterialUnitDataService
explain:jad 仅仅反编译源码
```

![image-20210818111934876](/Users/cuiwx/Library/Application Support/typora-user-images/image-20210818111934876.png)

#### 反编译指定的方法

```sh
[arthas@1]$ jad cn.com.duiba.tuia.media.manager.service.adx.MaterialUnitDataService buildDataCondition
explain:反编译指定函数
```

![image-20210818112310863](/Users/cuiwx/Library/Application Support/typora-user-images/image-20210818112310863.png)

## `mc`-内存编译器，内存编译`.java`文件为`.class`文件

> eg:https://arthas.aliyun.com/doc/mc.html
>

#### 反编译`.class`文件到/root/下

```sh
[arthas@1]$ jad --source-only cn.com.duiba.tuia.media.manager.service.media.impl.ExternalPlatformMediaAccountServiceImpl > /root/ExternalPlatformMediaAccountServiceImpl.java
explain:控制台输出内容重定向到指定文件。注意：mc命令有可能失败。如果编译失败可以在本地编译好.class文件，再上传到服务器
```

#### 编译`.class`到/root/下

```sh
[arthas@1]$ mc -d /root/ /root/ExternalPlatformMediaAccountServiceImpl.java
explain:编译指定.java文件到指定目录下
```

![image-20210818173153845](/Users/cuiwx/Library/Application Support/typora-user-images/image-20210818173153845.png)

## `retransform`-加载外部的`.class`文件，`retransform`到`JVM`里

> eg:https://arthas.aliyun.com/doc/retransform.html
>

#### `retransform`指定的`.class`文件

```sh
[arthas@1]$ retransform /root/cn/com/duiba/tuia/media/manager/service/media/impl/ExternalPlatformMediaAccountServiceImpl.class
explain:会热替换掉JVM中加载的类，退出arthas后，retransform过的类依然生效
```

![image-20210818173519997](/Users/cuiwx/Library/Application Support/typora-user-images/image-20210818173519997.png)

#### 查看`retransform entry`

```sh
[arthas@1]$ retransform -l
explain:每加载一个.class文件则记录一个retransform entry
```

![image-20210818173810629](/Users/cuiwx/Library/Application Support/typora-user-images/image-20210818173810629.png)

#### 显示触发`retransform`

```sh
[arthas@1]$ retransform --classPattern cn.com.duiba.tuia.media.manager.service.media.impl.ExternalPlatformMediaAccountServiceImpl
explain:对于同一个类，当存在多个 retransform entry时，如果显式触发 retransform ，则最后添加的entry生效(id最大的)
```

![image-20210818175648080](/Users/cuiwx/Library/Application Support/typora-user-images/image-20210818175648080.png)

# `monitor/watch/trace`相关

## `monitor`-方法执行监控

> eg:https://arthas.aliyun.com/doc/monitor.html
>

#### 监控`buildDataCondition`方法的调用情况

```sh
[arthas@1]$ monitor -c 5 cn.com.duiba.tuia.media.manager.service.adx.MaterialUnitDataService buildDataCondition
explain:-c 是指定监控周期，单位s，可以统计方法的调用次数，成功次数，失败次数，RT，失败比例
```

![image-20210818114822457](/Users/cuiwx/Library/Application Support/typora-user-images/image-20210818114822457.png)

#### 监控条件表达式过滤统计结果

```sh
[arthas@1]$ monitor -c 20 cn.com.duiba.tuia.media.manager.service.adx.MaterialUnitDataService buildDataCondition "params[0].dataType==1"
explain: 满足条件才会进行监控统计
```

![image-20210818142308184](/Users/cuiwx/Library/Application Support/typora-user-images/image-20210818142308184.png)

## `watch`-方法执行数据观测

> eg:https://arthas.aliyun.com/doc/watch.html
>

#### 观察方法的入参和返回值

```sh
[arthas@1]$ watch cn.com.duiba.tuia.media.manager.service.adx.MaterialUnitDataService getHologresResultDtoList "{params,returnObj}" -x 2
explain:-x 是指定输出结果的属性遍历深度，默认为 1
```

![image-20210818120318897](/Users/cuiwx/Library/Application Support/typora-user-images/image-20210818120318897.png)

#### 观察方法入参

```sh
[arthas@1]$ watch cn.com.duiba.tuia.media.manager.service.adx.MaterialUnitDataService getHologresResultDtoList "{params,returnObj}" -x 2 -b 
eg:观察方法入参(返回值为空（事件点为方法执行前，因此获取不到返回值)。-b 是在方法掉用之前观察；-s 是在方法返回之后观察；-f是在方法结束之后（正常返回/异常返回）观察；
```

![image-20210818135736768](/Users/cuiwx/Library/Application Support/typora-user-images/image-20210818135736768.png)

#### 同时观察方法调用前和调用后返回

```sh
[arthas@1]$ watch cn.com.duiba.tuia.media.manager.service.adx.MaterialUnitDataService getHologresResultDtoList "{params,target,returnObj}" -x 2 -b -s -n 2
explain:-n 表示只执行2次
```

![image-20210818140424450](/Users/cuiwx/Library/Application Support/typora-user-images/image-20210818140424450.png)

#### 观察满足条件表达式的方法参数

```sh
[arthas@1]$ watch cn.com.duiba.tuia.media.manager.service.adx.MaterialUnitDataService getHologresResultDtoList "{params,target,returnObj}" "params[0].dataType==4" -x 2 -v
explain:-v 则会打印Condition express的具体值和执行结果，方便确认
```

![image-20210818142842578](/Users/cuiwx/Library/Application Support/typora-user-images/image-20210818142842578.png)

#### 根据耗时进行过滤

```sh
[arthas@1]$ watch cn.com.duiba.tuia.media.manager.service.adx.MaterialUnitDataService getHologresResultDtoList "{params,target,returnObj}" "#cost>2000" -x 2 -v
explain:耗时大于2000ms的调用将会打印
```

![image-20210818143427685](/Users/cuiwx/Library/Application Support/typora-user-images/image-20210818143427685.png)

## `trace`-方法内部调用路径，并输出方法路径上的每个节点上耗时

> eg:https://arthas.aliyun.com/doc/trace.html

`trace` 能方便的帮助你定位和发现因 RT 高而导致的性能问题缺陷，但其每次只能跟踪一级方法的调用链路。

#### `trace`函数`getNotOrderByData`

```sh
[arthas@1]$ trace cn.com.duiba.tuia.media.manager.service.adx.MaterialUnitDataService getNotOrderByData
```

![image-20210818144132852](/Users/cuiwx/Library/Application Support/typora-user-images/image-20210818144132852.png)

#### `trace`多个类或者多个函数

```sh
[arthas@1]$ trace -E cn.com.duiba.tuia.media.manager.service.adx.MaterialUnitDataService getNotOrderByData|getHologresResultDtoList
```

![image-20210818145323068](/Users/cuiwx/Library/Application Support/typora-user-images/image-20210818145323068.png)

#### 动态`trace`

## `stack`-输出当前方法被调用的调用路径

> eg:https://arthas.aliyun.com/doc/stack.html

#### `stack`方法`buildDataCondition`的调用路径

```sh
[arthas@1]$ stack cn.com.duiba.tuia.media.manager.service.adx.MaterialUnitDataService buildDataCondition
```

![image-20210818145945836](/Users/cuiwx/Library/Application Support/typora-user-images/image-20210818145945836.png)

#### 根据条件表达式来过滤

```sh
[arthas@1]$ stack cn.com.duiba.tuia.media.manager.service.adx.MaterialUnitDataService buildDataCondition "{params[0].dataType==1}" -v ??????
```

## `tt`-方法执行数据的时空隧道，记录下指定方法每次调用的入参和返回信息，并能对这些不同的时间下调用进行观测

> eg:https://arthas.aliyun.com/doc/tt.html

#### 记录调用

```sh
[arthas@1]$ tt -t cn.com.duiba.tuia.media.manager.service.adx.MaterialUnitDataService getHologresResultDtoList
```

![image-20210818152858603](/Users/cuiwx/Library/Application Support/typora-user-images/image-20210818152858603.png)

#### 搜索调用记录

```sh
[arthas@1]$ tt -l
```

![image-20210818152946082](/Users/cuiwx/Library/Application Support/typora-user-images/image-20210818152946082.png)

#### 筛选出`getHologresResultDtoList`方法的调用信息

```sh
[arthas@1]$ tt -s 'method.name=="getHologresResultDtoList"'
```

![image-20210818153126363](/Users/cuiwx/Library/Application Support/typora-user-images/image-20210818153126363.png)

#### 查看具体的调用信息

```sh
[arthas@1]$ tt -i 1002
```

![image-20210818153304940](/Users/cuiwx/Library/Application Support/typora-user-images/image-20210818153304940.png)

#### 重做一次

```sh
[arthas@1]$ tt -i 1002 -p
```

![image-20210818153526085](/Users/cuiwx/Library/Application Support/typora-user-images/image-20210818153526085.png)

## `Profiler`-使用`async-profiler`生成火焰图

> eg:https://arthas.aliyun.com/doc/profiler.html

#### 启动`profiler`、获取已采集的`sample`的数量、查看`profiler`的状态、停止`profiler`

```sh
[arthas@50191]$ profiler start
Started [cpu] profiling
[arthas@50191]$ profiler status
[cpu] profiling is running for 321 seconds
[arthas@50191]$ profiler getSamples
1431
[arthas@50191]$ profiler stop
OK
profiler output file: /Users/cuiwx/Documents/IdeaProjects/springdemo/arthas-output/20210820-104550.svg
[arthas@50191]$ session (e48cc39b-d2b3-46a1-b4ea-6773ba368a61) is closed because server is going to shutdown.
cuiwx@cuiwxdeMacBook-Pro ~ %
```

![image-20210820105635983](/Users/cuiwx/Library/Application Support/typora-user-images/image-20210820105635983.png)

#### 通过浏览器查看`arthas-output`下面的`profiler`结果

```http
  http://localhost:3658/arthas-output/
```

![image-20210820105817351](/Users/cuiwx/Library/Application Support/typora-user-images/image-20210820105817351.png)

![image-20210820110137536](/Users/cuiwx/Library/Application Support/typora-user-images/image-20210820110137536.png)

