---
layout: post
categories: nodejs
---

# 问题说明
用mocha做单元测试是正常的，但是用Istanbul（nyc）时，却奇怪报下面错误

```
     MongoError: ReferenceError: cov_1vg4vh8ipw is not defined :
@:1:23

      at Connection.<anonymous> (node_modules/mongodb-core/lib/connection/pool.js:443:61)
      at processMessage (node_modules/mongodb-core/lib/connection/connection.js:364:10)
      at Socket.<anonymous> (node_modules/mongodb-core/lib/connection/connection.js:533:15)
      at addChunk (_stream_readable.js:263:12)
      at readableAddChunk (_stream_readable.js:250:11)
      at Socket.Readable.push (_stream_readable.js:208:10)
      at TCP.onread (net.js:601:20)
```


# 解决方法

参考nyc官网上的issue[Promise rejected with: 'cov_2itgu9hdrb is not defined](https://github.com/istanbuljs/nyc/issues/514), 找到引起错误的代码，通过/* istanbul ignore next */注释掉就可以了


对应的应用程序代码大概为：
```js
    async statTotal(tableName, query) {
        let collection = this.getCollection(tableName);
        let schema = await this.getSchema(tableName);
        query = CastHelper.castQuery(schema, query);
        let funcBody = "var ret={_count:1};\n"
        for (let k in schema.obj) {
            if (k == '序号') {
                continue;
            }
            let type = schema.obj[k].type||schema.obj[k];
            if (type == Number) {
                funcBody += `if(typeof this['${k}'] == 'number'){ \n`
                funcBody += `    ret['${k}']=this['${k}']; \n`
                funcBody += `}else{ \n`
                funcBody += `    ret['${k}']=0; \n`
                funcBody += `} \n`
            }
        }
        funcBody += '\n emit("",ret); \n'
        let map = new Function(funcBody);
        /* istanbul ignore next */
        let reduce = function (id, values) { var ret = values[0]; for (let i = 1; i < values.length; i++) { for (let k in ret) { ret[k] += values[i][k]; } } return ret; };
        let ret_db = await collection.mapReduce(map, reduce, { query: query, out: { inline: 1 } });
        return _.get(ret_db, '0.value', {});
    }
```

# 探寻路径

## nyc的include、exclude方法

初步判断可能是mongoose或者mongodb的问题，于是在package.json 中增加了 nyc 参数，发现问题依旧
```json
  "nyc": {
    "excludeNodeModules": true,
    "include":["src/db/*.js"]
  },
```

## 在nyc的issue中搜索“is not defined”，找到很多类式issue

例如[cov_1fanynun32 is not defined](https://github.com/istanbuljs/nyc/issues/629)

# 总结心得

以后碰到类似问题，可以在github的issue中，根据关键字搜索，如果搜不到再提issue向别人求助