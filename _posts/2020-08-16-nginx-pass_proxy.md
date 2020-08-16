---
layout: post
categories: Linux 系统管理
---

**采用容器集群化部署后，nginx作为路由控制，对于proxy_pass参数的灵活应用就相当于掌握了一把瑞士军刀**
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

