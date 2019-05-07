# 在自己的服务器上GitLab搭建

## 1）安装

参考官网：https://about.gitlab.com/installation/#centos-7

注：可能比较慢，部分教程可以指定中国yum源下载

```shell
sudo yum install -y curl policycoreutils-python openssh-server
sudo systemctl enable sshd
sudo systemctl start sshd
sudo firewall-cmd --permanent --add-service=http
sudo systemctl reload firewalld

#下面可选 发送邮件的 我没有安装
sudo yum install postfix
sudo systemctl enable postfix
sudo systemctl start postfix

curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ee/script.rpm.sh | sudo bash

#记得修改为自己的域名
sudo EXTERNAL_URL="http://gitlab.example.com" yum install -y gitlab-ee

```



## 2）配置修改：

因为我自己的服务器已经有nginx了，这里需要修改gitlab的自身的nginx监控端口

```shell
sudo vim /etc/gitlab/gitlab.rb

#修改为8090(自己根据情况定) 默认为nil
nginx['listen_port'] = 8090
```



## 3）修改服务器默认nginx配置：

主要是添加代理到gitlab的nginx上：

```shell
upstream  git{
    # 域名对应 gitlab配置中的 external_url
    # 端口对应 gitlab 配置中的 nginx['listen_port']
    server  127.0.0.1:8090;
}


server{
    listen 80;
    # 此域名是提供给最终用户的访问地址 这里要和gitlab指定的域名一致
    server_name gitlab.example.com;

    location / {
        # 这个大小的设置非常重要，如果 git 版本库里面有大文件，设置的太小，文件push 会失败，根>据情况调整
        client_max_body_size 50m;

        proxy_redirect off;
        #以下确保 gitlab中项目的 url 是域名而不是 http://git，不可缺少
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        # 反向代理到 gitlab 内置的 nginx
        proxy_pass http://git;
        index index.html index.htm;
    }
```



## 4）启动运行：

```shell
#Reconfigure the application
sudo gitlab-ctl reconfigure
sudo gitlab-ctl start/stop/restart
```

注意：运行后直接访问，用root,第一次访问会让修改root密码，修改后用新的密码访问即可