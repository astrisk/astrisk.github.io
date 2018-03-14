---
title:  "Tsung Cluster install"
date:   2018-03-13 23:57:00
categories: performance
---


**OS Check**


{% highlight c %}
# vim /etc/security/limits.conf

*  soft  nofile  1024000
*  hard  nofile  1024000

{% endhighlight %}
Logout once and check logout ulimit -n


{% highlight c %}
# vim /etc/sysctl.conf

# General gigabit tuning
net.core.rmem_max = 16777216
net.core.wmem_max = 16777216
net.ipv4.tcp_rmem = 4096 87380 16777216
net.ipv4.tcp_wmem = 4096 65536 16777216

# This gives the kernel more memory for TCP
# which you need with many (100k+) open socket connections
net.ipv4.tcp_mem = 50576 64768 98152

# Backlog
net.core.netdev_max_backlog = 2048
net.core.somaxconn = 1024
net.ipv4.tcp_max_syn_backlog = 2048
net.ipv4.tcp_syncookies = 1

# tcp local port range
net.ipv4.ip_local_port_range = 4000 65000


# sysctl -p /etc/sysctl.conf

{% endhighlight %}



**依赖包安装**

{% highlight c %}
# yum -y updae
# yum list perl gcc unixODBC unixODBC-devel  ncurses-devel  gnuplot python python-matplotlib.x86_64

{% endhighlight %}

**erlang**

{% highlight c %}
# wget http://www.erlang.org/download/otp_src_20.1.tar.gz
# tar -zxvf otp_src_20.1.tar.gz
# cd otp_src_20.1
# ./configure
# make && make install

# erl #验证
# ln -s /usr/local/erlang/lib/erlang/bin/erl /usr/bin/erl #如果不设置，Tsung会找不到erlang
{% endhighlight %}

**Tsung**

{% highlight c %}
# wget http://tsung.erlang-projects.org/dist/tsung-1.7.0.tar.gz
# tar -zxvf tsung-1.7.0.tar.gz
# cd tsung-1.7.0
# ./configure
# make && make install
# tsung -v

{% endhighlight %}


**Tsung cluster Configure**

- 按照以上步骤在每个slave上安装相应程序
- master - slave实现ssh免密钥登录
- 在cluster中每台服务器的hosts文件中增加map

{% highlight c %}
	172.31.0.198 tsung00
	172.31.0.231 tsung01
	172.31.3.163 tsung02
	
{% endhighlight %}

**Template-toolkit**
{% highlight c %}
# wget wget http://cpan.org/modules/by-module/Template/Template-Toolkit-2.24.tar.gz  
# tar -zxvf Template-Toolkit-2.24.tar.gz
# cd Template-Toolkit-2.24
# perl Makefile.PL 
# make 
# make test
# make install

{% endhighlight %}

**Gnuplot**

{% highlight c %}
# wget http://nchc.dl.sourceforge.net/project/gnuplot/gnuplot/4.4.0/gnuplot-4.4.0.tar.gz
# tar -zxvf gnuplot-4.4.0.tar.gz
# cd gnuplot-4.4.0
# ./configure
# make && make install


{% endhighlight %}

**Tsung脚本**

{% highlight c %}
  <!-- Client side setup -->
  <clients>
    <client host="tsung00" use_controller_vm="true"/>
    <client host="tsung01" use_controller_vm="true"/>
    <client host="tsung02" use_controller_vm="true"/>
  </clients>

{% endhighlight %}

**Run tsung**

{% highlight c %}
# tsung -f http_simple.xml -l ./logs start

# ps aux | grep erl # 分别在cluster各个节点看是否启动了相应Tsung 进程

{% endhighlight %}

**Report**

{% highlight c %}
# ln -s /usr/local/lib/tsung/bin/tsung_stats.pl  /usr/bin/rp
# cd /tsung-1.7.0/examples/logs/20171122-1542
# rp
# python -m SimpleHTTPServer 8090

在浏览器可以输入URL：http://xxxx:8090/report.html查看测试结果
http://35.177.121.79:8090/report.html
{% endhighlight %}