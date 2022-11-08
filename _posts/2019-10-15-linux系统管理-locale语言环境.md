---
layout: post
categories: Linux 系统管理
---

linux下语言环境的问题，一般不太会碰到，要是真遇到字体问题也是挺烦人的，不解决看到系统乱码和错误提示挺膈应人，解决后就清爽多了。linux的语言环境是通过locale命令展开的，原理、操作命令、问题样例如下

# 概念理解

## 变量说明

- 在Linux中通过locale来设置程序运行的不同语言环境，locale由 ANSI C提供支持
- 在locale环境中，有一组变量，代表国际化环境中的不同设置
  - LC_COLLATE，定义该环境的排序和比较规则
  - LC_CTYPE，用于字符分类和字符串处理，控制所有字符的处理方式，包括字符编码，字符是单字节还是多字节，如何打印等。是最重要的一个环境变量。 LC_MONETARY，货币格式
  - LC_NUMERIC，非货币的数字显示格式
  - LC_TIME，时间和日期格式
  - LC_MESSAGES，提示信息的语言。
  - LANGUAGE参数，它与LC_MESSAGES相似，但如果该参数一旦设置，则LC_MESSAGES参数就会失效。 - - LANGUAGE参数可同时设置多种语言信息，如LANGUAGE="zh_CN.GB18030:zh_CN.GB2312:zh_CN"
  - LANG，LC_*的默认值，是最低级别的设置，如果LC_*没有设置，则使用该值。类似于 LC_ALL
  - **LC_ALL，它是一个宏，如果该值设置了，则该值会覆盖所有LC_*的设置值。注意，LANG的值不受该宏影响**
- locale的命名规则样例 zh_CN.GBK，zh代表中文， CN代表大陆地区，GBK表示字符集
  
## 命令

1. 查看当前系统语言环境

```bash
[root@w117 ~]#locale
LANG=en_US.UTF-8
LC_CTYPE="zh_CN.utf8"
LC_NUMERIC="zh_CN.utf8"
LC_TIME="zh_CN.utf8"
LC_COLLATE="zh_CN.utf8"
LC_MONETARY="zh_CN.utf8"
LC_MESSAGES="zh_CN.utf8"
LC_PAPER="zh_CN.utf8"
LC_NAME="zh_CN.utf8"
LC_ADDRESS="zh_CN.utf8"
LC_TELEPHONE="zh_CN.utf8"
LC_MEASUREMENT="zh_CN.utf8"
LC_IDENTIFICATION="zh_CN.utf8"
LC_ALL=zh_CN.utf8
```

2. 查看系统内安装的locale
  
```bash
[root@w117 ~]# locale -a  |grep zh_CN
zh_CN
zh_CN.gb18030
zh_CN.gb2312
zh_CN.gbk
zh_CN.utf8
```

1. 安装locale(网上还有另一个种方法，编辑/etc/locale.gen，然后执行locale-gen)
```
sudo locale-gen en_US.UTF-8
```

4. 文件/etc/default/locale 保存了locale的默认设置，若没有生效，参考下面问题1


# 常见问题解决

## 问题:ssh登录时报错“manpath: can't set the locale; make sure $LC_* and $LANG are correct”

1. 检查/etc/default/locale文件，内容如下：
```
LC_ALL="en_US.UTF-8"
LANG="zh_CN.UTF-8"
LANGUAGE="zh_CN:zh:en_US:en"
```

2. 执行`locale -a`,发现zh_CN.utf8和en_US.utf8也是可用的

3. 应该是/etc/default/locale没有生效，找到[Why is /etc/default/locale not sourced at login via ssh?](https://askubuntu.com/questions/749440/why-is-etc-default-locale-not-sourced-at-login-via-ssh)，发现“Both /etc/default/locale and /etc/environment are supposed to be parsed by PAM. They are not script files, and hence should not be sourced”

4. 通过修改/etc/ssh/sshd_config的参数`UsePAM yes`,然后`sudo systemctl restart sshd`,再重新登录~~~ok



# 问题：终端时的中文乱码问题


现在是在mac默认环境下的测试
```bash
locale -a  |grep zh_CN
#zh_CN
#zh_CN.gb18030
#zh_CN.gb2312
#zh_CN.gbk
#zh_CN.utf8
export LC_ALL=en_US.utf8
date
#-bash: local: 
export LC_ALL=zh_CN.utf8
date
#-bash: local: 只能在函数中使用
export LC_ALL=zh_CN.gbk
date
#-bash: local: ֻ���ں�����ʹ��
```
说明：通过`locale -a`看到的字符集，并非设置好了就正常，还跟终端支持的编码有关

网上有人测试说，设为zh_CN.gb18030、zh_CN.gbk、zh_CN.gb2312都能正常显示简体中文，但是设为zh_CN.utf-8是乱码，而在我的mac主机，设为zh_CN.gb18030、zh_CN.gbk、zh_CN.gb2312都乱码，zh_CN.utf-8正常。

原因：这种情况一般是终端和服务器的字符集不匹配，MacOSX下默认的是utf8字符集。

### 问题：用xshell查看中文正常，但是vim是乱码，原因可能是xshell工具的编码问题

例如xshell选择编码：[xshell怎么设置编码，xshell中文乱码解决方法](https://jingyan.baidu.com/article/47a29f2495eeb4c014239983.html)

打开xshell，点击上方的编码图标
![xshell修改编码方式](https://exp-picture.cdn.bcebos.com/51cd85cec7f88a777bb37eff6e4a2f27e6eff8c1.jpg?x-bce-process=image%2Fresize%2Cm_lfit%2Cw_500%2Climit_1%2Fformat%2Cf_jpg%2Fquality%2Cq_80)


## 问题：Mac环境下vim遇到的语言问题

执行 `vim x` 出现下列错误：

```
Warning: Failed to set locale category LC_NUMERIC to en_CN.
Warning: Failed to set locale category LC_TIME to en_CN.
Warning: Failed to set locale category LC_COLLATE to en_CN.
Warning: Failed to set locale category LC_MONETARY to en_CN.
Warning: Failed to set locale category LC_MESSAGES to en_CN.
```

解决方法：
```
vim ~/.bash_profile 
#OR (vim  ~/.bashrc) 
export LC_ALL=en_US.UTF-8 #或者 zh_CN.UTF-8，可以通过locale -a查看系统内安装的locale
```

有的主机叫做“en_US.UTF-8” 有的用“en_US.utf8”
参加 https://stackoverflow.com/questions/56716993/error-message-when-starting-vim-failed-to-set-locale-category-lc-numeric-to-en



# 参考

- [How to Change or Set System Locales in Linux](https://www.tecmint.com/set-system-locales-in-linux/amp/):
- [把语言环境变量改为英文](https://wiki.ubuntu.com.cn/%E4%BF%AE%E6%94%B9locale)
- [Linux下使用locale命令设置语言环境](https://www.cnblogs.com/dolphi/p/3622570.html)
- [Ubuntu Linux: Start / Stop / Restart / Reload OpenSSH Server](https://www.cyberciti.biz/faq/howto-start-stop-ssh-server/)
- [Why is /etc/default/locale not sourced at login via ssh?](https://askubuntu.com/questions/749440/why-is-etc-default-locale-not-sourced-at-login-via-ssh)
- [CentOS6与CentOS7启动流程](https://www.linuxdiyf.com/linux/32089.html)