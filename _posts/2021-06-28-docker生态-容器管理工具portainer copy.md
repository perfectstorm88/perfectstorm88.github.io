

# 介绍
Portainer是Docker的图形化管理工具，提供状态显示面板、应用模板快速部署、容器镜像网络数据卷的基本操作（包括上传下载镜像，创建容器等操作）、事件日志显示、容器控制台操作、Swarm集群和服务等集中管理和操作、登录用户管理和控制等功能。功能十分全面，基本能满足中小型单位对容器管理的全部需求。

通过portainer服务端，连接agent，达到管理agent节点上docker容器的目的，如下所示：
![Portainer server and agent](https://camo.githubusercontent.com/84ced73c340e19c56df93b5f70cbc090581d46de5a146d573ce919eef9ad088e/68747470733a2f2f636f64657472656575736572636f6e74656e742e73332e616d617a6f6e6177732e636f6d2f75706c6f6164732f38613865656431352d393361362d343762622d383038312d6638633962323638643533312f6469616772616d312e706e67)

portainer服务端可以支持多个不同的端点(endpoints)类型,如下：


下面一个edge agent的架构图
![overview of the edge agent's architecture](https://img2018.cnblogs.com/blog/1900199/201912/1900199-20191219213901571-1420625500.jpg)

# 安装部署

## 汉化说明

从 [github的portainer汉化项目](https://github.com/eysp/public) 下载源文件：
解压后对应，映射到容器的/public目录，否则需要把”./portainer-ce/public:/public“注释掉

## 初始密码说明

官方步骤里，这一步不是必备操作，但安全起见还是启动时就把密码设置好。
```bash
htpasswd -nb -B admin "your-password" | cut -d ":" -f 2
# n 不更新文件，输出到stdout
# b 通过命令行获取密码，而不用提示方式
# B 使用bcrypt加密方式，默认是md5
```

若配置到docker-compose.yaml，因为这个密码中含有有$需要进行转义，替换为两个"$$",否则会报错：
```log
ERROR: Invalid interpolation format for "command" option in service "portainerS": 
```


## 最终的docker-compose
参考[官方docker安装方式](https://documentation.portainer.io/v2.0/deploy/ceinstallswarm/)

```yaml
version: "3.7"
services:
  portainerS:
    image: portainer/portainer-ce
    command: [ "--admin-password=$$2y$$05$$7ewuPZtowbKRTihtCnLIw.VBF5y8rujsS8Y4mFnBy1s.SOozrPDsG" ]
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - ./portainer-ce/public:/public
      - portainer_data:/data
    ports:
      - "28038:8000"
      - "29038:9000"
  portainerA:
    image: portainer/agent
    ports:
      - "29039:9001"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /home/data/docker/volumes:/var/lib/docker/volumes
volumes:
  portainer_data:
```
上图portainerA中的/var/lib/docker对应”/etc/docker/daemon.json“中的data-root值，如果自己安装时调整过，记得进行修改。
例如我的/etc/docker/daemon.json对应内容如下，则需要替换过来

```json
{
 "data-root":"/home/data/docker"
}
```

通过`docker-compose up -d `启动后，就打开 http://ip:port/进行访问了


# 通过nginx代理时的注意要点

```nginx
location /portainer/ {

    #升级http1.1到 websocket协议
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";

    # 关闭访问日志，否则查看nginx日志时，会有一大堆portainer的访问日志
    access_log  off;  

    proxy_pass http://portainerS:9000/; #后端服务路径
}
```

# 参考：

- [官方docker swarm安装方式](https://documentation.portainer.io/v2.0/deploy/ceinstallswarm/)
- [官方docker安装方式](https://documentation.portainer.io/v2.0/deploy/ceinstallswarm/)
- [BCrypt 密码加密和解密](https://www.jianshu.com/p/fc910a1f7c8d/):Bcrypt就是一款加密工具，可以比较方便地实现数据的加密工作。你也可以简单理解为它内部自己实现了随机加盐处理,Bcrypt生成的密文是60位的,而MD5的是32位的。
- [Centos安装htpasswd_Nginx中使用htpasswd](https://www.itbiancheng.com/linux/4807.html)
- [nginx设置账号密码--htpasswd的使用](https://www.cnblogs.com/zhaoyingjie/p/12442861.html): nginx使用默认的MD5密码
- [docker compose command: Invalid interpolation format for “command” option in service](https://stackoverflow.com/questions/59158629/docker-compose-command-invalid-interpolation-format-for-command-option-in-ser/)
- [使用Portainer集中管理多地域内网运行的Docker实例](https://www.cnblogs.com/wwek/p/12070379.html)
- [WebSocket proxying](http://nginx.org/en/docs/http/websocket.html)：nginx官方案例
