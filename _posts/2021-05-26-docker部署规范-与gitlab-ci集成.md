# docker部署规范-与gitlab-ci集成

本规范根据多个项目实践，总结如下

## 持续集成的工程目录结构

一个完整产品会包含多个服务，比如web服务、java服务、python服务等
而且每个服务的代码都会对应不同的代码工程，工程目录结构如下所示

```bash
[产品名称]-web/             #【web前端工程】-->前端开发人员
  src/
  .gitlab-ci.yml     #提交代码后，触发CI操作
  build.sh           #构建docker镜像的辅助脚本，编译源码、build、tag、push等
  nginx.conf         #基于nginx构建的镜像，关于nginx的预定义配置
  Dockerfile         
[产品名称]-java/            #【java工程】-->java开发人员
  src/
  .gitlab-ci.yml   
  build.sh   
  Dockerfile 
[产品名称]-python/          #【python工程】-->python开发人员
  src/
  .gitlab-ci.yml   
  pre.Dockerfile     #python的包安装比较耗时间，进行pre镜像构建
  Dockerfile         #每次修改代码，基于pre镜像构建，速度快
  build.sh   
[产品名称]-deploy           #【部署工程】-->架构师、运维人员
  conf
    docker-compose.yml #编排脚本
  init-data  #初始化的数据，如数据库脚本、程序加载时的初始数据等
  run-data   #运行过程中的数据，如数据库存储目录，程序的临时文件目录
  log        #程序日志目标
  back       #程序备份目录
```

## 构建镜像build.sh
通过build.sh脚本构建镜像，执行命令`build.sh 工程名 分支名`

- 读取工程名和分支名
- 根据源代码构建项目
- 构建镜像,镜像名格式为:`仓储地址/工程名:分支名`
- push镜像到仓储服务器

对于第二步`根据源代码构建项目`,不同技术栈的build.sh写法会有不同,有时相同技术栈的写法也略有差异

```bash
# 读取工程名和分支名
if [ -n "$1" ] ; then reposityName=$1; else echo -e "usage:\n\t./build.sh reposityName [tagName]" && exit 1 ;  fi
if [ -n "$2" ] ; then tagName=$2; else tagName="master" ; fi
imageName=$reposityName:$tagName

echo "===>step1: build web for $imageName"
npm install
if [ -d dist ]; then rm -rf dist; fi
npm run build:prod
echo "===>step2: build docker image:$imageName"
docker build -t $imageName .
imageName2=10.1.192.120:5000/$imageName
docker tag $imageName $imageName2
echo "===>step3: push docker image to reposity"
docker push  $imageName2
```

## Dockerfile说明

不同技术栈的对应的Dockerfile是不同的

### web服务的Dockerfile

```dockerfile
FROM nginx
# Create app directory
RUN mkdir -p /usr/src/app
# 拷贝部署文件
COPY dist  /usr/src/app/
COPY nginx.conf  /etc/nginx/
```

### java服务的Dockerfile

```dockerfile
FROM java:8

WORKDIR /usr/local/app
# 拷贝jar到镜像中，对应不同项目，此处的jar包名称是不一样的
COPY ./scientific-system/target/scientific-system-2.6.jar  /usr/local/app/
## wait小工具，等待依赖的服务就绪
COPY ./wait /wait  
RUN chmod +x /wait
CMD /wait &&  java -jar scientific-system-2.6.jar
```

### python服务的Dockerfile

因为python若有头开始打包，消耗的时间比较长，所以分为两步，

- 当需要安装依赖包时，由Pre.Dockerfile进行预构建
- 当项目源代码改变时，触发Dockerfile进行构建

Pre.Dockerfile的样例如下：

```dockerfile
FROM centos7-python:3.8.10
COPY requirements.txt .
RUN pip install  -r requirements.txt
```

当需要安装依赖包时，以人工的方式执行`docker build -f Pre.Dockerfile -t 产品名称:pre  .`,先构建出一个中间镜像，当源代码提交时，对应的Dockerfile如下:

```dockerfile
FROM 产品名称:pre
WORKDIR /usr/local/app
COPY .  /usr/local/app
ENTRYPOINT ["python"]
CMD ["-u","main_api.py"]
```

## 与CI集成的.gitlab-ci.yml

- 计算工程目录前缀：规则`产品名称_deploy_分支名称`
- 构建项目：
  - 进入源代码目录
  - 切换分支，拉取最新代码
  - 构建镜像，并上传到镜像仓储服务器,镜像名格式：`仓储服务器/工程名:分支名称`
- 更新集群服务：
  - 进入部署目录
  - 拉取镜像,镜像名格式：`仓储服务器/工程名:分支名称`
  - 重启服务

```yaml
stages:
  - deploy
deploy:
  stage: deploy
  tags:
    - ai-test-by-shell
  script:
    # 计算工程目录前缀,可以自动提取前两个单词，也可以人工写死，如export PROJECT_DEPLOY=ywz_deploy
    - export PROJECT_DEPLOY=$(echo $CI_PROJECT_NAME |awk -F '_' '{print $1"_"$2"_deploy"}')
    - export PROJECT_DEPLOY="${PROJECT_DEPLOY}_${CI_COMMIT_BRANCH}"  # 与分支进行组合，ywz_deploy_master
    - echo "CI_PROJECT_NAME=$CI_PROJECT_NAME   CI_COMMIT_BRANCH=$CI_COMMIT_BRANCH PROJECT_DEPLOY=$PROJECT_DEPLOY"
    # 构建项目
    - source /root/.bashrc && cd /home/build/$CI_PROJECT_NAME  && git checkout $CI_COMMIT_BRANCH && git pull && sh build.sh $CI_PROJECT_NAME $CI_COMMIT_BRANCH
    # 更新集群  
    - ssh root@w118 "source /root/.bashrc && cd /home/build/${PROJECT_DEPLOY} && docker pull 10.1.192.120:5000/$CI_PROJECT_NAME:$CI_COMMIT_BRANCH && docker-compose up -d"
  only:
    - master
```

# 参考：

- [后面带下划线的变量如何处理](https://unix.stackexchange.com/questions/358272/bash-variable-substitution-of-variable-followed-by-underscore)

