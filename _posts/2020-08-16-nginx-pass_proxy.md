---
layout: post
categories: Linux 系统管理
---

**采用容器集群化部署后，nginx作为路由控制，对于proxy_pass参数的灵活应用就相当于掌握了一把瑞士军刀**

# 转发路径拼接

在nginx中配置proxy_pass代理转发时，如果在proxy_pass后面的url加/，表示绝对根路径；如果没有/，表示相对路径，把匹配的路径部分也给代理走。



例如用 http://192.168.1.1/proxy/test.html 进行访问。

第一种：
```
location /proxy/ {
    proxy_pass http://127.0.0.1/aaa/;
}
```
代理到URL：http://127.0.0.1/aaa/test.html


第二种（相对于第一种，最后少一个 / ）
```
location /proxy/ {
    proxy_pass http://127.0.0.1/aaa;
}
```
代理到URL：http://127.0.0.1/aaa/proxy/test.html


# 路径正则
location 支持的语法 location [=|~|~*|^~|@] pattern { ... }，乍一看还挺复杂的，来逐个看一下。

## 「=」 修饰符：要求路径完全匹配
例如：location = /abcd {..}

- http://website.com/abcd匹配
- http://website.com/ABCD可能会匹配 ，也可以不匹配，取决于操作系统的文件系统是否大小写敏感（case-sensitive）。ps: Mac 默认是大小写不敏感的
- http://website.com/abcd?param1&param2匹配，忽略 querystring
- http://website.com/abcd/不匹配，带有结尾的/
- http://website.com/abcde不匹配

## 「~」修饰符：区分大小写的正则匹配
例如: location ~ ^/abcd$ {}
^/abcd$这个正则表达式表示字符串必须以/开始，以$结束，中间必须是abcd

- http://website.com/abcd匹配（完全匹配）
- http://website.com/ABCD不匹配，大小写敏感
- http://website.com/abcd?param1&param2匹配，忽略 querystring
- http://website.com/abcd/不匹配，不能匹配正则表达式
- http://website.com/abcde不匹配，不能匹配正则表达式

## 「~*」不区分大小写的正则匹配
例如: location ~ ^/abcd$ {}

- http://website.com/abcd匹配 (完全匹配)
- http://website.com/ABCD匹配 (大小写不敏感)
- http://website.com/abcd?param1&param2匹配
- http://website.com/abcd/ 不匹配，不能匹配正则表达式
- http://website.com/abcde 不匹配，不能匹配正则表达式

## 匹配的顺序

当有多条 location 规则时，nginx 有一套比较复杂的规则，优先级如下：

- 精确匹配 =
- 前缀匹配 ^~（立刻停止后续的正则搜索）
- 按文件中顺序的正则匹配 ~或~*
- 匹配不带任何修饰的前缀匹配。


参考文章: [彻底弄懂 Nginx location 匹配](https://juejin.im/post/6844903849166110733)
配置完成后，可以通过在线工具进行正则表达式的：https://www.regextester.com/94055

# 正则表达+proxy_pass的用法

https://stackoverflow.com/questions/53353572/proxy-pass-cannot-have-uri-part-in-location-given-by-regular-expression

```
location ~ ^/([A-Za-z0-9]+) {
    proxy_pass http://api/urlshortener/v1/$1;
}
```

# nginx中try_files（Vue的history模式）

```
location / {
            try_files $uri $uri/ /index.html;
}
```

当用户请求 http://localhost/example 时，这里的 $uri 就是 /example

- 硬盘里尝试找这个文件。如果存在名为 /$root/example
- 没有找到，看有没有名为 /$root/example/ 的目录
- 没有找到，会fall back 到 try_files 的最后一个选项 /index.html发起一个内部 “子请求”，也就是相当于 nginx 发起一个 HTTP 请求到 http://localhost/index.html

参考：

- [每天一点点之vue框架开发 - History 模式下线上路由报404错误](https://www.cnblogs.com/cap-rq/p/10148957.html)
- [nginx中try_files](https://www.cnblogs.com/boundless-sky/p/9459775.html)