---
layout: post
categories: python 
---

不管是mac还是linux都默认安装了python环境，但是版本偏低，开发和部署环境一般都需要安装最新版本，但是python官网上有没有quickstart的安装说明，
比较好用的是anaconda

之前觉得anaconda还是非常好用的，今天登陆官网，发现都是GUI安装，又重又笨，间接快速的命令行安装放在比较隐藏的位置。操作步骤专门记录下：

# 通过shell安装

```
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O ~/miniconda3.sh
# wget https://repo.anaconda.com/miniconda/Miniconda3-latest-MacOSX-x86_64.sh -O ~/miniconda3.sh
bash ~/miniconda3.sh -b -p $HOME/miniconda3
```

# 设置环境变量

```
echo 'export PATH=~/miniconda3/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
```

# 检查python版本

```
python --version 
#结果为Python 3.7.3
```