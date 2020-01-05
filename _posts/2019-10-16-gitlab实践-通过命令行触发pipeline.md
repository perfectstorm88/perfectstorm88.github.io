---
layout: post
categories: 系统管理 gitlab
---

# 原理
应用场景如下：通过http触发gitlab的pipeline，执行.gitlab-ci.yml，就可以做你想做的事情了

![gitlab-trigger-pipeline-by-http](http://img.lichangzhen.top/2019/gitlab-trigger-pipeline-by-http.jpg)

# 配置和测试过程

## 在gitlab的工程上，创建一个trigger token
your project's **Settings**->**CI/CD**->**Pipeline triggers**

![triggers_page](https://gitlab.com/help/ci/triggers/img/triggers_page.png)


## 对应的http命令样例
样例如下
```bash
curl -X POST \
     -F token=TOKEN \  # 上面生成的token
     -F "ref=dev" \                             # 只能是branch或者tags
     -F "variables[DESCRIPTION]=deb" \          # 传递给job的变量
     http://gitlab.lingxi.co/api/v4/projects/360/trigger/pipeline
```

- 首先http的域名为自己的域名，例子中为http://gitlab.lingxi.co
- projectId 通过**project**->**Details**查看，例子中为360
- token 为上面创建的trigger token
- ref 只能是这个project已经存在的branch名称或者tags名称
- variables 用来传递给job的变量，这些变量可以在.gitlab-ci.yml中使用

# 真实案例:用户远程重启后台的多个主机上的服务

![restart-remote-service-by-web-user](http://img.lichangzhen.top/2019/restart-remote-service-by-web-user.jpg)

## 业务后台的触发函数
```js
let restartService = function(ref,client){
    const exec = require('child_process').exec;
    let cmd = `curl -X POST -F token=587612a50640ffe453880adc6470d7 -F ref=${ref} -F variables[client]=${client}  http://gitlab.lingxi.co/api/v4/projects/360/trigger/pipeline`
    exec(cmd, function(err, stdout, stderr) {
        console.log(stdout);
        console.error(stderr);
    });
}
```
## 部署工程的.gitlab-ci.yml代码

在多个主机上，远程执行deploy.sh文件
```yaml
stages:
  - deploy_dev
  - deploy_production

deploy_production:
  stage: deploy_production
  tags:
    - mg_ci_111
  script:
    - cd /home/gitlab-runner/jfjun-mg-deploy
    - git checkout master
    - ssh root@47.99.73.225 "bash -s" < deploy.sh ${client}  # 主机1上执行
    - ssh root@47.110.145.231 "bash -s" < deploy.sh ${client} # 主机2上执行
  only:
    - master
```

## 执行脚本deploy.sh的内容

```bash
#!/bin/bash
. ~/.nvm/nvm.sh
ROOTPATH=$PWD
echo args: $*
[ $# -ne 1 ] && exit 0;
echo root path is: $ROOTPATH
PROJECTS="$(ls |grep jfjun-mg|grep -v jfjun-mg-base)"
echo all mg projects: $PROJECTS;
for project in ${PROJECTS}
do
    cd $ROOTPATH/$project
    des=$(cat local.json|grep name|cut -d':' -f 2|grep -o "[^,\"\ ]*");
    echo $des
    if [[ $des == $1 ]]; then
        git pull
        npm run gulp pm2 # 重启命令
    fi;
done
```

# 参考

- [Pipelines API](https://docs.gitlab.com/ee/api/pipelines.html)
- [Pipelines API](https://gitlab.com/help/api/pipelines.md)
- [Triggering pipelines through the API](http://gitlab.comd/help/ci/triggers/README.md)
- [ssh 远程执行命令](https://www.cnblogs.com/youngerger/p/9104144.html)
