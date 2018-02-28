## 使用场景:

- 简化配置，构建一次可以在任意环境运行。
- 提高开发效率，入职第一天搞环境？不需要，直接docker。标配环境
- 应用隔离，服务器整合
- 快速部署
- 自动化扩容
- 更适合微服务


镜像：类似于虚拟机镜像。容器：类似一个轻量级的沙箱。仓库：类似于代码仓库。

```shell
➜  ~ docker version
Client:
 Version:	17.12.0-ce
 API version:	1.35
 Go version:	go1.9.2
 Git commit:	c97c6d6
 Built:	Wed Dec 27 20:03:51 2017
 OS/Arch:	darwin/amd64

Server:
 Engine:
  Version:	17.12.0-ce
  API version:	1.35 (minimum version 1.12)
  Go version:	go1.9.2
  Git commit:	c97c6d6
  Built:	Wed Dec 27 20:12:29 2017
  OS/Arch:	linux/amd64
  Experimental:	true
```




## 镜像:

```Shell
#下载镜像文件到本地  docker pull name[:tag] 默认tag为latest
➜  ~ docker pull ubuntu:14.04
14.04: Pulling from library/ubuntu
#也可以直接从远端获取 docker pull hub.c.163.com/public/ubuntu:14.04

#查看本地已经下载的镜像列表 -a 查看所有信息 -q只输出ID信息
➜  ~ docker images
REPOSITORY                     TAG                 IMAGE ID            CREATED             SIZE
ubuntu                         14.04               dc4491992653        4 weeks ago         222MB
local_topology                 latest              ebb6ac1b94ed        6 months ago        579MB

# 打标签
➜  ~ docker tag ubuntu:14.04 myunbuntu:latest

# 获取详细信息
➜  ~ docker inspect ubuntu:14.04
[
    {
        "Id": "sha256:dc4491992653ecf02ae2d0e9d3dbdaab63af8ccdcab87ee0ee7e532f7087dd73",
        "RepoTags": [
            "myunbuntu:latest",
            "ubuntu:14.04"
        ],
#获取详细信息里面的单向
➜  ~ docker inspect -f {{".RepoTags"}} ubuntu:14.04
[myunbuntu:latest ubuntu:14.04]

#搜索镜像
➜  ~ docker search nginx
NAME                                                   DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
nginx                                                  Official build of Nginx.                        8014                [OK]
jwilder/nginx-proxy                                    Automated Nginx reverse proxy for docker con…   1282                                    [OK]
richarvey/nginx-php-fpm                                Container running Nginx + PHP-FPM capable of…   526

#删除镜像 可以直接跟着镜像ID删除
➜  ~ docker rmi myunbuntu:latest #只是删除对应标签
Untagged: myunbuntu:latest
➜  ~ docker rmi ubuntu:14.04 #删除
Untagged: ubuntu:14.04
Untagged: ubuntu@sha256:e1c8bff470c771c6e86d3166607e2c74e6986b05bf339784a9cab70e0e03c7c3
Deleted: sha256:dc4491992653ecf02ae2d0e9d3dbdaab63af8ccdcab87ee0ee7e532f7087dd73
Deleted: sha256:1239c33230909cc231da97b851df65e252dc9811dcee2af0ecf3b225e2805a31
Deleted: sha256:ce4caf69568d9109febd1f5307b62d85ab84e7a947fded041be49c847b412e5a
Deleted: sha256:4c711cc0452303f0fb6ce885c84130e32bb649b03f690fd0e4626a874b1cc8cf
Deleted: sha256:a375921af0e34b1cb09a35af24265db01b1eb65edabadaf70d56505a60a6de2b
Deleted: sha256:c41b9462ea4bbc2f578e42bd915214d54948d960b9b8c6815daf8586811c2d38

#在有对应的容器下是无法删除的
➜  ~ docker rmi -f ubuntu:14.04 #强行删除

#导出镜像为压缩文件
➜  ~ docker save -o redis.tar redis:latest
#导入镜像文件
➜  ~ docker load --input redis.tar
Loaded image: redis:latest

#上传镜像到Docker Hub官方仓库
docker push name[:tag] user/test:latest
```

#### 创建镜像：

1）基于已经用的镜像创建，2）基于本地模板创建，3）基于Dockerfile创建。

## 容器：

```shell
#创建容器
➜  ~ docker create -it ubuntu:latest
#查找容器
➜  ~ docker ps -a
CONTAINER ID        IMAGE                             COMMAND                  CREATED             STATUS                          PORTS                                                                  NAMES
1be066aaaa6b        ubuntu:latest                     "/bin/bash"              11 minutes ago      Created                                                                                                confident_visvesvaraya
6c6617a9158a        swarm                             "/swarm mannage toke…"   5 months ago        Exited (0) 5 months ago                                                                                focused_almeida
```

