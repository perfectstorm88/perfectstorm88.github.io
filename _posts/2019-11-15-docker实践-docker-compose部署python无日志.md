---
layout: post
categories: docker python
---

docker-compose部署python无日志，还奇怪的现象，很简单的解决

# 问题描述

使用docker-compose部署一个python程序时，启动后，通过 `docker logs -f` 命令没有任何日志，一开始以为是程序的bug，但是通过 `docker exec -it` 进入容器，直接通过python 命令行启动时ok。

而且其他的python程序日志是可以正常显示的，通过搜索找到答案
[docker-compose not printing stdout in Python app](https://stackoverflow.com/questions/51362213)
```
python -u xxx.py # 增加-u参数
```

`-u`参数含义：强制stdout和stderr不被缓存
```
-u     : force the stdout and stderr streams to be unbuffered;
         this option has no effect on stdin; also PYTHONUNBUFFERED=x
```