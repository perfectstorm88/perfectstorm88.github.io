---
layout: post
categories: python
---


# 操作步骤
- 下载SimHei.tff
- 复制中文字体文件到matplotlib的fonts/ttf目录。完整的目录为/usr/local/lib/python*/dist-packages/matplotlib/mpl-data/fonts/ttf
- 修改配置/usr/local/lib/python*/dist-packages/matplotlib/mpl-data/matplotlibrc文件
```
# 搜索font.family配置项，将其#注释去掉，并将：号后面的值改为字段对应的名字。
font.family         : SimHei
 
# 搜索axes.unicode_minus配置项，将其#注释去掉，并将：号后面的值改为False
axes.unicode_minus  : False
```
- 清空matplotlib使配置生效`rm ~/.cache/matplotlib -R`

# 参考：

- [ubuntu下解决matplotlib生成图片中文乱码](https://blog.csdn.net/huuinn/article/details/78968966)：centos也适用
- [SimHei](https://github.com/StellarCN/scp_zh)：字体库包含黑体(SimHei.tff)