---
layout: post
categories: Linux 系统管理
---
Linux磁盘文件共享工具，samba介绍

# 安装
apt install -y samba
```
service smbd restart # 重启 这个service名称是smbd而不是samba
```
# 配置
在/etc/samba/smb.conf最后添加
```
[share]
      path = /home/phinecos/share
      available = yes
      browsealbe = yes
      public = yes
      writable = yes
```

创建samba账号
```
  sudo touch /etc/samba/smbpasswd
  sudo smbpasswd -a phinecos
```
然后会要求你输入samba帐户的密码
如我输入的是：lingxi-gpu  密码为lingxi
# 参考
[Linux中Samba详细安装](http://www.cnblogs.com/whiteyun/archive/2011/05/27/2059670.html)

[samba应用常见问题 和 修改后的smb.conf文件](http://blog.csdn.net/ruanjianruanjianruan/article/details/46954681)
