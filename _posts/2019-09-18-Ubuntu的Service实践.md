---
layout: post
categories: 系统管理
---

# 系统启动过程说明
简要的说，linux的inittab过程可以归结为三大步骤

- 1.系统初始化：rc.sysinit
- 2.服务启动：/etc/rc{RUNLEVEL}.d/下的服务 (大多数软连接到/etc/init.d目录的服务配置)
  - 其中的RUNLEVEL为系统的运行级别, 一般的linux分8个级别: 0-6和一个'S'级别.
- 3.本地初始化：/etc/rc.local（用户自启动的启动任务）

## RUNLEVEL说明
在/etc/rc{RUNLEVEL}.d/中，RUNLEVEL为系统的运行级别, 一般的linux分8个级别: 0-6和一个'S'级别.

- 0代表关机(halt);
- 6代表重启(restart);
- 1级别是单用户模式(single),
- 2-5各有不同. 但是在userlinux(包括ubuntu)中2-5级别是毫无差别的.
- 'S'级别是一个比较特殊的级别, 他应该是先于其他级别运行的级别(这一点有待考证).

这里说明一下, 0-6级别的运行是互斥的, 而不是叠加运行, 也就是说如果进入(move into)4级别, 不是指0-3都要运行, 而只是完成4级别里所规定的服务.

如果要查看系统当前的运行级别可以使用命令:
```
runlevel
```

# 创建一个service的过程

将shadowsocks加入系统服务
输入编辑文件命令`vi /etc/systemd/system/shadowsocks.service`并回车
```conf
[Unit]
Description=Shadowsocks
[Service]
TimeoutStartSec=0
ExecStart=/usr/bin/ssserver -c /etc/shadowsocks.json
[Install]
WantedBy=multi-user.target
```


# 参考
- [设置服务开机自启rc.local init.d](https://blog.csdn.net/songw2018/article/details/82829807)
- [/etc/rc.d/init.d 详解](https://blog.csdn.net/SONGW2018/article/details/81950146)
- [ubuntu 16.04 service 基础要点](https://my.oschina.net/janpoem/blog/802708)：如何通过systemd增加一个service
- [Ubuntu Service系统服务说明与使用方法](http://www.mikewootc.com/wiki/linux/usage/ubuntu_service_usage.html)：不使用systemd，直接在/etc/init.d目录增加服务脚本