| 说明                      | 选项                   |
| ----------------------- | -------------------- |
| -a,—attach=[]           | 是否绑定到标准输入，输出和错误      |
| -d,—detach=true\|false  | 是否在后台运行容器，默认为否       |
| —group-add=[]           | 运行容器的用户组             |
| —expose=[]              | 指定容器暴露的端口和端口范围       |
| —ipc=""                 | 容器ipc命名空间            |
| —net="bridge"           | 容器网络模式               |
| -t                      | 是否分配一个伪终端，默认为false   |
| --rm                    | 容器退出后是否自动删除          |
| —add-host=[]            | 在容器内添加一个主机到IP地址的映射关系 |
| —dns-search             | DNS搜索域               |
| —dns=[]                 | 自定义DNS               |
| -e,—env=[],—env-file=[] | 指定容器内的环境变量           |
| -h,—host=""             | 主机名称                 |
| —ip="",—ip6=""          | 容器ipv4地址 ,ipv6地址     |
| —link=                  | 链接到其他容器              |
| —name=""                | 容器别名                 |
|                         |                      |
|                         |                      |
|                         |                      |

还有很多参数。

```shell
#启动容器

➜  ~ docker start 1be066aaaa6b
1be066aaaa6b

#新建并启动容器
➜  ~ docker run ubuntu /bin/echo 'Hi Docker'
Hi Docker

#停止容器
➜  ~ docker stop d074e494069f
d074e494069f
➜  ~ docker ps -qa
d074e494069f
1be066aaaa6b
6c6617a9158a
9071d88eac2d

#重新启动
➜  ~ docker restart d074e494069f
d074e494069f
#进入容器
docker attach 容器 类似-d
docker exec 容器

#删除容器 要强制删除-f
➜  ~ docker rm d074e494069f
d074e494069f

#批量删除
➜  ~ docker rm -f $(docker ps -qa)
1be066aaaa6b
b145831810aa
c3a8c84f9958
a22a43c63be7
54b69fad3c39
a8ea7c92a20c

#导出容器
docker export -o test.tar d074e494069f
docker export d074e494069f > test.tar
#导入容器
docker import test.tar - test/ubuntu:v1.0

```

## 仓库：

登录远端docker login，或者配置.dockercfg文件。

### 搭建本地仓库：

```shell
docker run -d -p 5000:5000 register
#register为官方的一个仓库镜像
#指定镜像上传目录
docker run -d -p 5000:5000 -v /opt/data/register:/tmp/register register
```



## 数据管理：

### 数据卷：

即将主机的系统目录挂载映射到容器目录。使用-`v` 参数，实现数据共享，快速修改。

```shell
docker run -d -P --name web -v /src/myweb:/webapp python app.py
#默认是读写rw权限，可以指定容器的权限为只读ro
docker run -d -P --name web -v /src/myweb:/webapp:ro python app.py
```

把本地的~/myweb挂在容器的/webapp下

### 数据卷容器：

即在容器和容器之间共享数据，当然也可以用数据卷，数据卷容器，即单独拿一个容器出来，供其他容器使用。

```shell
#创建一个数据卷容器
docker run -it -v /dbdata --name dbdata ubuntu
#使用改容器 可以挂载多个
docker run -it --volumes-from dbdata --name db1 ubuntu
docker run -it --volumes-from dbdata --name db2 ubuntu
```

可以用来备份数据

```shell
$ docker run --volumes-from dbdata -v $(pwd):/backup --name backwork ubuntu tar cvf /backup/backup.tar /dbdata
#恢复
$ docker run -v /dbdata --name dbdata2 ubuntu /bin/bash
```

## 容器互联：

### 端口映射：

```shell
#-P 大写 随机分配一个端口映射到本地端口
$ docker run -d -P training/webapp python app.py
#-p 小写 可以指定多个 指定本地端口5001映射到容器端口5000
$ docker run -d -p 5001:5000 -p 3000:80 training/webapp python app.py

#查看端口映射
➜  ~ docker port a8ea7c92a20c
23/tcp -> 0.0.0.0:23
443/tcp -> 0.0.0.0:443
80/tcp -> 0.0.0.0:80
#可以限定本地ip，默认是所有ip
$ docker -run -d -p 127.0.0.1:3000:80 training/webapp python app.py
```

### 容器互联：

首先需要一个唯一好记的名字。—name 名字

```shell
$ docker run -d --name web --link db:aliasdb training/webapp python app.py
# --link name:alias name为要链接的容器名称，alias为指定一个别名
```

打开一个虚拟通道，并在容器中的环境变量，和/etc/hosts添加访问支持。

## Dockerfile：

dockerfile由一行命令组成，并且支持#注释。

