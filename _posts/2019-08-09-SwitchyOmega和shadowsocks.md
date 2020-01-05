---
layout: post
categories: 网络 系统管理
---

# SwitchyOmega和shadowsocks的工作原理
- 1.浏览器插件SwitchyOmega区分是否需要通过代理访问外网，
- 2.若需要外网，然后通过socks5协议发到shadowsocks的client，
- 3.shadowsocks的client和server之间通过加密隧道，有多种加密方式，并非受限于ssh的RSA加密技术
- 4.shadowsocks的server解码为socks5协议包后，发送到对应的网站服务器

![SwitchyOmega和shadowsocks工作原理](http://img.lichangzhen.top/SwitchyOmega和shadowsocks工作原理.jpg)

# 安装Shadowsocks服务器(在外网))

## 在CENTOS 7上搭建Shadowsocks

### 安装Shadowsocks

```bash
sudo yum -y install epel-release
sudo yum -y install python-pip
sudo pip install shadowsocks
```
### 配置shadowsocks
输入编辑文件命令`sudo vi /etc/shadowsocks.json`

```json
{
    "server":"0.0.0.0",
    "server_port":50013,
    "local_port":1080,
    "password":"1234567890",
    "timeout":600,
    "method":"aes-256-cfb"
}
```
### 将shadowsocks加入系统服务
输入编辑文件命令`sudo vi /etc/systemd/system/shadowsocks.service`并回车

```conf
[Unit]
Description=Shadowsocks
[Service]
TimeoutStartSec=0
ExecStart=/usr/bin/ssserver -c /etc/shadowsocks.json
[Install]
WantedBy=multi-user.target
```

### 启动shadowsocks服务并设置开机自启
```bash
# 设置开机自启命令
systemctl enable shadowsocks
# 启动命令
systemctl start shadowsocks
#查看状态命令
systemctl status shadowsocks
```
## 在Ubuntu搭建Shadowsocks

### 安装Shadowsocks
```
sudo apt update
sudo apt install python3-pip
sudo pip3 install shadowsocks
```
'配置shadowsocks'和'启动shadowsocks服务并设置开机自启'与centos 上相同

## 如果海外服务器ping不通，也ssh不上，如何做呢？

可以使用 http://ping.chinaz.com 这个工具检测ip地址：

- 如何海外能ping通，但是国内不通，则说明被强
- 如果都ping不通，则说明安全组和服务本身的问题


**租用亚马逊服务器有个好处，停止-启动一下，IP地址会变（然后再测试下新的ip地址是否被封）**

# 在本机或者局域网安装sslocal

同样安装，执行命令如下
```bash
sslocal -s 134.209.99.5 -p 50013 -k 1234567890 -l 18315 -t 1000 -m aes-256-cfb
```

# 安装chrome浏览器插件SwitchyOmega
网上有很多资料，不再啰嗦

插件下载地址：

- [SwitchyOmega](http://img.lichangzhen.top/software/SwitchyOmega_Chromium.crx)
- [OmegaOptions.bak](http://img.lichangzhen.top/software/OmegaOptions.bak)


# 另外一种方式是ssh tunnel方式
与shadowsocks原理图相似，但是加密隧道的协议不一样，一种是ssh，另一个可以采用多种加密方式，所以GFW更难分析和拦阻

![SwitchyOmega和ssh tunnel工作原理](http://img.lichangzhen.top/SwitchyOmega和ssh tunnel工作原理.jpg)

# shadowsocks的客户端下载

https://shadowsocks.org/en/download/clients.html

已经验证：

- andiod
- maxos https://shadowsocks.org/en/download/clients.html
  - ShadowsocksX-NG 

# 参考：

- [在CENTOS 7上搭建Shadowsocks图文教程](https://www.4spaces.org/install-shadowsocks-on-centos-7/)
- [你也能写个 Shadowsocks](https://segmentfault.com/a/1190000011862912)
- [科学上网原理](https://github.com/Pines-Cheng/blog/issues/28):非常棒，网上搜了半天，到这儿来，全部原理搞清楚了
  - 讲了GFW的原理，
  - SSH Tunnel原理，ssh的特征很明显，GFW会检查ssl和ssh，统计包长度和方向，判断是否http，进行reset
  - VPN，比shadowssocks更底层，通过操作系统接口虚拟出一张网卡。vpn是通过编写一套网卡驱动并注册到操作系统实现的虚拟网卡，这样数据只要经过网卡收发就可以进行拦截处理
  - shadowsocks 是将原来 ssh 创建的 Socks5 协议拆开成 server 端和 client 端，ss-local 和 ss-server 两端通过多种可选的加密方法进行通讯
  - PAC模式，代理自动配置，就是SwitchyOmega的功能，Shadowsocks已经支持了，不需要chrome再安装这个插件
  
- [翻墙基本原理和这堵墙是如何“筑成”的](https://fanqiang.network/10790.html):这篇文章深度更深，和上一篇科学上网原理结合看，更能丰富你的知识面

- [socks5 协议简介](http://zhihan.me/network/2017/09/24/socks5-protocol/),不错的文章，了解socks5协议的原理

- [Mac中的Shadowsocks客户端](https://crifan.github.io/scientific_network_summary/website/server_client_mode/ss_client/client_mac.html):