---
layout: post
categories: JAVA msyql 数据库
---

# 问题描述
用java程序连接mysql数据库，报下面错误
```
com.alibaba.druid.pool.DruidDataSource 872 init - init datasource error, url: jdbc:mysql://localhost:3306/fns?serverTimezone=UTC&useSSL=false java.sql.SQLNonTransientConnectionException: Public Key Retrieval is not allowed
	at com.mysql.cj.jdbc.exceptions.SQLError.createSQLException(SQLError.java:110) ~[mysql-connector-java-8.0.12.jar!/:8.0.12]
	at com.mysql.cj.jdbc.exceptions.SQLError.createSQLException(SQLError.java:97) ~[mysql-connector-java-8.0.12.jar!/:8.0.12
```


# 解决方法1
解决此异常问题需要将allowPublicKeyRetrieval=true和useSSL=false。

```java
jdbc:mysql://localhost:3306/db-name?useUnicode=true&characterEncoding=UTF-8&allowPublicKeyRetrieval=true&useSSL=false
```
# 解决方法2(推荐)
Start from MySQL 8, the authentication plugin is changed to "caching_sha2_password".

We changed the authentication plugin to "mysql_native_password", script as following:
```sql
mysql -u root -p

mysql> use mysql

mysql> select host, user from user;
+-----------+------------------+
| host      | user             |
+-----------+------------------+
| %         | root             |
| localhost | mysql.infoschema |
| localhost | mysql.session    |
| localhost | mysql.sys        |
| localhost | root             |
+-----------+------------------+
5 rows in set (0.00 sec)

mysql> ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY 'Your New Password';
Query OK, 0 rows affected (0.08 sec)

mysql> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.01 sec)
```


# 参考：
- [MySQL 8 - Public Key Retrieval is not allowed](https://qiita.com/shaching/items/5fe3d5df691b4ec53084)
- [Java连接Mysql数据库异常：Public Key Retrieval is not allowed](https://www.cjavapy.com/article/399/)