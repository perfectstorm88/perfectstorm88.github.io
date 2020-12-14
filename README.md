

fork from https://github.com/mzlogin/mzlogin.github.io


启动方式:
```
export SSL_CERT_FILE=~/Downloads/cacert.pem && bundle exec jekyll serve
```

# 启动问题：
1、Liquid Exception: SSL_connect returned=1 errno=0 state=error: certificate verify failed (unable to get local issuer certificate) in /_layouts/page.html

```
Download https://curl.haxx.se/ca/cacert.pem
Add a environment variable SSL_CERT_FILE values /path/to/cacert.pem
export SSL_CERT_FILE=~/Downloads/cacert.pem
```

