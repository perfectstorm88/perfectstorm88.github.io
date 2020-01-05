---
layout: post
categories: python
---

# 软件准备

- python，建议通过conda安装python3.6及以上，[下载链接](https://repo.anaconda.com/miniconda/)
- VScode，最新版本https://code.visualstudio.com/

# 安装ython插件
1、打开VScode，按下快捷键Cmd+Shift+X，进入插件管理页面。
2、在搜索栏输入python。
3、选择插件，点击安装。
![a](https://pic3.zhimg.com/v2-44c0a6007b2d4bbe7c76beeb99ffe16a_b.jpg)

# 首次运行python程序
1. 创建一个文件hello.py
```
print("Hello World")
```
2. 通过cmd+shift+P执行Python:Select interpreter选择对应python版本的解释器
3. 执行文件
   1. 通过cmd+shift+P执行Python:Run Python File in Terminal，执行文件
   2. 或者右键，选择Run Python File in Terminal
   3. 后者python文件右上角的运行按钮(绿色三角形)

至此就完成了vscode中执行python的基本操作

# 特性:debuging
根据官方文档[Debuging](https://code.visualstudio.com/docs/python/debugging),总结如下：

1. 打开`Debug view`
2. 选择debuging的设置按钮，即`open the launch.json`，或者 **Debug > Open configurations**
3. 增加一个Configuration，可以通过launch.json的**Add Configuration**按钮，或者**Debug > Add configuration**

![add-configuration](https://code.visualstudio.com/assets/docs/python/debugging/add-configuration.png)

4. 在debuging期间，状态条显示当前的状态信息,如下所示
   1. 显示当前python解释器，也可以点击重新选择
   2. 显示当前默认的launch configuration, 也可以点击重新选择
   3. 编码格式
   4. 告警信息等
  

![b](https://code.visualstudio.com/assets/docs/python/debugging/debug-status-bar.png)

## launch.json的配置选项
样例文件如下：
```
    {
      "name": "Python: Current File (Integrated Terminal)",
      "type": "python",
      "request": "launch",
      "program": "${file}",
      "console": "integratedTerminal"
    },
```

- name:debug configuration的下拉列表中显示的名字
- type：debuger的类型如python,nodejs
- request:debug的模式
  - launch：启动一个通过program定义的文件
  - attach: attach到一个运行进程上,参见[Remote debugging](https://code.visualstudio.com/docs/python/debugging#_remote-debugging)
- program:程序入口，${file}或者具体的文件路径
- pythonPath:指向包含python解释器的文件夹
- args: 启动参数
- stopOnEntry: 是否在程序的第一个行中断,默认为false，是在断点中断
- console:输出显示的位置，有三种类型
  - internalConsole: 对应Debug Console
  - integratedTerminal：
  - externalTerminal
- cwd：debugger的当前工作目录，默认是${workspaceFolder} 
- redirectOutput:默认是true，即把程序output定向到debug output 中。如果是false,程序结果不会显示在debug output中
- justMyCode：默认为true，即只限于用户自己的代码，如果是false，则可以定义标准库的代码
- django：Django web framework
- pyramid： Pyramid app 
- env:自定义的环境变量
- envFile:执行一个自定义的环境变量文件
- gevent: 可以支持gevent monkey-patched code,gevent是一个基于协程的网络库，可以提高python并发

## Attach to a local script
调试一个被其它进程调用的python文件，例如web服务器
## Remote debugging
当程序运行在远端主机时，也可以通过VSCode进行本地debug

### ptvsd介绍
https://github.com/Microsoft/ptvsd/


# 参考

[Python in Visual Studio Code](https://code.visualstudio.com/docs/languages/python)
[用VScode配置Python开发环境](https://zhuanlan.zhihu.com/p/31417084)