---
layout: post
categories: 网络 系统管理
---

>之前配置过vpn代理，发现被墙的网站可以通过vpn顺畅访问。如果是国内网站，再从VPN绕一圈就太慢了。
>公司的同事装了myetunnel ssh auto proxy,可以自动区分是否被墙的网站，自动翻墙。
>不过他是Windows版，于是我也折腾了一会儿，终于把Mac版自动区分搞好了


# 1. 原理
翻墙原理：
**浏览器插件方式**
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTUwNDI5MjI1MjI5MTIz)

**VPN方式**
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTUwNDI5MjI1MzEyNzU2)
`（VPN也可以路由表的方式区分国家网络）http://blog.jaekj.com/archives/1603.html`
远程代理的配置，参见：http://blog.csdn.net/lichangzhen2008/article/details/45366475

# 2. ssh代理
## 2.1. ssh远程服务器
申请一个国外的VPS就可以了.
我是在亚马逊上申请VPS，注册用户可以免费使用一年t2.micro.

## 2.2. 本地ssh代理

关于本地ssh代理，这篇文章[各平台 SSH 免费客户端 SSH代理客户端](http://blog.csdn.net/bbplayers/article/details/6853252 )，列举了很多方式

对于Linux，建议使用终端命令:**ssh -qTfnN -D 7070 remotehost**
All the added options are for a ssh session that’s used for tunneling.
-q :- be very quite, we are acting only as a tunnel.
-T :- Do not allocate a pseudo tty, we are only acting a tunnel.
-f :- move the ssh process to background, as we don’t want to interact with this ssh session directly.
-N :- Do not execute remote command.
-n :- redirect standard input to /dev/null.
In addition on a slow line you can gain performance by enabling compression with the -C option.

我本机的执行命令如下(**麻烦的是，每次用户都要重启一次**）
```bash
ssh -qTfnN -D 2022 -i ~/remote_login/pem/amazon_free_li.pem root@52.68.78.183
```

52.68.78.183是亚马逊VPS的主机ip
2022在ssh本地代理的侦听端口，后面的SwitchyOmega在配置时要用到这个端口
 
# 3. SwitchyOmega安装部署

##3.1 阅读官方说明文档
[SwitchyOmega 新功能](https://github.com/FelisCatus/SwitchyOmega/wiki/SwitchyOmega-%E6%96%B0%E5%8A%9F%E8%83%BD)

## 3.2 插件安装
中国大陆地区用户建议[参考图文设置](https://github.com/FelisCatus/SwitchyOmega/wiki/GFWList)教程进行配置。

## 3.3 导入配置
在导入导出页面点击“从备份文件恢复”
![](https://github.com/FelisCatus/SwitchyOmega/wiki/images/t1/step2.png)
导入下面内容(拷贝粘帖到文件再导入)
```json
{
  "+GFWed": {
    "bypassList": [
      {
        "conditionType": "BypassCondition",
        "pattern": "<local>"
      }
    ],
    "color": "#dd6633",
    "fallbackProxy": {
      "host": "127.0.0.1",
      "port": 2022,
      "scheme": "socks5"
    },
    "name": "GFWed",
    "profileType": "FixedProfile",
    "revision": "14e90e8be8e"
  },
  "+__ruleListOf_自动切换": {
    "color": "#99dd99",
    "defaultProfileName": "direct",
    "format": "AutoProxy",
    "matchProfileName": "GFWed",
    "name": "__ruleListOf_自动切换",
    "profileType": "RuleListProfile",
    "revision": "14e90e53ff6",
    "ruleList": "",
    "sourceUrl": "https://autoproxy-gfwlist.googlecode.com/svn/trunk/gfwlist.txt"

  },
  "+自动切换": {
    "color": "#00f200",
    "defaultProfileName": "__ruleListOf_自动切换",
    "name": "自动切换",
    "profileType": "SwitchProfile",
    "revision": "14e90e7a963",
    "rules": [
      {
        "condition": {
          "conditionType": "HostWildcardCondition",
          "pattern": "autoproxy-gfwlist.googlecode.com"
        },
        "profileName": "GFWed"
      }
    ]
  },
  "-confirmDeletion": true,
  "-downloadInterval": 1440,
  "-enableQuickSwitch": false,
  "-monitorWebRequests": true,
  "-quickSwitchProfiles": [],
  "-refreshOnProfileChange": true,
  "-revertProxyChanges": true,
  "-showInspectMenu": true,
  "-startupProfileName": "自动切换",
  "schemaVersion": 2
}
```

关于上述配置的几点说明：

- 初始情景模式， "-startupProfileName": "自动切换"
- 快速切换关闭，"-enableQuickSwitch": false
- 在图标上显示当前页面由于网络原因而未加载的资源数量，"-monitorWebRequests": true
- 情景模式GFWed
    - 类型：代理服务器 FixedProfile
    - 配置代理"fallbackProxy": {"host": "127.0.0.1","port": 2022,"scheme": "socks5"}
    - 不代理名单bypassList，无
- 情景模式自动切换
    - 类型：自动切换模式 SwitchProfile
    - 切换规则，"autoproxy-gfwlist.googlecode.com"切换到情景模式GFWed
    - 可以添加规则列表，以便引用他人在线发布的一组规则
        - AutoProxy
        - 在线规则地址"sourceUrl": "https://autoproxy-gfwlist.googlecode.com/svn/trunk/gfwlist.txt"

### 3.4 更新规则列表
设置代理服务器，就是本地的ssh代理127.0.0.1:2022,这个本地的ssh代理与远端的VPS建立一个加密隧道。
![](https://github.com/FelisCatus/SwitchyOmega/wiki/images/t1/step3.png)
自动更新规则列表，如果自动更新失败，检查下本地的ssh代理和远端的VPS是否ok
![](https://github.com/FelisCatus/SwitchyOmega/wiki/images/t1/step5.png)

# 4. 参考资料
[各平台 SSH 免费客户端 SSH代理客户端](http://blog.csdn.net/bbplayers/article/details/6853252)

[sshtunel on mac](http://en.katzueno.com/2012/08/12/the-best-and-simple-way-to-establish-ssh-tunnel-on-mac-os-x/#.VT9HTJOUcik)

>You can use Terminal to port forward to your localhost, and access. But it’s troublesome to command each time on the Terminal. I’ve tried Fugu, SSHTunnel, and SSH Tunnel Manager. But they all DID NOT **support RSA public key authentification**. Meekat is great Free app, but it’s no longer maintained… It may not work in the future.
After a few days of the research, I’ve finally able to find the  best and simple SSH tunnel app called Coccinellida!
COCCINELLIDA – SIMPLE SSH TUNNEL MANAGER FOR MAC OS X

[什么是 Port Forwarding及SSH Tunnel](http://sky-bruce.blogspot.jp/2009/04/port-forwarding.html)


[另一种方法：直接写入，缺点是不能直接更新](http://www.cnblogs.com/anee/p/4146175.html)


[vps](https://zh.wikipedia.org/wiki/%E8%99%9A%E6%8B%9F%E4%B8%93%E7%94%A8%E6%9C%8D%E5%8A%A1%E5%99%A8) 虚拟专用服务器（英语：Virtual private server，缩写为 VPS），是指通过虚拟化技术在独立服务器中运行的专用服务器。每个使用VPS技术的虚拟独立服务器拥有各自独立的公网IP地址、操作系统、硬盘空间、内存空间、CPU资源等，还可以进行安装程序、重启服务器等操作，与运行一台独立服务器完全相同。