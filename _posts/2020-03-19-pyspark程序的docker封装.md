---
layout: post
categories: 算法
---

把pyspark写的程序封装成docker，可以调用外面的spark集群

启动docker：docker run -v `pwd`:/tmp -it ml_platform_python:pre bash
```
pip install hdfs
pip install pypandoc
pip install pyspark
```

启动python程序，竟然成功了，就这么简单！
python tests/trainHelper_FPGrowth.py


# 在docker里面启动
现象：WARN TaskSchedulerImpl: Initial job has not accepted any resources; check your cluster UI to ensure that workers are registered and have sufficient resources

这个现象不太好定位,打开master的web UI==》打卡Application==》stdout、stderr===点击stderr可以看到下列日志，
```
Spark Executor Command: "/home/ian/installed/java/bin/java" "-cp" "/home/ian/installed/spark/conf/:/home/ian/installed/spark/jars/*:/home/ian/installed/hadoop/etc/hadoop/" "-Xmx4096M" "-Dspark.driver.port=35074" "org.apache.spark.executor.CoarseGrainedExecutorBackend" "--driver-url" "spark://CoarseGrainedScheduler@f54587fef5f6:35074" "--executor-id" "8" "--hostname" "10.1.192.119" "--cores" "72" "--app-id" "app-20200320141303-0053" "--worker-url" "spark://Worker@10.1.192.119:25091"
========================================

Exception in thread "main" java.lang.reflect.UndeclaredThrowableException
	at org.apache.hadoop.security.UserGroupInformation.doAs(UserGroupInformation.java:1713)
	at org.apache.spark.deploy.SparkHadoopUtil.runAsSparkUser(SparkHadoopUtil.scala:64)
Caused by: java.io.IOException: Failed to connect to f54587fef5f6:35074
	at org.apache.spark.network.client.TransportClientFactory.createClient(TransportClientFactory.java:245)
	at org.apache.spark.network.client.TransportClientFactory.createClient(TransportClientFactory.java:187)
	at org.apache.spark.rpc.netty.NettyRpcEnv.createClient(NettyRpcEnv.scala:198)
	at org.apache.spark.rpc.netty.Outbox$$anon$1.call(Outbox.scala:194)
```

其原因在于：
driver启动==>cluster Master进行资源分配==>worker连接dirver。若driver在docker内启动的话，需要暴露对应端口（指定侦听端口+暴露侦听端口）

## 解决方法：


docker-compose的配置如下
```yaml
ports:
- 7077:7077
- 20002:20002
- 6060:6060
```
驱动程序的配置如下
```
esSparkConf.setMaster("spark://127.0.0.1:7077");
esSparkConf.setAppName("datahub_dev");

esSparkConf.setIfMissing("spark.driver.port", "20002");
esSparkConf.setIfMissing("spark.driver.host", "MAC_OS_LAN_IP");
esSparkConf.setIfMissing("spark.driver.bindAddress", "0.0.0.0");
esSparkConf.setIfMissing("spark.blockManager.port", "6060");
```
## 
```bash
docker run -v `pwd`:/tmp -it \
-p 20002:20002 \
-p 26060:26060 \
-e SPARK_MASTER="spark://10.1.192.118:7077" \
-e spark.driver.port=20002  \
-e spark.driver.host=10.1.192.120   \
-e spark.driver.bindAddress=0.0.0.0 \
-e spark.blockManager.port=26060 \
-e HDFS_MASTER_HOST=10.1.192.118 \
-w /tmp \
ml_platform_python:pre python ./tests/trainHelper_FPGrowth.py
```
docker run -v `pwd`:/tmp -it -w /tmp \
ml_platform_python:pre python ./tests/trainHelper_FPGrowth.py

## 注意worker和driver的版本，必须是3.6版本
Exception: Python in worker has different version 3.6 than that in driver 3.7, PySpark cannot run with different minor versions.Please check environment variables PYSPARK_PYTHON and PYSPARK_DRIVER_PYTHON are correctly set


# 参考：
- [Running Spark driver program in Docker container - no connection back from executor to the driver?](https://stackoverflow.com/questions/45489248/running-spark-driver-program-in-docker-container-no-connection-back-from-execu)