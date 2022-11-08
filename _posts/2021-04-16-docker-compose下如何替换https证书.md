

# docker-compose下如何替换https证书

1. 先确认证书，正常情况下有两个文件一个.key结尾，另一个是.crt或者.pem结尾，先查看.crt或者.pem的内容

```bash
openssl x509 -in STAR_lcz_com.crt -text
```
2. 把证书上传到服务器部署目录(例如/home/build/my_deploy)的cert文件夹
3. 检查部署目录下的Dockerfile文件，确认cert目录下的证书拷贝到镜像中,例如

```dockerfile
FROM my_xxx_web
COPY nginx.conf  /etc/nginx/nginx.conf
COPY cert /etc/nginx  #  目录中不包含cert路径
# ADD cert /etc/nginx # 目录中包含cert路径
```
4. 修改部署目录下nginx.conf,如
```conf
server {
    listen 443 ssl;
    server_name x.abc.com; #这个域名必须是你申请ssl证书的时候绑定的域名
    ssl_certificate /etc/nginx/x.abc.com.pem; #SSL 证书文件路径，由证书签发机构提供
    ssl_certificate_key /etc/nginx/x.abc.com.key; #SSL 密钥文件路径，由证书签发机构提供
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4; #使用此加密套件。
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2; #使用该协议进行配置。
    ssl_prefer_server_ciphers on;
    location / {
        root html;
        index index.html index.htm;
    }
}
```

5. 重启web容器
```bash
dc up —build -d web
```
6. 检查logs,并用浏览器访问https地址，确认证书是否已经生效
```bash
dc logs -f --tail=20 web
```