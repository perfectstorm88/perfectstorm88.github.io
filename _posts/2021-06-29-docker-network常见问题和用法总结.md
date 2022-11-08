

# 网络冲突问题(docker和docker-compose)

启动docker-compose时，网络冲突现象：

dokcer引擎启动时使用了`--bip`参数，但是该参数被docker-compose忽略，在 docker0 之外，有启动了一个桥接接口，名称为br-3d2f2e1ebfc7，导致与本机的网段冲突
```
[root@yuwenzhen build]# ifconfig
br-3d2f2e1ebfc7: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.26.0.1  netmask 255.255.0.0  broadcast 172.26.255.255
        inet6 fe80::42:9fff:fe82:2210  prefixlen 64  scopeid 0x20<link>
        ether 02:42:9f:82:22:10  txqueuelen 0  (Ethernet)
        RX packets 6575  bytes 580741 (567.1 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 5760  bytes 171239954 (163.3 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

docker0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 191.168.1.1  netmask 255.255.255.0  broadcast 191.168.1.255
        inet6 fe80::42:fbff:fe9f:8944  prefixlen 64  scopeid 0x20<link>
        ether 02:42:fb:9f:89:44  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 3  bytes 266 (266.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

ens32: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 173.18.1.55  netmask 255.255.255.0  broadcast 173.18.1.255
        inet6 fe80::c6e8:ee4b:21f0:22fe  prefixlen 64  scopeid 0x20<link>
        ether 00:50:56:90:0b:65  txqueuelen 1000  (Ethernet)
        RX packets 1135954  bytes 110513197 (105.3 MiB)
        RX errors 0  dropped 67  overruns 0  frame 0
        TX packets 45647  bytes 13797639 (13.1 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

解决方法：修改 `/etc/docker/daemon.json`，没有此文件则创建，`default-address-pools`可以同时被docker引擎和docker-compose使用
```json
{  
"default-address-pools" : [     
     {      
      "base" : "172.31.0.0/16",       
      "size" : 24    
     }   
 ]
}
```
修改完后通过`systemctl restart docker` 重启docker引擎

参考：

- [How does compose chooses subnet for default network?](https://github.com/docker/compose/issues/4336)：这上面给了使用说明
- [Docker Networking](https://towardsdatascience.com/docker-networking-919461b7f498):几种网络模式
- [修改Docker默认的网段](https://blog.csdn.net/eagle89/article/details/105440080)：bip只适用于docker engine不适用docker-compose
- [特性还是bug？】Docker-compose 网段冲突，即使修改了docker0网段也不行](www.xiaomaidong.com/?p=1336)


思路扩展：docker-compose的用法相当自定义了网络，这个网段呢跟docker0不一样

![docker-netword user define network](https://miro.medium.com/max/700/1*-O2YqpRXmDBP_PzSNOhyyA.png)


# 网络不能删除问题

docker清除时，执行`docker-compose down -v`时报错如下：

```bash
Removing network ml_platform_deploy_national_default
ERROR: error while removing network: network ml_platform_deploy_national_default id 7c4214d5aae5d1340a7ba1da7fc13f9e9176aaf3e979c056b8318ea420b6cf2e has active endpoints
```

解决方法如下：
step1： 通过inspect命令查看`docker inspect 网络名或者id`，信息如下：

```json
[
    {
        "Name": "ml_platform_deploy_national_default",
        "Id": "7c4214d5aae5d1340a7ba1da7fc13f9e9176aaf3e979c056b8318ea420b6cf2e",
        "Created": "2020-12-21T09:58:06.645385113+08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.23.0.0/16",
                    "Gateway": "172.23.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": true,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "2754b25cbe0cd15579c7f98f31f772b5762c6e2c94bc1abe506abb98f9df4c72": {
                "Name": "ml_platform_deploy_national_mysql_1",
                "EndpointID": "c43660ec44622aa8f51fc933c7e7718690d0c9bc6dbc95195e83eb1f34605962",
                "MacAddress": "02:42:ac:17:00:02",
                "IPv4Address": "172.23.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {
            "com.docker.compose.network": "default",
            "com.docker.compose.project": "ml_platform_deploy_national",
            "com.docker.compose.version": "1.24.1"
        }
    }
]
```

step2： 执行`docker network disconnect [OPTIONS] NETWORK CONTAINER`强制断开网络

如下:
```
docker network disconnect  -f ml_platform_deploy_national_default  ml_platform_deploy_national_mysql_1`
```

- NETWORK 第一步输出json对应的Name 
- CONTAINER 第二步输出json对应的Containers.xxx.Name


# 使用已存在的网络

应用场景，适合跨多个docker服务使用同一个网络

```yml
networks:
  default:
    external:
      name: my-pre-existing-network
```

# 参考

- [docker-compose网络设置之networks](https://blog.csdn.net/Kiloveyousmile/article/details/79830810):

