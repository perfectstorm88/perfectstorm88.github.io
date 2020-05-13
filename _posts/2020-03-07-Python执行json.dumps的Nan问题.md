---
layout: post
categories: python
---

# 概述
python用json转的字符串，因为存在NaN导致web前端无法解析。
网上有些方法，如

- 自定义JSONEncoder 
- 使用simplejson包
- 执行pandas的fillna方法

但若普通list中已经存在np.NaN,则上述都无法生效。只能采用比较low字符串替换方法如：
```
json.dumps(obj).replace(', NaN',', 0').replace('NaN, ','0, ')
```

# 测试代码
```
class MyEncoder(json.JSONEncoder):
    def default(self, obj):
        print('a'*20,type(obj),obj)
        if isinstance(obj, numpy.integer):
            return int(obj)
        elif isinstance(obj, numpy.floating):
            return float(obj)
        elif isinstance(obj, numpy.ndarray):
            obj = np.where(np.isnan(obj), '', obj)
            return obj.tolist()
        else:
            return json.JSONEncoder.default(self, obj) 
            
# df = pd.DataFrame(np.random.randn(5, 3), index=['a', 'c', 'e', 'f', 'h'],
#                      columns=['one', 'two', 'three'])
df2 = np.array([1, 2, np.NaN])
df3 = np.array([np.NaN,0,0])
x = list(df2/df3)
print(json.dumps([[[df2],[x]]],cls=MyEncoder))
```

# 参考：
- [Python 优雅地 dumps 非标准类型](https://hexiangyu.me/2017/11/11/python-json-encode/)
- [Python JSON encoder convert NaNs to 'null' instead](https://stackoverflow.com/questions/28639953/python-json-encoder-convert-nans-to-null-instead)