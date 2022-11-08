
# loguru
guru[gu:ru:]:翻译是古鲁、领袖、专家，loguru意指日志大师，日志领袖

![loguru](https://raw.githubusercontent.com/Delgan/loguru/master/docs/_static/img/logo.png)

![demo](https://raw.githubusercontent.com/Delgan/loguru/master/docs/_static/img/demo.gif)

重要概念：No Handler, no Formatter, no Filter: one function to rule them all
即“没有Handler，没有Formatter，没有Filter：一个函数控制所有”

```python
logger.add(sys.stderr, format="{time} {level} {message}", filter="my_module", level="INFO")
```
logger.add()用来注册sink(水槽)，决定消息写到哪里以及如何写，sink可以是下列格式:

- 文件格式的对象，如sys.stderr or open("somefile.log", "w")
- 字符串格式的文件路径
- 一个回调函数，例如`lambda msg: print(msg)`
- 一个协程函数，coroutine function（用async def 定义的）
- 一个内置的日志句柄，例如，logging.StreamHandler

logger.remove()用来清除sink

# 用法小记

## 清除所有logger，重新定位到文件

```python
from loguru import logger
logger.remove()
logger.add(f'./my.log')
logger.info('这条日志只输出到日志')
```

## 定义时间格式、消息格式

```python
logger.add('./logs/info.log', format="{time:YYYY-MM-DD HH:mm:ss.SSS}|{level}|{message}")
```

## 过滤不需要的日志


## 记录异常信息,挺好用的功能功能
```python
try:
    a=3/0
except Exception as e:
    logger.exception("What?!")
```
结果如下：
```log
2021-05-14 22:12:41.539 | ERROR    | __main__:<module>:35 - What?!
Traceback (most recent call last):

> File "./loguru_test.py", line 33, in <module>
    a=3/0

ZeroDivisionError: division by zero
```

## 上下文捕获异常(捕获未知异常时更好用))
```python
@logger.catch
def start(self):
   a=3/0
```

## 日志滚动、保留、压缩
```python
logger.add("file_1.log", rotation="500 MB")    # Automatically rotate too big file
logger.add("file_2.log", rotation="12:00")     # New file is created each day at noon
logger.add("file_3.log", rotation="1 week")    # Once the file is too old, it's rotated
logger.add("file_X.log", retention="10 days")  # Cleanup after some time
logger.add("file_Y.log", compression="zip")    # Save some loved space
```

# 参考
- [Delgan/loguru](https://github.com/Delgan/loguru)