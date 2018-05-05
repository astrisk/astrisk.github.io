title:  "在GCP ubuntu上部署shadowsocks服务器"
date:   2018-05-5 08:37:00
categories: kubernetes-docker
---

- Linux选择ubuntu 16
- 地区选择asia-east1-c
- 配置：1核1G
- 新建一个外网IP

**更新源和升级系统&&安装shadowsocks**

{% highlight c %}

$ sudo su
# apt update
# apt upgrade
# echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf
# echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf
# sysctl -p
# lsmod | grep bbr
# apt-get update
# apt-get install python-pip
# pip install shadowsocks

{% endhighlight %}

**创建配置文件**

{% highlight c %}

# vim /etc/ss-conf.json

{
"server":"0.0.0.0",
"server_port":8838,
"local_address":"127.0.0.1",
"local_port":1080,
"password":"123456",
"timeout":600,
"method":"aes-256-cfb"
}

{% endhighlight %}

**创建启动脚本**

{% highlight c %}

# vim /etc/init.d/shadowsocks.sh

#!/bin/bash  
# command content  
/usr/bin/python /usr/local/bin/ssserver -c /etc/ss-conf.json -d start
exit 0


# chmod +x /etc/init.d/shadowsocks.sh

{% endhighlight %}


**设置开机启动**

{% highlight c %}

# update-rc.d shadowsocks.sh  defaults
# reboot #重启

{% endhighlight %}

**重启后登录服务器检查**

{% highlight c %}

# ps -ef | grep ssserver #检查进程是否开机启动

{% endhighlight %}