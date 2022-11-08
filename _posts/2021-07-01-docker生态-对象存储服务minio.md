# 介绍和原理
业内较为主流的开源存储框架MinIO、Ceph、SeaweedFS, 在github上MinIO的**star是最多的，远超其它框架,大众的选择没错的**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210701161527394.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpY2hhbmd6aGVuMjAwOA==,size_16,color_FFFFFF,t_70)

根据[minio官方](https://min.io/)介绍，可以应用于下列场景

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210701162239794.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpY2hhbmd6aGVuMjAwOA==,size_16,color_FFFFFF,t_70)



MinIO完全兼容S3标准接口，客户端和服务端之间通过http/https进行通信。MinIO提供客户端mc（MinIO Client）以支持UNIX命令，同时支持多语言的客户端SDK。![在这里插入图片描述](https://img-blog.csdnimg.cn/2021070116163065.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpY2hhbmd6aGVuMjAwOA==,size_16,color_FFFFFF,t_70)

## S3是啥？
S3是Simple Storage Service的缩写，即简单存储服务。亚马逊的名词缩写也都遵循这个习惯，例如Elastic Compute Cloud缩写为EC2等等。

# docker方式安装说明

特别说明：通过命令行方式安装使用

[MinIO快速入门指南](https://docs.min.io/cn/minio-quickstart-guide.html)

```bash
mkdir /home/minio-data
docker run -d --name minio -p 9000:9000 -v /home/minio-data:/data  minio/minio server /data
docker logs -f minio #  可以看到默认的'minioadmin:minioadmin’
```

# 附: docker-compose样例

```dockerfile
  minio:
    image: minio/minio:RELEASE.2021-05-22T02-34-39Z
    volumes:
      - ./back/:/data
    ports:
      - "8039:9000"
    environment:
      MINIO_ROOT_USER: admin
      MINIO_ROOT_PASSWORD: mypasswd
    command: server /data
```

# 通过nginx作为代理

minio构建的服务进行访问，**以/minio为前缀的,前端代码里是绝对路径**，用nginx做代理时，只支持`/minio/` 这个路径

```nginx
location /minio/ {
    proxy_pass http://10.17.157.168:8039/minio/;  # 
    client_max_body_size 500m;  # 
}
```

或者

```nginx
    location /minio {
      proxy_pass http://minio:9000;
      client_max_body_size 500m;  # 
    }
```

**注意要修改client_max_body_size的值**，否则会出现413 Request Entity Too Large错误

# 参考
- [基于 MinIO 对象存储框架的短视频点播平台设计](https://www.163.com/dy/article/GCFV3N2H0511FQO9.html)
- [对象存储服务分析数据](https://www.g2.com/categories/object-storage):Amazon Simple Storage Service (S3)最火
- [4 Open Source Object Storage Platforms for 2021](https://betterprogramming.pub/4-open-source-object-storage-platforms-for-2021-ceeaceb7e273)
- [断点续传方案：原理和想法可以的，部署太麻烦了](https://github.com/yuyuanshifu/minio-breakpoint-upload):本想部署的，太麻烦，而且后期不好维护，最后通过修改client_max_body_size参数凑合用


