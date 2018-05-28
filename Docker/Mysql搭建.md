## 准备：

```shell
➜  mysql pwd
/Users/borgxiao/docker/mysql

#这里也可以使用mysql的 因为公司需要，这里选择mariadb
➜  mysql docker pull mariadb
```



## 启动docker实例：

为了方便测试，就不做主从，这里主要考虑多实例

```shell
➜  mysql docker run --name mysql-3306 -p 3306:3306 -e MYSQL\_ROOT\_PASSWORD=123456 -d mariadb
➜  mysql docker run --name mysql-3307 -p 3307:3306 -e MYSQL\_ROOT\_PASSWORD=123456 -d mariadb
➜  mysql docker run --name mysql-3308 -p 3308:3306 -e MYSQL\_ROOT\_PASSWORD=123456 -d mariadb
```

