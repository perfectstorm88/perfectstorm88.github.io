---
layout: post
categories: nodejs
---

# node-gyp rebuild错误描述
```
/Users/lcz/.go/src/github.com/hyperledger/fabric-samples/fabcar/javascript>npm install  pkcs11js

> pkcs11js@1.0.19 install /Users/lcz/.go/src/github.com/hyperledger/fabric-samples/fabcar/javascript/node_modules/pkcs11js
> node-gyp rebuild

gyp WARN download NVM_NODEJS_ORG_MIRROR is deprecated and will be removed in node-gyp v4, please use NODEJS_ORG_MIRROR
gyp ERR! configure error
gyp ERR! stack Error: Command failed: /Users/lcz/miniconda3/bin/python -c import sys; print "%s.%s.%s" % sys.version_info[:3];
gyp ERR! stack   File "<string>", line 1
gyp ERR! stack     import sys; print "%s.%s.%s" % sys.version_info[:3];
gyp ERR! stack                                ^
gyp ERR! stack SyntaxError: invalid syntax
gyp ERR! stack
gyp ERR! stack     at ChildProcess.exithandler (child_process.js:294:12)
gyp ERR! stack     at ChildProcess.emit (events.js:182:13)
gyp ERR! stack     at maybeClose (internal/child_process.js:962:16)
gyp ERR! stack     at Socket.stream.socket.on (internal/child_process.js:381:11)
gyp ERR! stack     at Socket.emit (events.js:182:13)
gyp ERR! stack     at Pipe._handle.close (net.js:610:12)
gyp ERR! System Darwin 18.7.0
gyp ERR! command "/Users/lcz/.nvm/versions/node/v10.14.2/bin/node" "/Users/lcz/.nvm/versions/node/v10.14.2/lib/node_modules/npm/node_modules/node-gyp/bin/node-gyp.js" "rebuild"
gyp ERR! cwd /Users/lcz/.go/src/github.com/hyperledger/fabric-samples/fabcar/javascript/pkcs11js
gyp ERR! node -v v10.14.2
gyp ERR! node-gyp -v v3.8.0
gyp ERR! not ok
```
解决方法：
```bash
npm i -g node-gyp
node-gpy -v #
npm config set node-gyp  `which node-gyp` # 此处可能会有坑，比如node-gpy -v是个新版本，但是npm安装时是个旧版本
```
原因：

- 虽然重新安装了node-gyp，但是npm install时，默认使用的node-gyp -v v3.8.0 太老，必须要通过npm config set node-gyp 设置
  - 否则此处可能会有坑，比如node-gpy -v是个新版本，但是npm安装时是个旧版本
- 网上很多例子，但是针对mac，很多系统指向到xcode的问题，所以浪费了很多时间

# node-gyp是什么？

要理解node-gyp首先要知道什么是gyp(https://gyp.gsrc.io/index.md)。gyp其实是一个用来生成项目文件的工具，一开始是设计给chromium项目使用的，后来大家发现比较好用就用到了其他地方。生成项目文件后就可以调用GCC, vsbuild, xcode等编译平台来编译。至于为什么要有node-gyp，是由于node程序中需要调用一些其他语言编写的工具甚至是dll，需要先编译一下，否则就会有跨平台的问题，例如在windows上运行的软件copy到mac上就不能用了，但是如果源码支持，编译一下，在mac上还是可以用的。

# 参考：
- [node-gyp的作用是什么?](https://www.zhihu.com/question/36291768)