| 指令          | 说明                |
| ----------- | ----------------- |
| FROM        | 指定基础镜像，源镜像        |
| MAINTAINTER | 指定维护者信息           |
| RUN         | 运行命令              |
| CMD         | 指定启动容器时默认执行的命令    |
| LABEL       | 标签信息              |
| EXPOSE      | 容器监听的端口           |
| ENV         | 指定环境变量            |
| ADD         | 复制文件内容到容器里面（自动解压） |
| COPY        | 复制本机内容到容器里面，推荐使用  |
| ENTRYPOINT  | 指定镜像的默认入口         |
| VOLUME      | 创建数据卷挂载点          |
| USER        | 指定运行的用户名或UID      |
| WORKDIR     | 指定工作目录            |
| ARG         | 指定镜像内使用的参数        |
| ONBUILD     | 创建时只需的命令          |
| STOPSIGNAL  | 容器退出的信号值          |
| HEALTHCHECK | 如何健康检查            |
| SHELL       | 指定shell时的默认类型     |
|             |                   |

```shell
#基于dockerfile构建镜像
$ docker build -t buildnam:v1 ~/dockerfiledir/
```

## 案例：

### 系统：

Busybox：常用linux工具箱系统，只有几MB。

Alpine：面向安全的轻型Linux

Ubuntu/Debian：

CentOS/Fedora：

### WEB服务：

Apache,Nginx,Tomcat，Jetty(java的Servlet容器)，LAMP(集成环境)，Ghost(js的开源博客系统) ，Jenkins，Gitlab,

### 数据库：

Mysql，MongoDB，Redis,Memcached,CouchDB 面向文档的NoSQL数据库，以JSON格式存储数据。Cassandra 分布式数据库，支持分散数据存储。

### 分布式：

RabbitMQ消息队列，Celery任务队列。Hadoop分布式平台。Saprk，Storm，Elastisearch



## 流程：

运维制作基础镜像->开发利用基础镜像进行开发->开发提供应用镜像->测试部门进行测试->问题反馈->运维部署上线

进阶：网络，安全等

## Docker Compose：

定义和运行多个Docker容器。

| 命令      | 说明        |
| ------- | --------- |
| build   | 构建        |
| help    |           |
| kill    | 停止        |
| logs    | 查看服务输出    |
| pasue   | 暂停        |
| port    | 打印对应的公共端口 |
| ps      | 列出所有容器    |
| pull    |           |
| restart |           |
| rm      |           |
| run     |           |
| scale   |           |
| start   |           |
| stop    |           |

### 模板文件：

yml格式

```dockerfile
webapp:
image:examples/web
ports:
	- "80:80"
volumes:
	- "/data"
```

```shell
$ sudo docker-compose up
```

## Swarm：

容器云管理，集群管理。



## Mesos：

集群资源调度平台

## Kubernets：

生产级容器集群平台



## 扩展：

https://www.cnblogs.com/ningskyer/articles/6074804.html 10张图了解docker

```idl
docker version 查看docker的版本号，包括客户端、服务端、依赖的Go等
docker info 查看系统(docker)层面信息，包括管理的images, containers数等
docker search <image> 在docker index中搜索image
docker pull <image> 从docker registry server 中下拉image
docker push <image|repository> 推送一个image或repository到registry
docker push <image|repository>:TAG 同上，指定tag
docker inspect <image|container> 查看image或container的底层信息
docker images TODO filter out the intermediate image layers (intermediate image layers 是什么)
docker images -a 列出所有的images
docker ps 默认显示正在运行中的container
docker ps -l 显示最后一次创建的container，包括未运行的
docker ps -a 显示所有的container，包括未运行的
docker logs <container> 查看container的日志，也就是执行命令的一些输出
docker rm <container...> 删除一个或多个container
docker rm `docker ps -a -q` 删除所有的container
docker ps -a -q | xargs docker rm 同上, 删除所有的container
docker rmi <image...> 删除一个或多个image
docker start/stop/restart <container> 开启/停止/重启container
docker start -i <container> 启动一个container并进入交互模式
docker attach <container> attach一个运行中的container
docker run <image> <command> 使用image创建container并执行相应命令，然后停止
docker run -i -t <image> /bin/bash 使用image创建container并进入交互模式, login shell是/bin/bash
docker run -i -t -p <host_port:contain_port> 将container的端口映射到宿主机的端口
docker commit <container> [repo:tag] 将一个container固化为一个新的image，后面的repo:tag可选
docker build <path> 寻找path路径下名为的Dockerfile的配置文件，使用此配置生成新的image
docker build -t repo[:tag] 同上，可以指定repo和可选的tag
docker build - < <dockerfile> 使用指定的dockerfile配置文件，docker以stdin方式获取内容，使用此配置生成新的image
docker port <container> <container port> 查看本地哪个端口映射到container的指定端口，或者用docker ps 也可以看到。
```

