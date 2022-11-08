
@[TOC]

# 需求

- 用docker快速部署jupyter环境，方便跟别人分享jupyter链接
- jupyter的安全较差，在docker环境运行更重要的是能保证安全，哪怕被黑客攻击了，只会影响容器内环境，不会影响到整个主机环境
- 内置了Simhei字体，支持用matplotlib中文制图

# docker方式启动

## dockerfile代码
```dockerfile
FROM centos7-python:3.8.10
ENV NLS_LANGE="SIMPLIFIED CHINESE_CHINA.ZHS16GBK"
# 安装中文字体(中易黑体)，支持matplotlib中文梯子
RUN yum -y install fontconfig && yum clean all
COPY SimHei.ttf /usr/share/fonts/
# 常用的库，如pandas、matplotlib、excel、yaml等
RUN pip install matplotlib jupyter openpyxl pandas PyYAML

COPY requirements.txt .
RUN pip install --trusted-host mirrors.aliyun.com  -i http://mirrors.aliyun.com/pypi/simple/  -r requirements.txt

WORKDIR /work
CMD ["python","-m","jupyter","notebook","--port=8888","--allow-root","--ip=0.0.0.0","--no-browser","--NotebookApp.token='your password'"]
```

## 构建镜像
```
docker build -f Dockerfile -t docker-jupyter-zh  .
```
## 启动镜像
```
docker run -d --rm --name docker-jupyter-zh -v $(pwd)/work:/work -p 8092:8888 docker-jupyter-zh
```

# 更简洁的docker-compose方式

```yaml
version: "3"
services:
  docker-jupyter-zh:
    build: .
    ports:
      - "8092:8888"
    volumes:
          - ./work:/work
    command: ["python","-m","jupyter","notebook","--port=8888","--allow-root","--ip=0.0.0.0","--no-browser","--NotebookApp.token='your password'"]
```
启动命令：
```bash
docker-compose up -d  
```

# 验证服务
就可以通过 http://IP地址:8092/ 进行方式页面访问