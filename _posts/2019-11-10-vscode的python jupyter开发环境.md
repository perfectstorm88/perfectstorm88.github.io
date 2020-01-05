---
layout: post
categories: python
---

# 软件准备

- python，建议通过conda安装python3.6及以上，[下载链接](https://repo.anaconda.com/miniconda/)
  - python环境要安装jupyter
- VScode，最新版本https://code.visualstudio.com/

# Create, open, and save Jupyter Notebooks

- 通过 **Python: Create Blank New Jupyter Notebook** 创建一个新的.ipynb文件
- 通过 ** Python: Open in Notebook Editor** 打开文件，或者双击文件，或者右键菜单选择
- 保存

# Work with Jupyter code cells

- cell有三种工作状态:
  - unselected
  - 命令模式
  - 编辑模式，两个模式间通过esc和enter切换
- 增加cell:   
  - 通过+按钮
  - 命令模式下 A above或者B below
- 选择cell: 命令模式下通过J(up)K（down）或者上下键
- 运行单个cell：
  - Ctrl+Enter：运行
  - Shift+Enter：运行，下面插入新的，并且焦点移到新的cell
  - Alt+Enter：运行，下面插入新的，并且焦点在老的的cell（目的啥？？？）
- 运行所有cell
- 上下移动cell(比网页方便)
- 删除一个cell
- 在markdown和code间切换
- **支持IntelliSense**（比网页方便）




# View, inspect, and filter variables using the Variable explorer and Data viewer

- 查看所有变量,比网页方便多了

![a](https://code.visualstudio.com/assets/docs/python/jupyter/native-variable-explorer.png)

- Plot viewer：可以放到、缩小、导航
  - plot的图可以，如果graphviz的图片就不可以使用plot viewer功能

# Debug a Jupyter notebook

- 把ipynb保存为.py
- debug并且修改
- use the Python Interactive window to export the Python file as a Jupyter Notebook (.ipynb)

# 参考

[Working with Jupyter Notebooks in Visual Studio Code](https://code.visualstudio.com/docs/python/jupyter-support)