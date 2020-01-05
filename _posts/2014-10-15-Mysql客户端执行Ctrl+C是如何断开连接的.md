---
layout: post
categories: 数据库 MySQL
---

这是用过MySQL的人都很熟悉的动作：通过命令行客户端连接服务器，若消息堵塞，执行Ctrl+C断开连接。
那么断开连接的过程和原理是什么呢？使用wireshark抓包工具，通过下面语句可以窥视其秘密

```sql
mysql> select sleep(100);
^CCtrl-C -- sending "KILL QUERY 97" to server ...
Ctrl-C -- query aborted.
ERROR 1317 (70100): Query execution was interrupted
mysql> Ctrl-C -- exit!
Aborted
```
其消息交互如下图所示：
当输入"Ctrl-C"时，客户端会用新的线程发起查询，同时服务器管理上一个链接。

![Mysql客户端执行Ctrl+C的后台消息交互时序图](../../../../images/Mysql客户端执行Ctrl+C的后台消息交互时序图.jpg)



参见:[Mysql客户端执行Ctrl+C的后台消息交互时序图](https://processon.com/view/572c22dae4b0739b929ba590)
