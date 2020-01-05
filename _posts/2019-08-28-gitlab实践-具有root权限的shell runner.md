---
layout: post
categories: Linux 系统管理 gitlab
---

gitlab-runner真是个好东西，尤其是shell runner,喜欢的不要不要的，什么脏活累活，自动化任务都可以交给它。

随着现在云计算普及，5-6个人的小团队都会有好几台后台服务器，Linux上那套用户权限管理感觉太多余了，直接root操作多爽。

鉴于官网手册[Install GitLab Runner manually on GNU/Linux](https://docs.gitlab.com/runner/install/linux-manually.html) 还是用非gitlab-runner用户安装，导致写shell时会额外增加权限调整的工作量。在此专门写个小文说明下：

# 安装runner

```bash
# Linux x86-64
sudo curl -L --output /usr/local/bin/gitlab-runner https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-linux-amd64

sudo chmod +x /usr/local/bin/gitlab-runner

sudo gitlab-runner install --user=root --working-directory=/home/gitlab-runner # 官方文档还是user=gitlab-runner
sudo gitlab-runner start
```
# 注册runner

## step1 在gitlab上找到注册runner页面
例如:http://gitlab.lingxi.co/admin/runners,记录下url和token
![gitlab-register-runner](http://img.lichangzhen.top/2019/gitlab-register-runner.jpg)

## step2 在后台服务器上注册Runner

执行`sudo gitlab-runner register`命令
```bash
untime platform                                    arch=amd64 os=linux pid=30202 revision=1f513601 version=11.10.1
Running in system-mode.

Please enter the gitlab-ci coordinator URL (e.g. https://gitlab.com/):
http://gitlab.lingxi.co/
Please enter the gitlab-ci token for this runner:
MbXYws71x3X2isnQdE_-    # 次数输入token
Please enter the gitlab-ci description for this runner:
[ops2]: shell runner test
Please enter the gitlab-ci tags for this runner (comma separated):
test
Registering runner... succeeded                     runner=MbXYws71
Please enter the executor: docker-ssh+machine, kubernetes, docker-ssh, parallels, ssh, virtualbox, docker+machine, docker, shell:
shell   # executor类型为shell
Runner registered successfully. Feel free to start it, but if it's running already the config should be automatically reloaded!
```

执行步骤参见[Registering Runners](https://docs.gitlab.com/runner/register/index.html)

然后就可以在gitlab页面上可以看到注册状态了
http://img.lichangzhen.top/2019/gitlab-register-runner-status.jpg
http://img.lichangzhen.top/2019/gitlab-register-runner-status.jpg
![gitlab-register-runner-status](http://img.lichangzhen.top/2019/gitlab-register-runner-status.jpg)

## step2 修改runner参数，比如并发数等

执行`ps -ef|grep gitlab` 查看运行状态
```
 root     24171     1  0 11:33 ?        00:00:01 /usr/lib/gitlab-runner/gitlab-runner run --working-directory /home/gitlab-runner --config /etc/gitlab-runner/config.toml --service gitlab-runner --syslog --user root
```
执行`vim /etc/gitlab-runner/config.toml`,可以修改concurrent=3（默认为一个并发）
```toml
concurrent = 3
check_interval = 10
[session_server]
  session_timeout = 1800
[[runners]]
  name = "mg_ci_111"
  url = "http://gitlab.lingxi.co/"
  token = "Co1UmjA6zWzBmcLss4xx"
  executor = "shell"
   [runners.custom_build_dir]
   [runners.cache]
     [runners.cache.s3]
     [runners.cache.gcs]
```

执行`service gitlab-runner restart`重启服务，就可以完成了整个部署