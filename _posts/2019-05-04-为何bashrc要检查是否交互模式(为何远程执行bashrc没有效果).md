---
layout: post
categories: Linux 系统管理
---
在执行`ssh  root@localhost ". ~/.bashrc && which node"` 发现根本没有执行`.bashrc`文件

检查bashrc文件，发现有代码如下:
```bash
# If not running interactively, don't do anything
[ -z "$PS1" ] && return
```
知道原因了，但是却是不能注释的，因为"远程文件传输会失败!"

## 为何所有的Linux发行版本都检查这个呢，因为:
- 远程文件传输会失败
- .bashrc保存了客户自定义的配置，比如变量、alias、提示等，对于非交互shell中，这些是无意义的。

## 测试注释后的效果：
如果把`[ -z "$PS1" ] && return` 注释掉，执行下面语句是失败的
```
echo 'aaa'>> a.txt && scp a.txt  root@localhost:~/b.txt
```

# 阅读:
- [Why does bashrc check whether the current shell is interactive?](https://unix.stackexchange.com/questions/257571/why-does-bashrc-check-whether-the-current-shell-is-interactive)

