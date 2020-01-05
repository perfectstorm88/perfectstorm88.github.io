---
layout: post
categories: 数据库 MySQL 大数据 JAVA 
---

最早发布在[MyCat Server Issue中](https://github.com/MyCATApache/Mycat-Server/issues/292)

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
