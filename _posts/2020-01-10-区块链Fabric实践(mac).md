---
layout: post
categories: 区块链
---

# 预安装
## 安装docker和docker-compose

## 安装go(mac)
参考：https://ahmadawais.com/install-go-lang-on-macos-with-homebrew/

修改.bashrc文件或者.bash_profile文件
```
# Go development
export GOPATH="${HOME}/.go"
export GOROOT="$(brew --prefix golang)/libexec"
export PATH="$PATH:${GOPATH}/bin:${GOROOT}/bin"
```
创建GOPATH目录
```
test -d "${GOPATH}" || mkdir "${GOPATH}"
test -d "${GOPATH}/src/github.com" || mkdir -p "${GOPATH}/src/github.com"
brew install go
```

## 下载Fabric
在GOPATH目录下的结构如下所示
```
-src
---github.com
------hyperledger
--------fabric
--------fabric-samples
```

下载fabric和fabric-samples
```bash
# 
# 进入文件hyperledger文件路径,$GOPATH/src/github.com为上个步骤创建的文件夹
mkdir -p $GOPATH/src/github.com/hyperledger && cd $GOPATH/src/github.com/hyperledger
# 从git上克隆fabric项目
git clone https://github.com/hyperledger/fabric.git
cd fabric
git tag #查看所有版本，点击q退出
git checkout v1.4 #切换到tag中你想要切换到的版本

cd ..  #退回到hyperledger文件夹
# 从git上克隆fabric相关例子
git clone https://github.com/hyperledger/fabric-samples.git
cd fabric-samples
git tag #查看所有版本，点击q退出
git checkout v1.0.6 #切换到fabric相同的版本
```
下载运行hyperledger fabric所需要的docker 镜像和二进制文件
```
cd $GOPATH/src/github.com/hyperledger/fabric-samples
bash bootstrap.sh
```
下载结果检查
```bash
docker images |grep hyperledger #检查是否包含下列镜像
#hyperledger/fabric-ca 
#hyperledger/fabric-tools
#hyperledger/fabric-ccenv   
#hyperledger/fabric-orderer
#hyperledger/fabric-peer
#hyperledger/fabric-zookeeper
#hyperledger/fabric-kafka
#hyperledger/fabric-couchdb
cd $GOPATH/src/github.com/hyperledger/fabric-samples/bin && ls # 检查二进制文件是否下载完毕
#configtxgen		cryptogen		fabric-ca-client	idemixgen		peer
#configtxlator		discover		get-docker-images.sh	orderer
```
至此，完成了构建fabric网络的预安装工作，下面开始构建第一个fabric网络

# 构建第一个fabric网络

## 1.查看byfn.sh帮助信息
```bash
./byfn.sh -h
Usage:
  byfn.sh <mode> [-c <channel name>] [-t <timeout>] [-d <delay>] [-f <docker-compose-file>] [-s <dbtype>] [-l <language>] [-i <imagetag>] [-v]
    <mode> - one of 'up', 'down', 'restart', 'generate' or 'upgrade'
      - 'up' - bring up the network with docker-compose up
      - 'down' - clear the network with docker-compose down
      - 'restart' - restart the network
      - 'generate' - generate required certificates and genesis block
      - 'upgrade'  - upgrade the network from version 1.1.x to 1.2.x
    -c <channel name> - channel name to use (defaults to "mychannel")
    -t <timeout> - CLI timeout duration in seconds (defaults to 10)
    -d <delay> - delay duration in seconds (defaults to 3)
    -f <docker-compose-file> - specify which docker-compose file use (defaults to docker-compose-cli.yaml)
    -s <dbtype> - the database backend to use: goleveldb (default) or couchdb
    -l <language> - the chaincode language: golang (default) or node
    -i <imagetag> - the tag to be used to launch the network (defaults to "latest")
    -v - verbose mode
  byfn.sh -h (print this message)

Typically, one would first generate the required certificates and
genesis block, then bring up the network. e.g.:

	byfn.sh generate -c mychannel
	byfn.sh up -c mychannel -s couchdb
        byfn.sh up -c mychannel -s couchdb -i 1.2.x
	byfn.sh up -l node
	byfn.sh down -c mychannel
        byfn.sh upgrade -c mychannel

Taking all defaults:
	byfn.sh generate
	byfn.sh up
	byfn.sh down
```
## 2.生成网络
第一步生成我们各种网络实体的所有证书和密钥，`genesis block`用于引导排序服务，以及配置`Channel`所需要的一组交易配置集合。
```bash
./byfn.sh -m generate # 如果你选择不提供channel名称，则脚本将使用默认名称mychannel
```
## 3.启动网络
```bash
./byfn.sh -m up
```
启动所有容器，驱动一个端到端的应用场景
## 4.关闭网络
```bash
./byfn.sh -m down
```
将关闭容器，移除加密材料和4个配置信息
## 5.
# 【重新】运行fabric-samples进行学习，其他文档太乱了！！
[Writing Your First Application](https://hyperledger-fabric.readthedocs.io/en/master/write_first_app.html#launch-the-network)

## 使用fabric-samples的官方案例
工程地址:https://github.com/hyperledger/fabric-samples ,使用版本为2.0.0-beta版本，代码如下(整合了官方的readme文档)
```bash
git clone https://github.com/hyperledger/fabric-samples # 下载样例工程
cd fabric-samples #
# Fetch bootstrap.sh from fabric repository using
curl -sS https://raw.githubusercontent.com/hyperledger/fabric/master/scripts/bootstrap.sh -o ./scripts/bootstrap.sh
# Change file mode to executable
chmod +x ./scripts/bootstrap.sh
# Download binaries and docker images
./scripts/bootstrap.sh 2.0.0-beta 1.4.4 0.4.18 #格式为./scripts/bootstrap.sh [version] [ca version] [thirdparty_version]
# 参考https://hyperledger-fabric.readthedocs.io/en/master/install.html，最新模型参数
```
## 启动网络和服务节点
下面脚本建立并运行了一个示例网络，并且已安装并实例化FabCar智能合约
```
cd fabcar
./startFabric.sh javascript
```
## 验证功能
当我们创建网络时，创建了一个管理员用户（字面上称为admin）作为证书颁发机构（CA）的注册商,第一步是使用enroll.js程序为管理员生成私钥，公钥和X.509证书,
```bash
node enrollAdmin.js #证书保存到wallet目录
```
有了管理员的凭据，可以在电子钱包中注册一个新用户`user1`，该用户将用于查询和更新
```bash
node registerUser.js  
```
区块链网络中的每个对等方都托管ledger的副本，应用程序可以通过调用智能合约来查询分类账
```bash
node query.js
```
更新ledger
```
node invoke.js
```

# 参考
- [Mac环境安装Hyperledger Fabric](https://www.jianshu.com/p/a59ff954d3b2):mac安装参考大部分，不过文中一些脚本的目录是不对的
- [构建第一个fabric网络](https://hyperledgercn.github.io/hyperledgerDocs/build_network_zh)：
- [百度区块链xuperchain](https://xchain.baidu.com/)