---
layout: post
categories: python
---

Pandas DataFrame连接表，Merge, Join, Concat的对比

Pandas.DataFrame操作表连接有三种方式：merge, join, concat。下面就来说一说这三种方式的特性和用法。

# merge
相当于SQL中的JOIN。该函数的典型应用场景是，两张表有相同内容的列（即SQL中的键），现在我们想把两张表整合到一张表里。
```
DataFrame.merge(self, right, how='inner', on=None, left_on=None, right_on=None, left_index=False, right_index=False, sort=False, suffixes=('_x', '_y'), copy=True, indicator=False, validate=None) → 'DataFrame'
```
- right：DataFrame或者named Series;self表示left的哪个Dataframe
- how：指的是合并(连接)的方式有inner(内连接),left(左外连接),right(右外连接),outer(全外连接);默认为inner！
- on : 指的是用于连接的列索引名称。必须存在右右两个DataFrame对象中，如果没有指定且其他参数也未指定，则以两个DataFrame的列名交集做为连接键
- left_on：左则DataFrame中用作连接键的列名;这个参数中左右列名不相同，但代表的含义相同时非常有用。
- right_on：右则DataFrame中用作 连接键的列名
- left_index：使用左则DataFrame中的行索引做为连接键，用到这个参数时，就有点类似于接下来要说的JOIN函数了。
- right_index：使用右则DataFrame中的行索引做为连接键
- sort：默认为True，将合并的数据进行排序。在大多数情况下设置为False可以提高性能
- suffixes：字符串值组成的元组，用于指定当左右DataFrame存在相同列名时在列名后面附加的后缀名称，默认为('_x','_y')
- copy：默认为True,总是将数据复制到数据结构中；大多数情况下设置为False可以提高性能
- indicator：在 0.17.0中还增加了一个显示合并数据中来源情况；如只来自己于左边(left_only)、两者(both)
- validate：支持one_to_one，one_to_many，many_to_one，many_to_many

默认以重叠列名当做链接键
也可以用行索引当连接键，使用参数left_index=True, right_index=True. 但是这种情况下最好用JOIN

# join
```
DataFrame.join(self, other, on=None, how='left', lsuffix='', rsuffix='', sort=False) → 'DataFrame'[source]
```


# concat
concat 轴向连接。就是单纯地把两个表拼在一起，这个过程也被称作绑定（binding）或堆叠（stacking）。因此可以想见，这个函数的关键参数应该是 axis，用于指定连接的轴向。axis=1 在行中操作，axis=0是在列中操作。
默认是axis=0,即垂直堆叠。

# 参考网站:
- https://zhuanlan.zhihu.com/p/45442554
- https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.join.html#pandas.DataFrame.join




