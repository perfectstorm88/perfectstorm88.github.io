---
layout: post
categories: 数据库 MySQL 大数据 JAVA
top: true
---

最早发布在[MyCat Server Issue中](https://github.com/MyCATApache/Mycat-Server/issues/303)
# Buffer机制
# HeapByteBuffer与DirectByteBuffer

## HeapByteBuffer与DirectByteBuffer介绍

ByteBuffer一般在网络交互过程中java使用得比较多，尤其是以NIO的框架中，分为两类：
HeapByteBuffer：内存是分配在堆上的，直接由Java虚拟机负责垃圾收集，你可以把它想象成一个字节数组的包装类，如下伪码所示：

``` java
HeapByteBuffer extends ByteBuffer {
    byte[] content;
    int position, limit, capacity;
    ......
}
```

DirectByteBuffer：是通过JNI在Java虚拟机外的内存中分配了一块（java堆内存由Xmx控制，而堆外内存由-XX:MaxDirectMemorySize控制），该内存块并不直接由Java虚拟机负责垃圾收集，但是在DirectByteBuffer包装类被回收时，会通过Java Reference机制来释放该内存块。如下伪码所示：

``` java
DirectByteBuffer extends ByteBuffer {
    long address;
    int position, limit, capacity;
    ......
}
```

除了上述区别，那么还有什么其他的区别呢？嘿嘿，让我们稍微深入一点，翻到sun.nio.ch.IOUtil.java，绝大部分Channel类都是通过这个工具类和外界进行通讯的，如FileChannel/SocketChannel等等。我简单的用伪码把write方法给表达出来（read方法也差不多，就不多做说明了）

``` java
int write(ByteBuffer src, ......) {
    if (src instanceof DirectBuffer)
        return writeFromNativeBuffer(...);
   ByteBufferdirect = getTemporaryDirectBuffer(src);
    writeFromNativeBuffer(direct,......);
    updatePosition(src);
    releaseTemporaryDirectBuffer(direct);
}
```

是的，在发送和接收前会把HeapByteBuffer转换为DirectByteBuffer，然后再进行相关的操作，最后更新原始ByteBuffer的position。这意味着什么？假设我们要从网络中读入一段数据，再把这段数据发送出去的话，采用HeapByteBuffer的流程是这样的：

**网络 --> 临时的DirectByteBuffer --> 应用HeapByteBuffer --> 临时的DirectByteBuffer --> 网络**

而采用DirectByteBuffer的流程是这样的：

**网络 --> 应用 DirectByteBuffer --> 网络**

可以看到，除开构造和析构临时DirectByteBuffer的时间外，起码还能节约两次内存拷贝的时间。那么是否在任何情况下都采用DirectBuffer呢？

不是。对于大部分应用而言，两次内存拷贝的时间几乎可以忽略不计，而构造和析构DirectBuffer的时间却相对较长,意味着如果采用DirectByteBuffer仅仅能节约掉两次内存拷贝的时间，而无法节约构造和析构的时间。

网络上整理的使用建议如下：
- 如果你做中小规模的应用（在这里，应用大小是按照使用ByteBuffer的次数和规模来做划分的），而且并不在乎这该死的细节问题，请选择HeapByteBuffer·  
- 如果采用DirectByteBuffer后性能并没有出现你所期待的变化，请选择HeapByteBuffer·
- 如果没有 **DirectByteBuffer Pool**，尽量不要使用DirectByteBuffer·除非你确定该ByteBuffer会长时间存在，并且和外界有频繁交互，可采用DirectByteBuffer·  
- 如果采用HeapByteBuffer，那么采用非聚集(gather)的write/read(ByteBuffer)效果反而可能超出聚集的write/read(ByteBuffer[])，因为聚集的write/read的临时DirectByteBuffer是非缓存的.

基本上，采用HeapByteBuffer总是对的！因为内存拷贝需要的开销对大部分应用而言都可以忽略不计。如果开发的是大规模的网络并发框架，对这些细节问题还是有必要有深入认识的，并且根据这些细节来调节自己的Buffer继承体系。
## MyCAT的Buffer体系

在MyCAT的NIO框架中，DirectByteBuffer和HeapByteBuffer都进行了使用：
- 优先使用DirectByteBuffer进行协议包的接受和发送，并且所有的DirectByteBuffer通过pool进行管理。
- 通过pool管理的DirectByteBuffer都是固定大小，通过参数chunksize配置，默认为16KB
- 大多数场景下DirectByteBuffer足够装下一个完整报文，当报文较小时，一次对DirectByteBuffer的操作可以读取或者写入多个报文。
- 较大报文一般指Row Data 报文，数据库表的每条记录内容对应一个Row Data，如果单条记录的字段数较多或者字段长度非常大，会导致Row Data 报文超过DirectByteBuffer的长度，此时一个DirectByteBuffer不能放下一个完整的MySQL协议报文，会临时采用HeapByteBuffer，处理完该报文即由GC回收
  ![mycatbuffer管理机制](http://img.lichangzhen.top/mycatmycatbuffer管理机制.jpg)
## DirectBuffer管理

**Direct内存设置**
- DirectByteBuffer的管理区域不是在java heap上，不能通过Xmx控制，可以通过参数：-XX:MaxDirectMemorySize来控制

**Direct内存监控**
Direct内存信息不同通过Runtime.getRuntime()获取到，但可以通过下面方法间接取到。

``` java
    /**
     * 打印DirectMemory信息
     * @throws Exception
     */
    private void printDirectBufferInfo() throws Exception 
    {
        Class<?> mem = Class.forName("java.nio.Bits");
        Field maxMemoryField = mem.getDeclaredField("maxMemory");
        maxMemoryField.setAccessible(true);
        Field reservedMemoryField = mem.getDeclaredField("reservedMemory");
        reservedMemoryField.setAccessible(true);        
        Field totalCapacityField = mem.getDeclaredField("totalCapacity");
        totalCapacityField.setAccessible(true);
        Long maxMemory = (Long) maxMemoryField.get(null);
        Long reservedMemory = (Long) reservedMemoryField.get(null);
        Long totalCapacity = (Long) reservedMemoryField.get(null);
        System.out.println("maxMemory="+maxMemory+",reservedMemory="+reservedMemory
                +",totalCapacity="+totalCapacity);
    }
```

**Direct内存回收**
- 直接内存的释放并不是由普通gc控制的，而是由full gc来控制的，直接内存会自己检测情况而调用system.gc()，但是如果参数中使用了DisableExplicitGC那么这是个坑了，所以啊，这玩意，设置不设置都是一个坑
- 那么full gc不触发，想自己释放这部分内存有方法吗？可以的。看它的源码中DirectByteBuffer发现有一个：Cleaner，是用来搞资源回收的。这个代码私有的，不能直接访问，需要通过反射来实现，通过调用cleaner()方法来获取它Cleaner对象，进而通过该对象，执行clean方法。

``` java
    /**
     * 显式清理
     * @param byteBuffer
     */
    @SuppressWarnings("restriction")
    public static void clean(final ByteBuffer byteBuffer) {
          if (byteBuffer.isDirect()) {
             ((sun.nio.ch.DirectBuffer)byteBuffer).cleaner().clean();
          }
    } 
    //测试人工调用DirectBuffer的清理
    @Test
    public void testDirectBufferClean() throws Exception {  
           ByteBuffer buffer = ByteBuffer.allocateDirect(1024 * 1024 * 1000);  
           System.out.println("start");  
           Thread.sleep(5000); 
           printDirectBufferInfo();
           clean(buffer);  
           System.out.println("end"); 
           printDirectBufferInfo();
           Thread.sleep(5000);  
    } 
```
- MyCAT的做法：通过pool对DirectByteBuffer进行管理，尽可能杜绝Direct内存回收操作 
# Sharing+LocalPool

除了采用DirectBuffer外，MyCAT采用了Sharing+LocalPool机制来提高Buffer对象的存取速度，原理如下图所示。其中LocalPool为单线程独有，对LocalPool的操作是无锁操作，只有LocalPool取不到时，再到共享的SharingPool部分去获取。存的动作类似，也是先存到LocalPool，LocalPool满，再存到SharingPool。
![sharingAndLocalPool](http://img.lichangzhen.top/mycatsharingAndLocalPool1.jpg)

在Mycat改造为Sharing+LocalPool机制之前，采用的是手写PoolByLock，即通过定长数组管理资源池，每次存取都进行ReentrantLock锁操作。

下面的性能测试对Sharing+LocalPool机制和PoolByLock进行了比较，另外还与JDK自带的ConcurrentLinkedQueue、ConcurrentLinkedDeque、LinkedBlockingQueue进行了比较。

详细测试数据表格如下：

| threadnum | PoolByLock | Sharing+LocalPool | ConcurrentLinkedQueue | ConcurrentLinkedDeque | LinkedBlockingQueue |
| --- | --- | --- | --- | --- | --- |
| 1 | 23562 | 35435 | 26968 | 15797 | 12600 |
| 2 | 3744 | 71839 | 6032 | 4595 | 2704 |
| 3 | 13732 | 41254 | 6754 | 5235 | 9286 |
| 4 | 14513 | 46685 | 7540 | 6195 | 11003 |
| 5 | 13075 | 43215 | 7697 | 6396 | 9237 |
| 6 | 14425 | 63371 | 7762 | 6491 | 10699 |
| 7 | 13412 | 51124 | 8094 | 6544 | 11271 |
| 8 | 14068 | 71942 | 7947 | 6449 | 10371 |
| 9 | 14261 | 65876 | 7983 | 6619 | 9159 |
| 10 | 14249 | 62656 | 8059 | 6597 | 11856 |
| 20 | 13873 | 60313 | 8064 | 6811 | 10711 |
| 30 | 14261 | 69930 | 8269 | 6740 | 8435 |
| 40 | 14144 | 72046 | 8350 | 7339 | 9085 |
| 50 | 14434 | 55991 | 8097 | 6626 | 9609 |
| 60 | 13721 | 60168 | 8214 | 7405 | 9552 |
| 80 | 13484 | 63291 | 8159 | 7367 | 8435 |
| 100 | 12777 | 61957 | 8115 | 7373 | 8483 |
| 150 | 12318 | 71123 | 7918 | 7457 | 8033 |
| 200 | 13954 | 74626 | 8032 | 7419 | 8176 |
| 300 | 12966 | 80515 | 8136 | 6735 | 7272 |
| 500 | 12440 | 78003 | 8136 | 6729 | 5474 |

比较图表如下：
![各种队列性能比较](http://img.lichangzhen.top/mycat队列性能比较.JPG)

结论：
1、经过测试证明，在多线程并发情况下，Sharing+LocalPool具有非常优秀的存取性能，性能约为JDK自带ConcurrentLinkedQueue、ConcurrentLinkedDeque、LinkedBlockingQueue的10倍左右，是手写PoolByLock的5~6倍。
2、与JDK自带并发包中的API相比，手写PoolByLock还是具有一定的性能优势。
3、LinkedBlockingQueue和ConcurrentLinkedQueue相比，在不同的线程并发数下有不同的表现，但基本接近。
4、ConcurrentLinkedQueue比ConcurrentLinkedDeque略高，毕竟双向队列还是增加一些逻辑判断，如业务场景要求双向队列，这个性能差异可以忽略掉性能接近。
5、总体比较：
**Sharing+LocalPool >> 手写PoolByLock > LinkedBlockingQueue ≈ ConcurrentLinkedQueue > ConcurrentLinkedDeque**

上述测试代码在[ashkritblog](https://github.com/cangzhen/ashkritblog)中的pooltest目录下。
另外对Sharing+LocalPool机制进行了泛型封装，代码在sharinglocalpool中。
# java.lang.OutOfMemoryError: Direct buffer memory

在运行过程中可能会出现java.lang.OutOfMemoryError: Direct buffer memory异常

``` java
java.lang.OutOfMemoryError: Direct buffer memory
at java.nio.Bits.reserveMemory(Bits.java:658)
at java.nio.DirectByteBuffer.<init>(DirectByteBuffer.java:123)
at java.nio.ByteBuffer.allocateDirect(ByteBuffer.java:306)
```
## 一般检查方法
1. -XX:MaxDirectMemorySize加大，可以根据需求调大试试；根据Sharing+LocalPool机制可知，每个线程会保留一个ThreadLocal的Buffer池，在极端情况下，可能所有的Buffer都在LocalPool中，而Sharing一个对象也没有！所以一定要保证下面公式成立：

> 使用LocalPool的线程数 \* 每个LocalPool的Buffer个数上限 \* 每个buffer大小 > MaxDirectMemorySize
## 并发执行大表select操作会出现OOM异常

压测过程发现，并发执行大表的select \* 操作，会出现java.lang.OutOfMemoryError: Direct buffer memory异常，然后系统挂死.通过show @@bufferpool和show @@connection命令，发现buffer都堆积在每个链接的写队列上。写队列采用的是 **无限制写队列**

``` java
    protected final ConcurrentLinkedQueue<ByteBuffer> writeQueue = new ConcurrentLinkedQueue<ByteBuffer>();
```

**无限制写队列**
当多客户端并发执行大表的select \* 时，从后端读完后直接写到前端。当写速度跟不上时，buffer在写队列缓存，会导致写队列越来越大，最终出现OOM异常，挂死。

![buffer无长度限制写队列](http://img.lichangzhen.top/mycatmycatbuffer无长度限制写队列.jpg)

**有限制写队列**
参考cobar，cobar采用的是有长度限制的写队列，写操作时发现写队列已满，线程堵塞；其它线程(W线程或者R线程)写完后，会唤醒堵塞在写队列的线程。
采用这种方式时，必须保证读写线程、业务执行线程分离，以保证写队列满时，只会堵塞业务线程。当读线程和业务线程分离后，为了保证后端返回的数据依然，必须增加逻辑：读后的数据包先顺序入另一个业务数据队列，再由业务线程按顺序执行，这个逻辑会大幅增加调度成本。这个逻辑代码如下

``` java
    protected final BlockingQueue<byte[]> dataQueue = new LinkedBlockingQueue<byte[]>();
    protected final AtomicBoolean isHandling = new AtomicBoolean(false);

    protected void offerData(byte[] data, Executor executor) {
        if (dataQueue.offer(data)) {
            handleQueue(executor);
        } else {
            offerDataError();
        }
    }
    protected void handleQueue(final Executor executor) {
        if (isHandling.compareAndSet(false, true)) {
            // 异步处理后端数据
            executor.execute(new Runnable() {
                @Override
                public void run() {
                    try {
                        byte[] data = null;
                        while ((data = dataQueue.poll()) != null) {
                            handleData(data);
                        }
                    } catch (Throwable t) {
                        handleDataError(t);
                    } finally {
                        isHandling.set(false);
                        if (dataQueue.size() > 0) {
                            handleQueue(executor);
                        }
                    }
                }
            });
        }
    }
```

另外，核心问题是，虽然写队列有长度限制，但是业务数据队列(BlockingQueue<byte[]>)又是无长度限制的，另外业务线程池的任务缓冲队列也是无长度限制的（LinkedBlockingQueue<Runnable>），所以当前端写操作慢时，仍然会出现OOM异常。
由于业务数据队列存储的是解析后的byte[],所以此处的OOM是堆内存溢出，而非直接内容溢出
![buffer有长度限制写队列](http://img.lichangzhen.top/mycatmycatbuffer有长度限制写队列.jpg)

**折中方案**
当前端写操作慢时，都不可避免导致OOM异常，系统挂死；但是只要把大量buffer堆积写队列的链路从系统中剔除后，系统能够正常恢复，从可用性角度仍然可以接受，可以选择下面两种方法：
- `方法1:当发生OOM异常后，DBA通过kill命令杀死堆积链路,人工恢复` 
- `方法2:在向写队列放入buffer前，判断写队列大小，若超过预定长度，系统自动强制关闭堆积链路` 

**个人觉得，方法1在实际中使用更有效果一些！**

# 参考
- [java之HeapByteBuffer&DirectByteBuffer以及回收DirectByteBuffer](http://blog.csdn.net/xieyuooo/article/details/7547435)
- [转载-Non-direct与direct ByteBuffer区别](http://blog.csdn.net/lovehota/article/details/7407574)
