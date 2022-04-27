## 安装PHP

https://blog.csdn.net/qq_25194685/article/details/89742199

### 检索

➜  util docker search php

### 拉取php7.0

➜  util docker pull php:7.0-fpm
7.0-fpm: Pulling from library/php
177e7ef0df69: Pull complete
4c66f054f322: Pull complete
b7221dc910cf: Pull complete
71b3dbdbc6f4: Pull complete
Digest: sha256:8e8644c0fd61424939fff072ca617469550bbb659158301a2dcadf334746a1c2
Status: Downloaded newer image for php:7.0-fpm
docker.io/library/php:7.0-fpm

### 创建容器并运行

➜  webphp docker run --name php7 --link mysql:mysql -d -p 9000:9000 -v /Users/borgxiao/Documents/work/webphp:/www/wwwroot php:7.0-fpm
23b11a0d1c62a737c79ad2d523dda6141ff48defd97802eabd5738f6eceffcf3



➜  docker docker run --name php7 -d --net=host -p 9000:9000 -v /Users/borgxiao/Documents/work/webphp:/www/wwwroot php:7.0-fpm
668e680d0832842a4d5bfa3cb48fd748d7849fc7d96d31618dd03ae8465136e3



方式2：共享网络

➜  etc docker run --name php7 -d -p 9000:9000 --net=host -v /Users/borgxiao/Documents/work/webphp:/www/wwwroot php:7.0-fpm
WARNING: Published ports are discarded when using host network mode
81fbbf836d8b2898fd5470cd4607a462fe50f7e9082d08769eb29b8c3273cdcc

## 安装nginx

➜  webphp docker pull nginx
Using default tag: latest

➜  docker docker run --name nginx --link php7:php --link mysql:mysql -p 80:80 -v /Users/borgxiao/Documents/work/webphp:/www/wwwroot -v /Users/borgxiao/Documents/work/docker/nginx:/etc/nginx/conf.d -d nginx:latest
fb6fdd1f2dcf33e76a2f420629985a49801caac72ba4d76a90e7e465ce940a78

注：--link是为了连接在一起 ，这里把配置文件也加载进去，即不再需要复制和粘贴



➜  docker docker run --name nginx -p 80:80 --net=host -v /Users/borgxiao/Documents/work/webphp:/www/wwwroot -v /Users/borgxiao/Documents/work/docker/nginx:/etc/nginx/conf.d -d nginx:latest
56cf760acb7b5159dec27ff93c87a45b6a7ea2a587ebb2179dbef05d3cdc44a1



也可以把mysql和redis连接进入：docker run --name nginx --link mysql:db --link redis:redis --link php7:php -p 80:80 -v /home/www:/www/wwwroot -d nginx

### 修改nginx配置

/etc/nginx/conf.d/

#### 复制配置到本地：

docker cp nginx:/etc/nginx/conf.d/default.conf default.conf 

➜  webphp docker cp nginx:/etc/nginx/conf.d/default.conf default.conf
➜  webphp pwd
/Users/borgxiao/Documents/work/webphp

```nginx
location ~ \.php$ {
        root           /www/wwwroot;
        fastcgi_pass   php:9000;#这里用php的地址
        fastcgi_index  index.php;
        #fastcgi_param  SCRIPT_FILENAME  scripts$fastcgi_script_name;
        #这里更改为$document_root不然运行不起来
        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
        include        fastcgi_params;
    }
```



#### 复制回去

docker cp default.conf nginx:/etc/nginx/conf.d/default.conf



日志目录：

access_log  /var/log/nginx/access.log

## 启动和运行

启动容器
docker start nginx
关闭容器
docker stop nginx
重启容器
docker restart nginx
删除容器
docker rm nginx
进入容器
docker exec -it nginx /bin/bash
容器状态
docker status nginx

## 配置虚拟机

目前直接在/etc/nginx/conf.d/default.conf上加的，如果细一点可以弄一个专门目录，然后不同的域名加载不同的文件

