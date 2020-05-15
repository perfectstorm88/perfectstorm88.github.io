---
layout: post
categories: python
---

Pandas DataFrame连接表，Merge, Join, Concat的对比

Pandas.DataFrame操作表连接有三种方式：merge, join, concat。下面就来说一说这三种方式的特性和用法。

# merge 通过键拼接列
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

# join 拼接列，主要用于索引上的合并
```
DataFrame.join(self, other, on=None, how='left', lsuffix='', rsuffix='', sort=False) → 'DataFrame'[source]
```
- 只要两个表列名不同，不加任何参数就可以直接用。
- 如果两个表有重复的列名，需指定lsuffix, rsuffix参数。
- 其中参数的意义与merge方法基本相同,只是join方法默认为左外连接


# concat 可以沿着一条轴将多个对象堆叠到一起(列合并或者行合并)
concat 轴向连接。就是单纯地把两个表拼在一起，这个过程也被称作**绑定（binding）**或**堆叠（stacking）**。因此可以想见，这个函数的关键参数应该是 axis，用于指定连接的轴向。axis=1 在行中操作，axis=0是在列中操作。

默认是axis=0,即垂直堆叠，样例如下
```python
df1=pd.DataFrame(np.random.randn(3,4),columns=['a','b','c','d'])
df2=pd.DataFrame(np.random.randn(2,3),columns=['b','d','a'])
pd.concat([df1, df2], axis=1) # 对行操作，相当于水平连接
#注意到这里，左表和右表没有一个单元格是一样的，只是按照行索引水平堆在了一起，所以可以理解为相当于
pd.merge(df1,df2,left_index=True,right_index=True,how='outer') 
#或者
df1.join(df2, lsuffix="_l")
```
垂直堆叠就是axis=0，这种情况下有个参数比较特殊，叫' ignore_index= '，默认情况下是False。如果设成了True，就是把结果的合并表重新编排行索引。否则，行索引还是原来两个表里的值，比如"0,1,2,0,1"

```python
pd.concat([df1, df2], axis=0, ignore_index=True)
```
如果两张表的列名都不相同，垂直堆叠会扩展不同的列，生成一张更宽的表

# 总结
- 拼接列
  - 通过键值：用merge
  - 通过行索引：用join或者concat
    - 如果两个以上：concat
  - 创建层次化索引：用concat
- 拼接行
  - 用concat

concat的参数较复杂，还有很多未知参数

# 参考网站:
- [Pandas DataFrame连接表，Merge, Join, Concat的对比](https://zhuanlan.zhihu.com/p/45442554)
- https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.join.html#pandas.DataFrame.join
- [【pandas】[3] DataFrame 数据合并，连接（merge,join,concat)](https://blog.csdn.net/zutsoft/article/details/51498026)



