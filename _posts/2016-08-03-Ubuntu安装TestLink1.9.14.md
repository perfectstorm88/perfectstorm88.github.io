---
layout: post
categories: Linux 系统管理
---

本文是当时团队想引入一个测试跟踪系统，于是就比较了几个测试管理工具，最后选型了TestLink，下面是TestLink安装和使用过程

# TestLink介绍
Testlink是一个开源的测试管理工具，主要用于管理测试用例，从测试需求、测试计划、测试用例管理和用例执行，到最后的结果分析，一套完整的测试流程控制，帮助测试人员有效的控制测试过程。

下面说说Testlink的主要功能如下：   
1、 测试需求的管理   
2、 测试计划的管理   
3、 测试用例的管理   
4、 测试用例的执行   
5、 测试结果的分析 (包括测试结果的图表分析)   
6、 基于角色的用户管理

# TestLink安装

## 确认安装环境和版本

查看Linux版本和内核

```
1.uname -r #内核
2.cat /etc/issue # 查看内核版本
3.sudo lsb_release -a #查看版本,有些主机没有安装lsb_release命令，c适用于所有的linux，Centos、UbuntuRedhat、SuSE、Debian等发行版。

```

主机环境是Ubuntu 12.04.5 LTS   
软件版本：testlink-1.9.14.tar.gz

## 下载并安装必需软件：

```
1.sudo apt-add-repository ppa:ondrej/apache2
2.sudo LC_ALL=C.UTF-8 add-apt-repository ppa:ondrej/php
3.sudo apt-get install apache2   #安装apache2.4
4.sudo apt-get install mysql-server #安装mysql 5.5
5.sudo apt-get install php libapache2-mod-php php-mcrypt php-mysql #安装php7.0

```

安装完毕后，会启动msyqld服务和apache server

### 检查mysql是否正常

mysql -u root -p

```
1.show databases; --显示数据库列表
2.create database test ; -- 创建数据库
3.drop database test; -- 删除数据库

```

### 检查apache server是否正常

