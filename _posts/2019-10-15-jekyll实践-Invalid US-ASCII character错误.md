---
layout: post
categories: Jekyll web
---

在使用jekyll时，本来还是正常的，突然出现下面报错,这个错误其实挺简单，但是却前前后后花费我4、5个小时的时间，特此记录我浪费的时间：

# 问题说明

```
# bundle exec jekyll serve
Configuration file: /data/lcz/blog/_config.yml
            Source: /data/lcz/blog
       Destination: /data/lcz/blog/_site
 Incremental build: disabled. Enable with --incremental
      Generating...
       Jekyll Feed: Generating feed for posts
   GitHub Metadata: No GitHub API authentication could be found. Some fields may be missing or have incorrect data.
  Conversion error: Jekyll::Converters::Scss encountered an error while converting 'assets/css/style.scss':
                    Invalid US-ASCII character "\xE2" on line 5
jekyll 3.8.5 | Error:  Invalid US-ASCII character "\xE2" on line 5
```

# 解决方法

是环境变量的问题，执行`export LC_ALL="en_US.UTF-8"`就可以了

# 解决路径

1. 一开始在网上搜索的关键词是，Conversion error: Jekyll::Converters::Scss，网上答案很对，有些是告诉在*.scss文件中增加 uft-8, 可是本地没有*.scss都是*.css文件（消耗时间1-2小时）

2. 以为是版本问题，重新安装，升级到jekyll 4.0版本，但是问题工程的Gemfile限制，github-pages 只能运行在3.8.5下，只能通过bundle exec 执行jekyll serve(消耗时间2小时)

3. 在jekyll的github工程上，搜索关键字，一开始使用“Conversion error: Jekyll::Converters::Scss”，一堆信息，没发现有价值的issue（消耗时间0.5小时）
  
4. 然后搜索US-ASCII，才找到有用信息“I found that my local code was not utf-8，so I modified it to UTF-8. It's working”

5. 搜索如何查看和修改ubuntu的字体,并且发现本地的ubuntu环境会报错“emanpath: can't set the locale; make sure $LC_* and $LANG are correct”
   1. 通过增加 LC_ALL=zh_CN.UTF-8就可以了解决了


# 参考：

- [jekyll 3.4.0 | Error: Invalid US-ASCII character "\xE2" on line 10 #6171](https://github.com/jekyll/jekyll/issues/6171)
- [Full switch locale: Ubuntu server installed with no locales, how to enable locales systemwide?](https://askubuntu.com/questions/298971/full-switch-locale-ubuntu-server-installed-with-no-locales-how-to-enable-local)

-[Linux下使用locale命令设置语言环境](https://www.cnblogs.com/dolphi/p/3622570.html):
  + 在Linux中通过locale来设置程序运行的不同语言环境
  + 


