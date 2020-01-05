---
layout: post
categories: JAVA 性能调优
---

#Java Performance笔记一
##java监控工具
- jps  查看java进程号
- jcmd 打印java进程的基本类、线程、VM信息
- jhat 后加工工具，分析内存dump
- jmap 可以在线dump内存
- jinfo 查看jvm系统参数，可以动态设置参数
- jstat 可以查看gc和类加载情况
- jstack 查看线程堆栈情况
- jconsole 傻瓜式工具
- jvisualvm  傻瓜式工具，功能更强大，可以在线dump内存堆栈,也可以提供后处理工具

##基本vm信息
下面命令中，其中的5090是进程号
**VM启动时长**
- jcmd process_id VM.uptime
- 
**JVM版本**
- jcmd 5090 VM.version

**JVM 命令行**
- jcmd 5090 VM.command_line

**系统属性**
- jcmd 5090  VM.system_properties #输出结果等同于通过代码调用System.getProperties()
- 或者 jinfo -flags 5090
**注意:**jinfo要求jinfo版本和java进程的jvm版本要相同，否则会抛下面异常
```
/Users/lcz/remote_login>jinfo -sysprops 5090
Attaching to process ID 5090, please wait...
Exception in thread "main" java.lang.reflect.InvocationTargetException
    at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
    at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
    at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
    at java.lang.reflect.Method.invoke(Method.java:497)
    at sun.tools.jinfo.JInfo.runTool(JInfo.java:108)
    at sun.tools.jinfo.JInfo.main(JInfo.java:76)
Caused by: java.lang.InternalError: void* type hasn't been seen when parsing int*
    at sun.jvm.hotspot.HotSpotTypeDataBase.recursiveCreateBasicPointerType(HotSpotTypeDataBase.java:687)
    at sun.jvm.hotspot.HotSpotTypeDataBase.lookupType(HotSpotTypeDataBase.java:131)
    at sun.jvm.hotspot.HotSpotTypeDataBase.lookupOrCreateClass(HotSpotTypeDataBase.java:597)
    at sun.jvm.hotspot.HotSpotTypeDataBase.createType(HotSpotTypeDataBase.java:717)
    at sun.jvm.hotspot.HotSpotTypeDataBase.readVMTypes(HotSpotTypeDataBase.java:188)
    at sun.jvm.hotspot.HotSpotTypeDataBase.<init>(HotSpotTypeDataBase.java:86)
    at sun.jvm.hotspot.HotSpotAgent.setupVM(HotSpotAgent.java:403)
    at sun.jvm.hotspot.HotSpotAgent.go(HotSpotAgent.java:305)
    at sun.jvm.hotspot.HotSpotAgent.attach(HotSpotAgent.java:140)
    at sun.jvm.hotspot.tools.Tool.start(Tool.java:185)
    at sun.jvm.hotspot.tools.Tool.execute(Tool.java:118)
    at sun.jvm.hotspot.tools.JInfo.main(JInfo.java:138)
    ... 6 more
```

**JVM 调优标志**
The tuning flags in effect for an application can be obtained like this:
% jcmd process_id VM.flags [-all]
```bash
/Users/lcz/remote_login>jcmd 5090 VM.flags
5090:
-XX:InitialHeapSize=134217728 -XX:MaxHeapSize=2147483648 -XX:+UseCompressedOops -XX:+UseParallelGC 
/Users/lcz/remote_login>jcmd 5090 VM.flags -all
5090:
[Global flags]
    uintx AdaptivePermSizeWeight                    = 20              {product}           
    uintx AdaptiveSizeDecrementScaleFactor          = 4               {product}           
    uintx AdaptiveSizeMajorGCDecayTimeScale         = 10              {product}           
    uintx AdaptiveSizePausePolicy                   = 0               {product}           
    uintx AdaptiveSizePolicyCollectionCostMargin    = 50              {product}  
    uintx InitialHeapSize                          := 134217728       {product}  
    .....   
```

###如何使用调优标志

也可以通过下面命令，打印出平台默认值
java -XX:+PrintFlagsFinal -version
第一列表示类型，如bool intx uintx等
其中冒号表示这个标记是个非缺省值，如InitialHeapSize                          := 134217728  
值改变的原因：
- 在命令行中直接设置
- 一些选项因为其它选项值被间接修改
- JVM计算的默认值ergonomically

最后一列表示不同的平台，例如：
- `product` means that the default setting of the flag is uniform across all platforms; 
- `pd product `indicates that the default setting of the flag is platform-dependent.
 - `manageable` (the flag’s value can be changeddynamicallyduringruntime)
- ` C2 diagnostic`(theflagprovidesdiagnostic output for the compiler engineers to understand how the compiler is functioning)

`是不是太多信息？`
>例如AllocatePrefetchLines 默认值是3，若要修改这个值需要考虑1.应用的prefetch性能；2.CPU运行程序的特性；3.对JVM本身性能的影响

只有manageable的标志可以修改
```
[root@localhost ~]# java -XX:+PrintFlagsFinal -version|grep manageable
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

```

修改调优标志
```
jinfo
where <option> is one of:
    -flag <name>         to print the value of the named VM flag
    -flag [+|-]<name>    to enable or disable the named VM flag
    -flag <name>=<value> to set the named VM flag to the given value
    -flags               to print VM flags
[root@localhost ~]# jps
3145 Jps
2538 WrapperSimpleApp
[root@localhost ~]# jinfo -flag PrintGCDetails 2538
-XX:-PrintGCDetails
[root@localhost ~]# jinfo -flag +PrintGCDetails 2538
[root@localhost ~]# jinfo -flag PrintGCDetails 2538
-XX:+PrintGCDetails
[root@localhost ~]# jinfo -flag -PrintGCDetails 2538
[root@localhost ~]# jinfo -flag PrintGCDetails 2538
-XX:-PrintGCDetails
```

**线程信息**
- jstack 进程号 
另外可以参见第9章更详细信息

**内存分析**
- `jstat -gcutil 16002 1s` #观察内存变化情况
- jstat -gcutil `ps -ef|grep Dhotdb.home|grep -v grep|awk '{print $2}'` 1s #自动搜索pid然后跟踪内存 
- `jmap -dump:live,format=b,file=heap.bin 22741` #在线dump文件
- `jhat heap.bin` #观察内存变化情况
- 另外也可以通过Jconsole/JvisualVM直接dump，然后通过HPJmeter或者Eclipse的内存泄漏分析插件

**类信息**
jstat 
下面2538是线程号
```
[root@localhost ~]# jstat -class 2538
Loaded  Bytes  Unloaded  Bytes     Time   
  2312  4368.5        0     0.0       1.76
[root@localhost ~]# jstat -printcompilation  2538 
Compiled  Size  Type Method
    1325     89    1 sun/reflect/GeneratedMethodAccessor5 invoke
```
