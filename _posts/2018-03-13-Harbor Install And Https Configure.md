---
title:  "Docker Private Registry Harbor Install And Https Configure"
date:   2018-03-13 23:37:00
categories: kubernetes-docker
---


**安装环境**

{% highlight c %}
#  cat /etc/redhat-release
CentOS Linux release 7.2.1511 (Core)
Python 2.7.5 (default, Aug  4 2017, 00:39:18) #系统自带
# docker -v
Docker version 17.06.1-ce, build 874a737
# docker-compose -v
docker-compose version 1.17.0, build ac53b73

{% endhighlight %}

**安装docker**

{% highlight c %}
https://docs.docker.com/engine/installation/

{% endhighlight %}

**安装docker-compose**

{% highlight c %}
https://docs.docker.com/compose/install/

{% endhighlight %}

**安装Harbor**


**下载Harbor**

{% highlight c %}
# wget https://github.com/vmware/harbor/releases/download/v1.2.2/harbor-offline-installer-v1.2.2.tgz
# tar -zxvf harbor-offline-installer-v1.2.2.tgz
# cd harbor

{% endhighlight %}

**生成https需要证书**

{% highlight c %}
# mkdir -p /data/cert/
# cd /data/cert/
# export localdomain=harbor.qiusheng.cn
# echo $localdomain
# openssl req -nodes -subj "/C=CN/ST=BeiJing/L=BeiJing/CN=$localdomain" -newkey rsa:2048 -keyout $localdomain.key -out $localdomain.csr
# openssl x509 -req -days 3650 -in $localdomain.csr -signkey $localdomain.key -out $localdomain.crt
# openssl x509 -req -in $localdomain.csr -CA $localdomain.crt -CAkey $localdomain.key -CAcreateserial -out $localdomain.crt -days 10000

{% endhighlight %}

**修改配置**

{% highlight c %}
# cd ${HARBOR_HOME}
# vim harbor.cfg

#The IP address or hostname to access admin UI and registry service.
#DO NOT use localhost or 127.0.0.1, because Harbor needs to be accessed by external clients.
hostname = harbor.qiusheng.cn

#The protocol for accessing the UI and token/notification service, by default it is http.
#It can be set to https if ssl is enabled on nginx.
ui_url_protocol = https #https

#The path of cert and key files for nginx, they are applied only the protocol is set to https
ssl_cert = /data/cert/harbor.qiusheng.cn.crt
ssl_cert_key = /data/cert/harbor.qiusheng.cn.key

# 保存退出
# ./prepare


{% endhighlight %}

**安装启动Harbor**

{% highlight c %}
# ./install.sh

{% endhighlight %}

- 更新hosts文件（/etc/hosts）

	- 10.19.52.173 harbor.qiusheng.cn
- 在浏览器输入URL https://harbor.qiusheng.cn 登录HarborUI
	- 默认用户：admin 密码：Harbor12345

**此时docker client还不能登录，需要进行以下步骤配置证书，如果在不同机器，需要把服务器的证书下发到客户端**

**停止Harbor**

{% highlight c %}
# docker-compose down

{% endhighlight %}

**更新客户端证书**

{% highlight c %}
# mkdir -p /etc/docker/certs.d/harbor.qiusheng.cn
# cp /data/cert/harbor.qiusheng.cn.crt /etc/docker/certs.d/harbor.qiusheng.cn/ca.crt

{% endhighlight %}

**重启Harbor**

{% highlight c %}
# docker-compose up -d

{% endhighlight %}

**CentOS docker 客户端登录**

{% highlight c %}
# docker login harbor.qiusheng.cn
user:admin  password: Harbor12345

Username: admin
Password:
Login Succeeded

{% endhighlight %}

**Mac下配置docker客户端**

{% highlight c %}
# scp scp zhangqiusheng@xx.xx.xx.xx:~/ca.crt ./ #从服务器下载CA证书
# sudo cat ca.crt >> /etc/ssl/certs/ca-certificates.crt
# sudo cd /etc/ssl/certs/
# sudo security add-trusted-cert -d -r trustRoot -k /Library/Keychains/System.keychain ./ca-certificates.crt #添加操作系统信任证书
# 重启docker
# docker login harbor.qiusheng.cn -u admin -p Harbor12345

{% endhighlight %}

**参考文章**
- https://www.58jb.com/html/117.html
- https://www.cnrancher.com/harbor-docker-distribution-registry/
- https://github.com/vmware/harbor/blob/master/docs/installation_guide.md
- https://my.oschina.net/u/2306127/blog/787078
- http://itttl.com/blog/docker_for_mac_add_certs.html
