

# 介绍

phpMyAdmin是一个非常受欢迎的基于web的MySQL数据库管理工具。它能够创建和删除数据库，创建/删除/修改表格,删除/编辑/新增字段，执行SQL脚本等。

在我们的工作环境中，通过docker-compose一键完成整个服务集群的部署，包含java、nginx、mysql等。基于docker部署的phpMyAdmin可以配合一同部署的mysql，提供运维人员一种客户端访问方式，便于运维人员在快速部署后立即进入工作状态。

# 启动

## 常用环境变量说明

phpmyadmin是通过/etc/phpmyadmin/config.user.inc.php这个文件修改配置信息的，作为docker部署方式，支持环境变量修改配置，参考： https://docs.phpmyadmin.net/en/latest/setup.html#installing-using-docker

- PMA_ARBITRARY： 允许通过输入一个myql数据库的主机名进行登录
- PMA_HOST: myql数据库的主机名或者IP地址
- PMA_USER: myql数据库的用户名
- PMA_PASSWORD:  myql数据库的密码，在安全的内网环境下可以用，否则直接打开数据库太不安全了。

## docker-compose脚本
```yml
version: "3.7"
services:
  mysql:
    image: mysql:5.7.29
    ports:
      - "33038:3306"
    command: --default-authentication-plugin=mysql_native_password
      --lower_case_table_names=1 --event_scheduler=1
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: "!Aa123456"
      TZ: Asia/Shanghai
  phpmyadmin:
    image: phpmyadmin
    restart: always
    ports:
      - 18038:80
    environment:
      PMA_HOST: mysql
      #PMA_USER: 'root'
      #PMA_PASSWORD: "your mysql password"
```

# 登录界面

![登录界面](https://img0.baidu.com/it/u=1643804277,3896168869&fm=26&fmt=auto&gp=0.jpg)

# 如何通过nginx进行路由代理

比如:
```nginx

location /myadmin/{
  proxy_pass http://ip:port/
  
}
```
第一次登陆后成功后，会报404,不用管，重新打开这个页面即可 https://ip:port/myadmin/

# 参考：
- [官方Installing using Docker](https://docs.phpmyadmin.net/en/latest/setup.html#installing-using-docker)
- [phpMyAdmin使用教程](https://blog.csdn.net/u012767761/article/details/78238487)