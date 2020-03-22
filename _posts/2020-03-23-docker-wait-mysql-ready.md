---
layout: post
categories: docker  mysql
---

用docker-compose集成mysql时，mysql容器启动，但是侦听端口还没有启动，此时应用（比如java）启动，会导致启动失败。
在这种场景下，使用docker-compose的依赖是无法解决的，可以采用wait工具
# 解决方法：

## 步骤1：
在dockerfile文件中，增加wait命令，如下
```dockerfile
ADD https://github.com/ufoscout/docker-compose-wait/releases/download/2.2.1/wait /wait
RUN chmod +x /wait
CMD /wait && java -jar rest-boot.jar
```

## 步骤2：
在docker-compose中，增加需要等待的端口
```yml
mysql:
  image: mysql:5.7.29
  ports:
    - "3366:3306"
    - "3306:3306"
  command: --default-authentication-plugin=mysql_native_password
  restart: always
  environment:
    MYSQL_ROOT_PASSWORD: Aa123456
    MYSQL_DATABASE: fysczndz
  security_opt:
    - seccomp:unconfined
  volumes:
    - ./mysql/mydb_log.sql:/docker-entrypoint-initdb.d/mydb_log.sql
    - ./mysql/setup.sql:/docker-entrypoint-initdb.d/setup.sql
    - db_data:/var/lib/mysql
  healthcheck:
    test: mysql -h 127.0.0.1 -u root --password=$$MYSQL_ROOT_PASSWORD -e 'select * from sys_log' fysczndz
    interval: 2s
    timeout: 10s
    retries: 10
java:
  image: registry.cn-hangzhou.aliyuncs.com/wonders_ai/ai:new_pneumonia_java
  ports:
    - "7020:7001"
  environment:
    #spring.datasource.druid.driver-class-name: com.mysql.cj.jdbc.Driver
    WAIT_HOSTS: mysql:3306
```

# 参考

- [docker-compose-wait](https://github.com/ufoscout/docker-compose-wait)