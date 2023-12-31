# Lsky 搭建图床

本文介绍使用开源项目 Lsky 在 vps 上自建一个图床。

为什么要自建图床，其实存储图片的方式有很多种，例如

可以使用免费的 `Github + JsDelivr` ，但可能访问速度会有些影响

也可以使用国内服务器厂商的对象存储，例如腾讯云 COS ，阿里云 OSS等，付费也非常的便宜，一年就几十块钱。但是这类存储服务还是有各种限制的，比如大图片储存会有限制，不定期的审核机制，不可抗力的封禁，被恶意刷流量等等。

自建的好处就是依托于自有的服务器，方便管理，随时备份迁移，不怕丢失。

**当然如果你没有上述的那些顾虑，那直接使用大厂的对象存储，方便省事。**



## 搭建教程

这是 Lsky 的官网，https://docs.lsky.pro，

这里使用 docker 的方式来部署，这里要看另一个关于该项目的docker部署项目 https://github.com/HalcyonAzure/lsky-pro-docker

按照步骤如下：

### 1、新建一个该项目的目录

```sh
mkdir lsky-docker
```

### 2、新建 docker-compose 文件

进入到 lsky-docker 创建一个 docker-compose 文件

```sh
cd lsky-docker
mkdir docker-compose
```

```yml
version: '3'
services:
  lskypro:
    image: halcyonazure/lsky-pro-docker:latest
    restart: unless-stopped
    hostname: lskypro
    container_name: lskypro
    environment:
      - WEB_PORT=8089
    volumes:
      - $PWD/web:/var/www/html/
    ports:
      - "8089:8089"
    networks:
      - lsky-net

networks:
  lsky-net: {}
```

这里我没有直接在 docker 安装mysql数据库，因为我的本地服务器已经安装了 mysql，不想再起一个服务，占用内存。

如果你就是想用 docker 安装数据库，就直接用下面的 docker-compose 配置，然后跳过下面的新建数据库步骤即可

```yml
version: '3'
services:
  lskypro:
    image: halcyonazure/lsky-pro-docker:latest
    restart: unless-stopped
    hostname: lskypro
    container_name: lskypro
    environment:
      - WEB_PORT=8089
    volumes:
      - $PWD/web:/var/www/html/
    ports:
      - "9080:8089"
    networks:
      - lsky-net

  # 注：arm64的无法使用该镜像，请选择sqlite或自建数据库
  mysql-lsky:
    image: mysql:8.2.0
    restart: unless-stopped
    # 主机名，可作为"数据库连接地址"
    hostname: mysql-lsky
    # 容器名称
    container_name: mysql-lsky
    # 修改加密规则
    command: --default-authentication-plugin=mysql_native_password
    volumes:
      - $PWD/mysql/data:/var/lib/mysql
      - $PWD/mysql/conf:/etc/mysql
      - $PWD/mysql/log:/var/log/mysql
    environment:
      MYSQL_ROOT_PASSWORD: lAsWjb6rzSzENUYg # 数据库root用户密码，自行修改
      MYSQL_DATABASE: lsky-data # 可作为"数据库名称/路径"
    networks:
      - lsky-net

networks:
  lsky-net: {}
```



### 3、新建数据库

我用的是 1panel ，这里直接新建一个 lsky 的数据库即可

![image-20231227113558053](https://photo.daydaysup.com/st/2023/12/27/658c2ccad6712.png)



### 4、安装配置

输入地址 http://ip:8089/install 就可以进入安装界面了

![image-20231227114547098](https://photo.daydaysup.com/st/2023/12/27/658c2cca5056f.png)



点击下一步，进入到数据库的配置界面

- 数据库连接地址：服务器的 IP
- 数据库名称：创建数据库时的名称，lsky
- 数据库用户名：创建数据库时的用户名， lsky
- 数据库密码：创建数据库时的密码
- 管理员账号邮箱密码：这个是用来登陆控制台，自己记住就行



![image-20231227115407421](https://photo.daydaysup.com/st/2023/12/27/658c2cce24160.png)



## https 配置

如果需要给自己的项目配上 https ，增加安全性，可以参考之前的项目

首先假设，我这里通过 photo.daydaysup.com，那你需要在 cloudflare DNS里添加一条A记录，值为photo，指向当前服务器的IP

![image-20231227124541835](https://photo.daydaysup.com/st/2023/12/27/658c2cd223349.png)

由于之前已经配置过了daydaysup.com 这个博客域名的访问，只需要在之前的配置里，在最后面加上这一段重启nginx即可

新加的 nginx 配置如下：https 证书使用的泛域名，所以不需要重新申请证书。

注意了，如果访问http://photo.daydaysup.com的时候打开的页面是 daydaysup.com，清下DNS缓存之类的

```yml
server {
    listen 80;
    server_name photo.daydaysup.com;
    return 301 https://$server_name$request_uri;

    location / {
        proxy_pass http://123.456.789.12:8089;  # 使用新项目的端口
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

server {
    listen 443 ssl;
    server_name photo.daydaysup.com;

    ssl_certificate ssl/fullchain.pem;
    ssl_certificate_key ssl/cert.key;
    # 下面这些可要可不要
    ssl_session_timeout 5m;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;

    location / {
        proxy_pass http://123.456.789.12:8089;  # 使用新项目的端口
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

