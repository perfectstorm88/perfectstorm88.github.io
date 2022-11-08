

# 要点说明

- 宿主主机只需要安装Nvidia驱动，无需安装CUDA驱动，因此可以通过docker切换不同的CUDA版本
- docker版本要求19以上

![NVIDIA Container Toolkit](https://cloud.githubusercontent.com/assets/3028125/12213714/5b208976-b632-11e5-8406-38d379ec46aa.png)

# 安装

已经完成docker engine和NVIDIA的安装，然后再安装nvidia-docker2
docker 安装 参考：https://docs.docker.com/engine/install/ubuntu/
## 设置 repository 和 GPG key

```bash
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list
```

## 安装nvidia-docker2

```bash
sudo apt-get update
sudo apt-get install -y nvidia-docdatker2
sudo systemctl restart docker
```

## 验证：使用nvidia/cuda验证(很小，100M)

执行以下命令：
```
docker run --rm --gpus all nvidia/cuda:10.2-base nvidia-smi 
docker run --gpus '"device=1,2"' nvidia/cuda nvidia-smi 
docker run --gpus 0 nvidia/cuda nvidia-smi # 指定设备0
```
如果输出跟直接在宿主机上执行 nvidia-smi 一致则说明安装成功。

## 验证2：使用pytorch镜像验证(比较大，2G)

```bash
docker run --gpus 0 -it —rm -v /home/lcz:/home/lcz  pytorch/pytorch:1.8.1-cuda10.2-cudnn7-runtime bash
```

验证cuda版本

```python
import torch
torch.cuda.is_available()
```

# 构建d2l的学习镜像

编写Dockerfile文件

```log
FROM pytorch/pytorch:1.8.1-cuda10.2-cudnn7-runtime
RUN pip install d2l
CMD ["jupyter","notebook","--port=8090","--allow-root","--ip=0.0.0.0","--no-browser","--NotebookApp.token='abc123'"]
```

构建镜像

```bash
docker build -t d2l-pytorch .
```

启动镜像

```bash
docker run --gpus all -p 8090:8090 -v /home/lcz:/workspace  -d d2l-pytorch
```

# 附加命令

##  查看cuda版本
```
cat /usr/local/cuda/version.txt
```
或者

```
nvcc -V
```

# 参考：
- [nvidia-docker官方文档](https://github.com/NVIDIA/nvidia-docker)
- [巧用 Docker 快速部署 GPU 环境](https://mp.weixin.qq.com/s/zVYPSTrrnvOtlPAjoXIs4Q)
- [NVIDIA Container Toolkit 安装](http://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html#docker)
- [通过docker访问jupyter内容](https://stackoverflow.com/questions/38830610/access-jupyter-notebook-running-on-docker-container)