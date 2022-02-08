# JVM学习笔记

## java 参数
- [文档地址](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/index.html)
  - 包括启动参数等

### cup 飙高排查
- top后 
  按shift + p 查看cpu高占用进程
  按shift + m 查看高内存占用进程
  
- top 找 pid 如 31638
- 找线程。即tid
  top -H -p 15657
  top -H -p 31638
- 转16进制 1401 -> 579
  Printf “%x\n” 1401
  jstack 31638 | grep 579
- 上下10行也看看
  jstack jstack 31638 | grep 579 -A 10 -B 10
- 导出
  jstack 31638  >> 123.txt

### 查看某线程堆栈使用情况
jmap -heap pid
### 查看使用的是什么收集器
java -XX:+PrintCommandLineFlags -version
|参数	|描述|
|----|----|
|UseSerialGC|虚拟机运行再Client模式下的默认值，打开此开关后，使用Serial+Serial Old的收集器组合进行内存回收|
|UseParNewGC|打开此开关后，使用ParNew+Serial Old的收集器组合进行内存回收|
|UseConcMarkSweepGC	|打开此开关后，使用ParNew+CMS+Serial Old的收集器组合进行内存|
|UseParallelGC	|虚拟机运行在Server模式下的默认值，打开此开关后，使用ParallelSeavenge+ParallelOld的收集器组合进行内存回收|
|UseParallelOldGC	|打开此开关后，使用ParallelSeavenge+ParallelOld的收集器组合进行内存回收|
|SurvivorRatio	|新生代中Eden区域与Survivor区域的容量比值，默认为8，代表Eden：Survivor=8：1|
|PretenureSizeThreshold|	直接晋升到老年代的对象大小，设置这个参数后，大于这个參数的对象将直接在老年代分配|
|MaxTenuringThreshold|	晋升到老年代的对象年龄。每个对象在坚持过一次MinorGC之后，年龄就增加1，当超过这个参数值时就进人老年代|
### jvm参数
`-jar -Xms3000M -Xmx3000M -Xmn1500M -Xss256k -XX:PermSize=100m -XX:MaxPermSize=300m
 -XX:-HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/home/web/cts_ac_back/cts_ac_outerapi/logs/gc/java_pid_%p.hprof -XX:+PrintGCTimeStamps -XX:+PrintGCDateStamps -XX:+PrintGCDetails -Xloggc:/home/web/cts_ac_back/cts_ac_outerapi/logs/gc/gc.log `
 
### [Arthas](https://arthas.aliyun.com/doc/advanced-use.html)

- 安装

  ```
  curl -L https://arthas.aliyun.com/install.sh | sh
  ```

- 运行

  ```
  sh as.sh
  ```

- 安装idea插件

  安装两个插件“arthas idea” 和 “ArthasHotSwap”

- 获取观测命令

  方法上右键 -> arthas -> watch

- 执行命令

  ```
  watch com.nowcoder.cts.ac.task.service.impl.CodeStyleCheckServiceImpl doCheckStatus '{params,returnObj,throwExp}'  -n 5  -x 3 
  'params[0].getPlayToolDO().getId()==588'
  ```

  - 解释

    '{params,returnObj,throwExp}' 表示观测对象，这里选择了入参、出参和异常，-n 5 表示观测到5次后停止，-x 3 表示打印观测对象的时候只打印3层嵌套结构，'params[0].getPlayToolDO().getId()==588' 为条件表达式，满足该条件的才会被观测到。