```nginx
server {
    listen       80;
    server_name  www.haotian.com;

    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        root   /www/wwwroot/cao/new_version/php/public;
        index  index.php index.html index.htm;
        #thinkPHP
        if (!-e $request_filename){
            rewrite  ^(.*)$  /index.php?s=$1  last;   break;
        }
    }
    #兼容旧版接口,追加
    rewrite ^/hm_ucenter/web/index.php$ /api/index last;

    #error_page  404              /404.html;
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
    location ~ \.php$ {
        root           /www/wwwroot/cao/new_version/php/public;
        fastcgi_pass   php:9000;#这里用php的地址
        fastcgi_index  index.php;
        #fastcgi_param  SCRIPT_FILENAME  scripts$fastcgi_script_name;
        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
        include        fastcgi_params;
    }
}
```

## 安装PHP模块

```shell
# docker-php-ext-install pdo_mysql
```

## docker网络问题

docker run 创建 Docker 容器时，可以用 –net 选项指定容器的网络模式，Docker 有以下 4 种网络模式：

host 模式，使用 –net=host 指定。
container 模式，使用 –net=container:NAME_or_ID 指定。
none 模式，使用 –net=none 指定。
b**ridge 模式，使用 –net=bridge 指定，默认设置**

### host模式：

和主机共享网络

### container

这个模式指定新创建的容器和已经存在的一个容器共享一个 Network Namespace，而不是和宿主机共享

### none

Docker 容器拥有自己的 Network Namespace，但是，并不为 Docker容器进行任何网络配置。也就是说，这个 Docker 容器没有网卡、IP、路由等信息。

### bridge

bridge 模式是 Docker 默认的网络设置，此模式会为每一个容器分配 Network Namespace、设置 IP 等，并将一个主机上的 Docker 容器连接到一个虚拟网桥上。当 Docker server 启动时，会在主机上创建一个名为 docker0 的虚拟网桥

## 集群模式

```shell
#绑定在3307(外部) 并且映射数据库目录到本地
➜  docker docker run -p 3307:3306 --name mysql -v /Users/borgxiao/Documents/work/docker/mysql:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.6.51
ba699706831cd946e6d5f2f4597e2be986d44e61ea02f01fe03c69e8827b5911

➜  webphp docker inspect mysql | grep IPAddress
            "SecondaryIPAddresses": null,
            "IPAddress": "172.17.0.2",
                    "IPAddress": "172.17.0.2",
```





因为mac不支持host且容器和主机间无法通信，所以全部重新部署

```shell
➜  ~ docker run -p 6380:6379 --name redis4 -d redis:4
e68ccc2af77812fb695f26fe86737ec4dc6f74ee79bcdd7272a2366ebb5818d6

➜  docker docker run -p 3307:3306 --name mysql5 -v /Users/borgxiao/Documents/work/docker/mysql:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.6.51

➜  docker docker run --name php7 --link mysql5:mysql5 --link redis4:redis4 -d -p 9000:9000 -v /Users/borgxiao/Documents/work/webphp:/www/wwwroot php:7.0-fpm

# docker-php-ext-install pdo_mysql
Configuring for:
PHP Api Version:         20151012
Zend Module Api No:      20151012
Zend Extension Api No:   320151012


docker run --name tengine2 --link php7:php --link mysql5:mysql5 --link redis4:redis4 -p 80:80 -v /Users/borgxiao/Documents/work/webphp:/www/wwwroot -v /Users/borgxiao/Documents/work/docker/nginx:/etc/nginx/conf.d -d axizdkr/tengine:2.2.3


```

注：php安装模块比较耗时，所以更换了一个包含常用扩展的镜像

https://hub.docker.com/r/wy373226722/php/tags

```shell
➜  docker docker run --name php73 --link mysql5:mysql5 --link redis4:redis4 -d -p 9000:9000 -v /Users/borgxiao/Documents/work/webphp:/www/wwwroot wy373226722/php
9fb91625a198f538b0428555c72e53f71423f259a402a5e68fd4d6e17d9ee27e

➜  docker docker run --name tnginx2 --link php73:php -p 80:80 -v /Users/borgxiao/Documents/work/webphp:/www/wwwroot -v /Users/borgxiao/Documents/work/docker/nginx:/etc/nginx/conf.d -d axizdkr/tengine:2.2.3
88ebae62ac65a25485c1a9a3b6cdafc3c70e0f32d0a6a4a3665ab79d7e0c83f6

```

注：这样看 只需要php链接各种库即可，其他的不用