通过 [http://ip:port](http://ip:port) 确认访问是否正常

### 检查php版本

php -v

```
1.PHP 7.0.10-2+deb.sury.org~precise+1 (cli) ( NTS )
2.Copyright (c) 1997-2016 The PHP Group
3.Zend Engine v3.0.0, Copyright (c) 1998-2016 Zend Technologies
1.    with Zend OPcache v7.0.10-2+deb.sury.org~precise+1, Copyright (c) 1999-2016, by Zend Technologies

```

## 配置apache2

sudo vi /etc/apache2/httpd.conf   
添加以下内容：

```
1.AddType application/x-httpd-php .php .htm .html
2.AddDefaultCharset UTF-8
3.ServerName 127.0.0.1

```

修改后重启动apache2

```
1.sudo /etc/init.d/apache2 restart

```

### 对于80端口被占用情况(如果是默认80端口，忽略这一节)

sudo vi /etc/apache2/ports.conf   
把原来的80改为8080

```
1.NameVirtualHost *:8080
2.Listen 8080 

```

sudo vi /etc/apache2/sites-enabled/000-default   
把原来的80改为8080

```
1.<VirtualHost *:8080>

```

## 安装testlink版本包

从官网上下载最新的版本，并拷贝到apache默认的/var/www/ 目录   
[http://www.testlink.org.cn/download](http://www.testlink.org.cn/download)

```
1.tar zxvf testlink-1.9.14.tar.gz
2.sudo mv testlink-1.9.14 /var/www/testlink
3.cd /var/www/

```

## 执行testlink安装向导

[http://localhost:port/testlink/install/](http://localhost:port/testlink/install/)   
前三步直接点击。第四步，输入数据库名称、admin user、testlink user（后两个可以共用一个user）

   
第5步：会提示

-   连接数据库成功
-   创建数据库testlink
-   创建数据库testlink的用户
-   导入初始化脚本
-   **写配置文件失败**,如下图所示，把文件拷贝到/var/www/testlink/config_db.inc.php即可

![](http://static.oschina.net/uploads/space/2016/0831/113619_MAyp_2336177.png)

## 修改testlink配置文件

sudo vi config.inc.php

修改“user\_self\_signup”(是否允许用户自己注册)参数值为“FALSE”   
修改“config\_check\_warning_mode”参数值为“SILENT”   
修改为：tlCfg->default\_language = ‘zh\_CN’;(汉化)

## 成功

然后就可以访问testlink   
[http://localhost/testlink](http://localhost/testlink)   
汉化后，登录界面如下：

![](http://static.oschina.net/uploads/space/2016/0831/113619_yukL_2336177.png)

使用默认帐号”admin”登录，密码为”admin”，登录后修改密码。   
如果登录后，页面还是英文，最后帐号设置中，语言–选择chinese

# 踩过的坑

## apache默认80端口被占

安装apache2过程出现下面异常

```
1.Starting web server apache2                                                                                                                                                       **apache2: Could not reliably determine the server's fully qualified domain name, using 10.251.233.162 for ServerName**
2.(98)Address already in use: make_sock: could not bind to address 0.0.0.0:80
3.no listening sockets available, shutting down
4.Unable to open logs
5.Action 'start' failed.

```

1、第一个warn是因为apache的域名没有配置，若通过ip访问，可以忽略   
2、另外，如果是租用的云主机，会发现10.251.233.162这个ip比较奇怪，这是云主机的内网IP,取的本机的以太网卡ech0的地址，非我们用的映射后的公网IP。可通过下面命令确认

```
1.ifconfig eth0 | grep inet | awk '{ print $2 }'

```

3、第二个“Address already in use”，这是因为80端口被占，可能本机之前已经部署过其它服务，比如nginx；需要对apache server的默认端口进行修改

## PHP版本问题，要求5.4以上

之前参考的是\[官网的一篇文章\]\[1\]

   
**You are running on PHP 5.3.10-1ubuntu3.24, and TestLink requires PHP 5.4.0 or greater. This is fatal problem. You must upgrade it.**

需要把Ondřej Surý PPA加入到apt sources.list中

```
1.sudo apt-add-repository ppa:ondrej/php

```

## PHP5.4又依赖于apache2.4（当前是2.2）

需要先卸载apache2.2，再安装

```
1.sudo apt-get --purge remove apache2 ##卸载包和配置文件
2.sudo apt-get autoremove  ##删除自动安装的已经不需要的依赖包

```

# 参考文档

1.  [TestLink部署与介绍(Ubuntu)](http://www.testlink.org.cn/363.html):版本有些老，我在实践过程中 更新apache2.2 到2.4，和php5.3 到5.4以上踩了好几个坑
2.  [CentOS6.5装testlink1.9.14](http://www.centoscn.com/image-text/install/2015/1226/6580.html):本来想参考这个文档，暂时手里有一台ubuntu空闲资源，又不能重装系统，只能装在ubuntu上
3.  [How To Install Linux, Apache, MySQL, PHP (LAMP) stack on Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-install-linux-apache-mysql-php-lamp-stack-on-ubuntu):这个文章虽然也是Ubuntu 12.04 环境，但是php是5.3版本，没怎么参考
4.  [How To Install Linux, Apache, MySQL, PHP (LAMP) stack on Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-install-linux-apache-mysql-php-lamp-stack-on-ubuntu-16-04):对Ubuntu 12.04，安装最新版php和apache的命令是一样的
5.  [How do I upgrade PHP version to the latest stable released version?](http://askubuntu.com/questions/565784/how-do-i-upgrade-php-version-to-the-latest-stable-released-version):php从5.3升级到5.4及以上
6.  [how reinstall apache2](http://askubuntu.com/questions/111770/how-reinstall-apache2):卸载重装apache，sudo apt-get –purge remove apache2