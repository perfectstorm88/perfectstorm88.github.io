
# 最简单的案例

```python
# -*- coding: utf-8 -*-
"""
https://pythonbasics.org/flask-sessions/
[python--Flask学习（三）Flask中的session操作](https://blog.csdn.net/weixin_42417767/article/details/96618763)
"""
from flask import Flask, session, redirect, url_for, escape, request
app = Flask(__name__)
app.secret_key = 'any random string'

@app.route('/')
def index():
   print(session)
   if 'username' in session:
      username = session['username']
      return 'Logged in as ' + username + '<br>' + "<b><a href = '/logout'>click here to log out</a></b>"
   return "You are not logged in <br><a href = '/login'>" + "click here to log in</a>"


@app.route('/login', methods = ['GET', 'POST'])
def login():
   if request.method == 'POST':
      session['username'] = request.form['username']
      return redirect(url_for('index'))
   return '''
   <form action = "login" method = "post">
      <p><input type = "text" name = "username"/></p>
      <input type="submit" value="Submit" />
   </form>	
   '''
@app.route('/logout')
def logout():
   # remove the username from the session if it is there
   session.pop('username', None)
   return redirect(url_for('index'))


if __name__ == "__main__":
    app.run(debug=True)

```

# 原理

![session相关的关系图](https://www.programmersought.com/images/381/476ed09a2901f6c14a7ecdf5556af475.JPEG)

其活动图如下：

![](https://img-blog.csdn.net/20180502093913344?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMyMTA2MjA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)



# redis作为缓存
请求到来之前，获取session的方式
请求到来之前通过RequestContex 获取session, 由下图看出，open_session 调用session_interface,而session_interface，是SecureCookieSessionInterface（）的对象。

而 SecureCookieSessionInterface（）,提供了open，和save的方法，所以，可以使用 RedisSessionInterface 替换 SecureCookieSessionInterface,关键就是在配置文件中设置 session_interface指向哪个类

![SecureCookieSessionInterface](https://images2018.cnblogs.com/blog/1255078/201804/1255078-20180429160951421-992427422.png)


```python
class RedisSessionInterface(SessionInterface):
    """Uses the Redis key-value store as a session backend.

    .. versionadded:: 0.2
        The `use_signer` parameter was added.

    :param redis: A ``redis.Redis`` instance.
    :param key_prefix: A prefix that is added to all Redis store keys.
    :param use_signer: Whether to sign the session id cookie or not.
    :param permanent: Whether to use permanent session or not.
    """
  def __init__(self, redis, key_prefix, use_signer=False, permanent=True):
```

引入方法
```python
from flask_session import RedisSessionInterface
from redis import Redis
app.session_interface = RedisSessionInterface(redis=Redis(host='127.0.0.1',port=6379),key_prefix='a')
# use_signer 默认不对session-id加密
```



## 用redis查看
可以通过./redis-cli命令查看
常用命令：
1） 查看keys个数
keys * // 查看所有keys
keys prefix_* // 查看前缀为”prefix_”的所有keys
get 查看某个key的内容

2） 清空数据库
flushdb // 清除当前数据库的所有keys
flushall // 清除所有数据库的所有keys

## 使用docker-compose启动的redis

docker-compose exec  redis redis-cli

# 参考

- [Redis] redis-cli 命令总结](https://segmentfault.com/a/1190000022713251)
- [Python操作Redis数据库](https://www.cnblogs.com/cnkai/p/7642787.html)
- [redis session 超时时间_程序员必懂的Redis技术实战](https://blog.csdn.net/weixin_32673065/article/details/113627266)
- [flask_session——RedisSessionInterface 使用](https://www.cnblogs.com/huyangblog/p/8971353.html): RedisSessionInterface源码讲解
- [[Python] 用Session()优化requests的性能](https://zhuanlan.zhihu.com/p/114283369):客户端做session重用，与cookie无关
- [Flask 偏函数、g对象、flask-session、数据库连接池、信号、自制命令、flask-admin](https://mp.weixin.qq.com/s/cdbW1NOr7VN_Iw6cEb_z6w)