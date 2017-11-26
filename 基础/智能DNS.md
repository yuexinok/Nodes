## 简介：

源于：慕课网，讲师：Jeson

开源，稳定的DNS服务。域名解析服务。

## Bind服务：

www.bigbig.me.

.根域  me 一级域名  bigbig.me 二级域名

A记录：标准的 域名->IP地址

CNANE记录：域名->域名->IP地址  如www.bigbig.me —> bigbig.me -> 1.0.0.1(A记录) 用于多个域名指向同一个域名www.img.bigbig.me —>img1.aliyun.con ->10.0.1

准许一个域名，用另外运营商解析A记录。

MX记录：邮件服务的解析。

NS记录：域名->DNS解析服务->IP地址。

### 安装：

yum install bind bind-chroot

rpm -qa | grep bind

rpm -ql bind | more 查看安装哪些文件内容

/etc/init.d/named start

### 配置：

options{} 整个bind的全局选项

logging{} 日志

zone{} DNS域名解析

zooe "bigbig.me"{

​	type master;

​	file "bigbig.me.zone"

};

//bigbig.me.zone配置

$TTL 7200 //生效时间

@ IN SOA bigbig.me. **borgxiao.bigbig.me.** (101 1H 15M 1W 1D)  加粗的为邮箱，因为@为特殊字符换成点 SOA权威解析记录

bigbig.me. IN NS dns1.bigbig,me.

dns1 IN A 10.0.0.1

www IN A 2.2.2.2

### 测试：

dig @10.0.0.1 www.bigbig.me  查找DNS服务10.0.0.1的www.bigbig.me

正向解析：A记录

反向解析：PTR记录

配置DNS:/etc/resolv.conf

nslookup:通用，window支持

➜  bin nslookup www.bigbig.me
Server:		120.196.165.24
Address:	120.196.165.24#53

Non-authoritative answer:
Name:	www.bigbig.me
Address: 119.23.59.238

dig：专业

host:简单明了

➜  bin host -t NS www.bigbig.me
www.bigbig.me has no NS record

➜  bin host -t A www.bigbig.me
www.bigbig.me has address 119.23.59.238
➜  bin host www.bigbig.me
www.bigbig.me has address 119.23.59.238

## Bind负载均衡：

### DNS转发：

递归查询：client->server,然后server递归查询 一次请求得到结果

迭代查询：server->server1->server2->…结果 多次请求得到结果

recursion:yes/no 是否准许递归

forwarders

### DNS主从模式：

区域：正向，反向。全量区域传输，增量区域传输。

### DNS传输限制：

基于主机传输限制。

DES:对称加密，加密和解密用相同的密钥，简单快捷

IDEA:非对称加密  公钥，私钥，安全性高



## 智能DNS:

即，电信的访问电信，联通访问联通。。

1，减少动态服务的响应时间。

2，CDN加速。

3，负载均衡。

4，防止DDOS攻击。

通过IP库来实现。APINC。

DNS信息污染。因为使用的是UDP。

