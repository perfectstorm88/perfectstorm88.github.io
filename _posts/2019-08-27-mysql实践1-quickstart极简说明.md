---
layout: post
categories: mysql 数据库
---

mysql使用的极简说明，包含了安装、建库、建表、建数据、备份、恢复过程。
本次实验的mysql版本为5.7。

# 安装和配置(Ubuntu1.6)
直接使用apt安装就非常顺利，看来mysql的用户广，安装文件和手册基本扎实，[官方安装步骤](https://dev.mysql.com/doc/mysql-apt-repo-quick-guide/en/)看起来有些复杂，简单入门可以参考别人整理的文章[Ubuntu 安装 MySQL](https://www.jianshu.com/p/98332796985b) ，我执行下来顺利无卡壳

关键点：
## 让MySQL服务器可以被远程访问**

1.修改 Mysql 配置文件，去掉绑定的 ip
```bash
sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf

bind-address = 0.0.0.0

// 修改后进行重启
sudo /etc/init.d/mysql restart
```

2.进入 mysql 界面，进行登录授权
```sql
// 登录
mysql -uroot -p
// 登陆后进行授权，注意这里的 password 是数据库密码
// 注意注意！！！若通过mysqladmin -uroot -p123456 password 123  修改密码，并会不影响到这个地方密码
grant all privileges on *.* to 'root'@'%' identified by 'password';
flush privileges; 
```
3.有时配置都改好了，侦听端口也ok，但是仍然无法连接,

错误1：ERROR 2003 (HY000): Can't connect to MySQL server on 'xxxxx'

>应该是防火墙的问题，如果是云主机，重点检查下出入网规则

错误2：ERROR 1045 (28000): Access denied for user 'root'@'101.231.137.69' (using password: YES)

>我确认了好几次，本机可以登录，但是远程同样密码登录不了。后来发现grant all privileges语句中密码和 mysqladmin中的密码是不一样的

若通过mysqladmin修改密码，并会不影响到grant all privileges的密码
```
grant all privileges on *.* to 'root'@'%' identified by 'password';
```

```
mysqladmin -uroot -p123456 password 123
```




# Mac环境安装
直接官网下载版本安装即可：https://dev.mysql.com/downloads/mysql/5.7.html#downloads

注意注意：安装完后，还要修改环境变量,否则会找不到mysqldump命令
```
echo 'export PATH=/usr/local/mysql/bin:$PATH' >> ~/.bash_profile
source  ~/.bash_profile
```
# 创建数据库
```
mysqladmin -uroot -p123456 create db1
```
或者先执行`mysql -uroot -p` 登录后
```
show databases;
CREATE DATABASE db1;
use db1;  #选择一个数据库
```
# 创建表、插入数据

mysql -uroot -p123456 db1
```sql
CREATE TABLE IF NOT EXISTS test(
    a VARCHAR(100),
    b VARCHAR(100)
)ENGINE=InnoDB DEFAULT CHARSET=utf8;

insert into test values('aaaaaaaaaaaa','bbbbbbbbbbbb');
select * from test;
```

# 通过存储过程批量创建数据(为后面备份用)

```sql
CREATE TABLE IF NOT EXISTS test10w(
    a int,
    b VARCHAR(100)
);
DROP PROCEDURE IF EXISTS proc_initData;   --如果存在此存储过程则删掉
DELIMITER $
CREATE PROCEDURE proc_initData()
BEGIN
    DECLARE i INT DEFAULT 1;
    WHILE i<=100000 DO
        INSERT INTO test10w VALUES(i,'bbbbbbbbbbbbbbbbbbbbbb');
        SET i = i+1;
    END WHILE;
END $
CALL proc_initData();
```
# 备份数据
```
date && mysqldump -uroot --p123456 db1 > db1.sql && date
```
[mysql mysqldump只导出表结构或只导出数据的实现方法](https://www.jianshu.com/p/18c179e86cf5)
参数：
- --opt 导出表结构
- -t 导出数据
# 恢复数据
到db2,先创建db2
```
mysqladmin -uroot -p123456 create db2 
```
向db2中插入数据
```
mysql -uroot -p123456 db2 < db1.sql
```

# 阅读
* [Ubuntu 安装 MySQL](https://www.jianshu.com/p/98332796985b):执行下来顺利无卡壳
* [官方文档-Creating and Using a Database](https://dev.mysql.com/doc/refman/8.0/en/database-use.html)
* [备份与还原mysql 数据库的常用命令](https://www.cnblogs.com/nancyzhu/p/8511389.html):比较简单的备份和恢复，完整的要看官网
* [Mysql 循环插入10000条数据](https://blog.csdn.net/CSDN2497242041/article/details/79256063): 管数据用
* [mysql如何修改root用户的密码](https://www.cnblogs.com/qianzf/p/7089197.html)