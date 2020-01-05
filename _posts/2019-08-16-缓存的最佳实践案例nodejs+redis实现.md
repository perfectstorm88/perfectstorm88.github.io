---
layout: post
categories: redis 设计 nodejs
---
接上一篇《缓存的最佳实践案例》，展示了用nodejs+redis实现的过程和代码
代码主要是实现三种函数，再加init函数

  - init: 初始化
  - set：设置缓存
  - get: 获取缓存
  - reset: 清除缓存

驱动使用的[ioredis](https://github.com/luin/ioredis)这个库，相比[node_redis](https://github.com/NodeRedis/node_redis)这个很老的库，支持了await操作,当然了ioredis还有其它好处。

# 1.初始化
通过config读取redis配置
```js
//初始化redis
if(config.redis){//"redis://:lingxi88@47.99.88.111:6379"
    var Redis = require("ioredis");
    var redis = new Redis(config.redis);
    // if you'd like to select database 3, instead of 0 (default), call
    // client.select(3, function() { /* ... */ });
    redis.on("error", function (err) {
          console.log("redis Error " + err);
    });
    GlobalCache.redis = redis;
}
```
# set设置缓存
set时要设置超时时间，默认2周
```js
static async setGlobalBuf(redis, key1, key2, obj) {
    let key = `${key1}:${key2}`;
    let content = JSON.stringify(obj);
    logger.warn(`【cache】redis设置缓存：${key1},${key2}`)
    await redis.set(key, content);
    redis.expire(redis,EXPIRE_TIME);//设置过期时间
}
```

# 2.get缓存
每次get成功时，刷新过期时间
```js
static async getGlobalBuf(redis, key1, key2) {

    let key = `${key1}:${key2}`;
    let ret = await redis.get(key);

    if (!ret) {
        logger.error(`【cache】redis缓存不存在：${key1},${key2}`)
        return [false, null];
    }
    redis.expire(redis,EXPIRE_TIME);//每次get时，刷新过期时间
    let retObj = JSON.parseWithDate(ret);
    logger.warn(`【cache】redis重用缓存：${key1},${key2}`)
    return [true, retObj];
}
```
# 3.reset缓存
```js
static async resetGlobalBuf(redis, key1, key2) {
    logger.error(`【cache】redis清空缓存：${key1},${key2}`)
    let key = `${key1}:${key2}`;
    await redis.unlink(key);
}
```

## 4.reset缓存（通过前缀方式）
```js
static async resetGlobalBufByPrefix(redis, key1, key2) {
    logger.error(`【cache】redis清空缓存：${key1},${key2}`)
    let keyMatch = `${key1}:${key2}*`;
    return await deleteCacheByMatch(redis, keyMatch);
}
```
其中deleteCacheByMatch是通过高效的scanStream和实现，封装为Promise。
相关讨论参见
```js
var deleteCacheByMatch = (redis, keyMatch) => {
    return new Promise((resolve, reject) => {
        var stream = redis.scanStream({
            // only returns keys following the pattern of `user:*`
            match: keyMatch,
            // returns approximately 100 elements per call
            count: 100
        });
        var count = 0;
        stream.on('data', function (resultKeys) {
            count += resultKeys.length;
            if (resultKeys.length) {
                logger.error(`【cache】redis清空缓存：${JSON.stringify(resultKeys)}`);
                redis.unlink(resultKeys);
            }
        });
        stream.on("end", function () {
            // console.log("all keys have been visited", count);
            return resolve();
        });
    })
}
```

# 扩展：由于缓存保存的是json对象，对于Date类型反序列化还要做些处理
这儿对JSON方法进行扩展，在get函数，使用的`JSON.parseWithDate(ret)`，而不是`JSON.parse(ret)`

```js
var reISO = /^(\d{4})-(\d{2})-(\d{2})T(\d{2}):(\d{2}):(\d{2}(?:\.\d*))(?:Z|(\+|-)([\d|:]*))?$/;
var reMsAjax = /^\/Date\((d|-|.*)\)[\/|\\]$/;
/// <summary>
/// set this if you want MS Ajax Dates parsed
/// before calling any of the other functions
/// </summary>
JSON.parseMsAjaxDate = false;

JSON.useDateParser = function (reset) {
    /// <summary>
    /// Globally enables JSON date parsing for JSON.parse().
    /// replaces the default JSON parser with parse plus dateParser extension 
    /// </summary>    
    /// <param name="reset" type="bool">when set restores the original JSON.parse() function</param>

    // if any parameter is passed reset
    if (typeof reset != "undefined") {
        if (JSON._parseSaved) {
            JSON.parse = JSON._parseSaved;
            JSON._parseSaved = null;
        }
    } else {
        if (!JSON.parseSaved) {
            JSON._parseSaved = JSON.parse;
            JSON.parse = JSON.parseWithDate;
        }
    }
};

JSON.dateParser = function (key, value) {
    /// <summary>
    /// Globally enables JSON date parsing for JSON.parse().
    /// Replaces the default JSON.parse() method and adds
    /// the datePaser() extension to the processing chain.
    /// </summary>    
    /// <param name="key" type="string">property name that is parsed</param>
    /// <param name="value" type="any">property value</param>
    /// <returns type="date">returns date or the original value if not a date string</returns>
    if (typeof value === 'string') {
        var a = reISO.exec(value);
        if (a)
            return new Date(value);

        if (!JSON.parseMsAjaxDate)
            return value;

        a = reMsAjax.exec(value);
        if (a) {
            var b = a[1].split(/[-+,.]/);
            return new Date(b[0] ? +b[0] : 0 - +b[1]);
        }
    }
    return value;
};

JSON.parseWithDate = function (json) {
    /// <summary>
    /// Wrapper around the JSON.parse() function that adds a date
    /// filtering extension. Returns all dates as real JavaScript dates.
    /// </summary>    
    /// <param name="json" type="string">JSON to be parsed</param>
    /// <returns type="any">parsed value or object</returns>
    var parse = JSON._parseSaved ? JSON._parseSaved : JSON.parse;
    try {
        var res = parse(json, JSON.dateParser);
        return res;
    } catch (e) {
        // orignal error thrown has no error message so rethrow with message
        throw new Error("JSON content could not be parsed");
    }
};

JSON.dateStringToDate = function (dtString, nullDateVal) {
    /// <summary>
    /// Converts a JSON ISO or MSAJAX date or real date a date value.
    /// Supports both JSON encoded dates or plain date formatted strings
    /// (without the JSON string quotes).
    /// If you pass a date the date is returned as is. If you pass null
    /// null or the nullDateVal is returned.
    /// </summary>    
    /// <param name="dtString" type="var">Date String in ISO or MSAJAX format</param>
    /// <param name="nullDateVal" type="var">value to return if date can't be parsed</param>
    /// <returns type="date">date or the nullDateVal (null by default)</returns> 
    if (!nullDateVal)
        nullDateVal = null;

    if (!dtString)
        return nullDateVal; // empty

    if (dtString.getTime)
        return dtString; // already a date

    if (dtString[0] === '"' || dtString[0] === "'")
        // strip off JSON quotes
        dtString = dtString.substr(1, dtString.length - 2);

    var a = reISO.exec(dtString);
    if (a)
        return new Date(dtString);

    if (!JSON.parseMsAjaxDate)
        return nullDateVal;

    a = reMsAjax.exec(dtString);
    if (a) {
        var b = a[1].split(/[-,.]/);
        return new Date(+b[0]);
    }
    return nullDateVal;
};
```
# 参考：
- [缓存的最佳实践案例](http://blog.lichangzhen.top//2019/08/11/%E7%BC%93%E5%AD%98%E7%9A%84%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5%E6%A1%88%E4%BE%8B/)
- [JavaScript JSON Date Parsing and real Dates](https://weblog.west-wind.com/posts/2014/jan/06/javascript-json-date-parsing-and-real-dates)