---
layout: post
categories: Linux 系统管理 网络 安全
---

- 通过letsencrypt获取免费https证书
- 使用CDN(阿里云或七牛云)，支持https协议
- 免费证书3个月有效期，每3个月要重新获取证书，并更新到CDN配置

# 通过letsencrypt获取免费https证书

## nginx+ubuntu环境准备

访问地址：https://certbot.eff.org/lets-encrypt/ubuntuxenial-nginx

前置条件：
- http网站必须可以访问，并且是80端口
  - 意味着如果配置CDN的话，必须临时把CDN域名的CNAME改为A方式
- 必须已经安装nginx，并且执行sudo命令

## 如果配置了CDN，需要临时把CDN域名的CNAME改为A方式

## 执行certbot命令
执行`sudo certbot --nginx`命令如下：
```log
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Plugins selected: Authenticator nginx, Installer nginx

Which names would you like to activate HTTPS for?
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
1: lichangzhen.top
2: blog.lichangzhen.top
3: img.lichangzhen.top
4: www.lichangzhen.top
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Select the appropriate numbers separated by commas and/or spaces, or leave input
blank to select all options shown (Enter 'c' to cancel): 1 2 3 4
Obtaining a new certificate
Performing the following challenges:
http-01 challenge for lichangzhen.top
Waiting for verification...
Cleaning up challenges
Deploying Certificate to VirtualHost /etc/nginx/conf.d/blog.conf
Deploying Certificate to VirtualHost /etc/nginx/conf.d/blog.conf
Deploying Certificate to VirtualHost /etc/nginx/conf.d/blog.conf
Deploying Certificate to VirtualHost /etc/nginx/conf.d/blog.conf

Please choose whether or not to redirect HTTP traffic to HTTPS, removing HTTP access.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
1: No redirect - Make no further changes to the webserver configuration.
2: Redirect - Make all requests redirect to secure HTTPS access. Choose this for
new sites, or if you're confident your site works on HTTPS. You can undo this
change by editing your web server's configuration.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Select the appropriate number [1-2] then [enter] (press 'c' to cancel): 1

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Congratulations! You have successfully enabled https://img.lichangzhen.top,
https://www.lichangzhen.top, https://lichangzhen.top, and
https://blog.lichangzhen.top

You should test your configuration at:
https://www.ssllabs.com/ssltest/analyze.html?d=img.lichangzhen.top
https://www.ssllabs.com/ssltest/analyze.html?d=www.lichangzhen.top
https://www.ssllabs.com/ssltest/analyze.html?d=lichangzhen.top
https://www.ssllabs.com/ssltest/analyze.html?d=blog.lichangzhen.top
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/img.lichangzhen.top/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/img.lichangzhen.top/privkey.pem
   Your cert will expire on 2019-11-28. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot again
   with the "certonly" option. To non-interactively renew *all* of
   your certificates, run "certbot renew"
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
```

可以看到/etc/nginx/conf.d/blog.conf中增加了下面几行配置
```ini
{
    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/img.lichangzhen.top/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/img.lichangzhen.top/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
}
```
进一步打开/etc/letsencrypt/live/img.lichangzhen.top文件夹，发现下面四个文件：

| 文件名  | 文件作用   |
| ---- | ---- | ---- |
|cert.pem   |  服务端证书|
|chain.pem| 浏览器需要的所有证书但不包括服务端证书，比如根证书和中间证书|
|fullchain.pem| 包括了cert.pem和chain.pem的内容|
|privkey.pem|   证书的私钥|

而官方Readme的说明如下：
```
`privkey.pem`  : the private key for your certificate.
`fullchain.pem`: the certificate file used in most server software.
`chain.pem`    : used for OCSP stapling in Nginx >=1.3.7.
`cert.pem`     : will break many server configurations, and should not be used
                 without reading further documentation (see link below).
```


此时就可以用 https://blog.lichangzhen.top 访问了

# CDN配置https协议(已经验证阿里云和七牛云)

在SSL证书管理的地方,找到”上传自有证书“：

- 证书备注名： 自己命名，便于记忆即可
- 证书内容 ( PEM格式 )： 把 fullchain.pem的内容粘贴过来即可，例如/etc/letsencrypt/live/img.lichangzhen.top/fullchain.pem
- 证书私钥 ( PEM格式 )： 把 privkey.pem的内容粘贴过来即可，例如/etc/letsencrypt/live/img.lichangzhen.top/privkey.pem

![ssl证书管理编辑页面](http://img.lichangzhen.top/2019/https-ssl证书管理.jpg)



# 每3个月更新证书

配置CDN时把域名配置为CNAME方式，指向了阿里云地址，分为4步走

- 更新证书前，需要临时把域名改为A方式(临时取消CDN缓存,但服务能正常访问)
- 更新https证书，验证ok（此时没有CDN）
- 再把域名改为CNAME方式，指向阿里云的CDN服务器(CDN缓存重新生效，此时用的还是旧证书)
- 更新阿里云CDN的https配置(开始使用新证书)



# 参考

- [证书格式区别](https://wenku.baidu.com/view/6bdc7853336c1eb91a375dc5.html)
- [你的网站HTTPS了吗 | Let’s Encrypt](https://www.jianshu.com/p/817f2bec0fc5)