- 常用命令

  - dashboard

    看当前进程相关信息，id为线程id

    ```
    |  ①Thread相关信息                                                                                                                                                    
    |  线程id              线程名称                                                      线程组                                  线程优先级            线程状态             线程消耗的cpu百分比   运行总时间           线程当前的中断位状态    是否守护线程
    |  ID                  NAME                                                        GROUP                                   PRIORITY            STATE               %CPU                TIME                INTERRUPTED         DAEMON
    |  188                 Timer-for-arthas-dashboard-f5864b5b-762a-4fb5-8cc5-65559bd6 system                                  10                  RUNNABLE            19                  0:0                 false               true
    |  36                  pool-1-thread-1                                             main                                    5                   TIMED_WAITING       5                   0:1                 false               false
    |  33                  Abandoned connection cleanup thread                         main                                    5                   TIMED_WAITING       0                   0:0                 false               true
    |  179                 AsyncAppender-Worker-arthas-cache.result.AsyncAppender      system                                  9                   WAITING             0                   0:0                 false               true
    |  12                  AsyncFileHandlerWriter-225534817                            main                                    5                   TIMED_WAITING       0                   0:0                 false               true
    |  94                  Attach Listener                                             system                                  9                   RUNNABLE            0                   0:0                 false               true
    |  70                  ContainerBackgroundProcessor[StandardEngine[Catalina]]      main                                    5                   TIMED_WAITING       0                   0:0                 false               true
    |  34                  Druid-ConnectionPool-Create-300669762                       main                                    5                   WAITING             0                   0:0                 false               true
    |  35                  Druid-ConnectionPool-Destroy-300669762                      main                                    5                   TIMED_WAITING       0                   0:0                 false               true
    |  3                   Finalizer                                                   system                                  8                   WAITING             0                   0:0                 false               true
    |  13                  GC Daemon                                                   system                                  2                   TIMED_WAITING       0                   0:0                 false               true
    |  14                  NioBlockingSelector.BlockPoller-1                           main                                    5                   RUNNABLE            0                   0:0                 false               true
    |  15                  NioBlockingSelector.BlockPoller-2                           main                                    5                   RUNNABLE            0                   0:0                 false               true
    |  2                   Reference Handler                                           system                                  10                  WAITING             0                   0:0                 false               true
    |  4                   Signal Dispatcher                                           system                                  9                   RUNNABLE            0                   0:0                 false               true
    |  76                  ajp-nio-38009-Acceptor-0                                    main                                    5                   RUNNABLE            0                   0:0                 false               true
    |  74                  ajp-nio-38009-ClientPoller-0                                main                                    5                   RUNNABLE            0                   0:0                 false               true
    |  75                  ajp-nio-38009-ClientPoller-1                                main                                    5                   RUNNABLE            0                   0:0                 false               true
    |  187                 as-command-execute-daemon                                   system                                  10                  TIMED_WAITING       0                   0:0                 false               true
    |  73                  http-nio-37080-Acceptor-0                                   main                                    5                   RUNNABLE            0                   0:0                 false               true
    |  71                  http-nio-37080-ClientPoller-0                               main                                    5                   RUNNABLE            0                   0:0                 false               true
    |  72                  http-nio-37080-ClientPoller-1                               main                                    5                   RUNNABLE            0                   0:0                 false               true
    |
    |  ②内存信息                                                                                                               ③垃圾回收
    |  Memory                                             used              total            max              usage            GC
    |  （堆）                                                                                                                   （垃圾回收次数）
    |  heap                                               424M              1897M            1897M            22.37%           gc.ps_scavenge.count                                  19
    |  （伊甸园）                                                                                                                （垃圾回收消耗时间）
    |  ps_eden_space                                      311M              387M             403M             77.28%           gc.ps_scavenge.time(ms)                               1405
    |  （幸存者区）                                                                                                              （标记-清除算法的次数）
    |  ps_survivor_space                                  40M               144M             144M             27.74%           gc.ps_marksweep.count                                 3
    |  （老年代）                                                                                                               （标记-清除算法的消耗时间）
    |  ps_old_gen                                         72M               1365M            1365M            5.32%            gc.ps_marksweep.time(ms)                              446
    |  （非堆区）                                                                                                                                                                                                                                                                                                                                                                        
    |  nonheap                                            137M              141M             -1               97.49%                                                                                                                                   
    |  （代码缓存区）                                                                                                                                                                                                                                                                                                                                                                        
    |  code_cache                                         40M               41M              240M             16.99%                                                                                                                                   
    |  （元空间）                                                                                                                                                                                    
    |  metaspace                                          86M               89M              -1               97.09%                                                                                                                                   
    |  （压缩空间）                                                                                                                                                                                    
    |  compressed_class_space                             10M               10M              1024M            0.99%                                                                                                                                    
    |  direct                                             80K               80K              -                100.00%                                                                                                                                  
    |  mapped                                             0K                0K               -                NaN%                                                                                                                                     
    |                                                                                                                                                                                      
    |  ④运行信息                                                                                                                                                                                    
    |  Runtime                                                                                                                                                                                                                                                                                                                                                                        
    |  os.name                                                                                                                 Linux                                                                                                                   
    |  os.version                                                                                                              3.10.0-957.1.3.el7.x86_64                                                                                               
    |  java.version                                                                                                            1.8.0_101                                                                                                               
    |  java.home                                                                                                               /opt/jdk1.8.0_101/jre                                                                                                   
    |  systemload.average                                                                                                      0.03                                                                                                                    
    |  processors                                                                                                              8                                                                                                                       
    |  uptime                                                                                                                  11956s                                                                                                                  
    |________________________________________________________________________________________________________________________________________________________________________________________________________________________________________
    
    说明
    ID: Java级别的线程ID，注意这个ID不能跟jstack中的nativeID一一对应
    NAME: 线程名
    GROUP: 线程组名
    PRIORITY: 线程优先级, 1~10之间的数字，越大表示优先级越高
    STATE: 线程的状态
    CPU%: 线程消耗的cpu占比，采样100ms，将所有线程在这100ms内的cpu使用量求和，再算出每个线程的cpu使用占比。
    TIME: 线程运行总时间，数据格式为分：秒
    INTERRUPTED: 线程当前的中断位状态
    DAEMON: 是否是daemon线程
    
    通过上述信息，可以帮助我们快速定位相关问题线程。
    ```
