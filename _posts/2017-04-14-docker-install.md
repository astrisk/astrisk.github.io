---
title:  "Docker-install"
date:   2017-04-14 12:47:10
categories: DOCKER-HACKATHON
---

Docker提供yum和rpm两种安装方式,可以根据自己的实际情况选择安装方式。yum安装需要联网，可以直接在线安装；而rpm一般用于不能直接连接外网的环境，先下载rpm包，上传到服务器，然后安装。

## 使用yum安装

**Install yum-utils, which provides the yum-config-manager utility:**


{% highlight c %}

$ sudo yum install -y yum-utils

{% endhighlight %}


**set up the stable repository**

{% highlight c %}

$ sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

{% endhighlight %}

![docker-install1]({{ site.url }}/images/docker/docker-install1.png)

**Optional: Enable the edge repository**

{% highlight c %}

$ sudo yum-config-manager --enable docker-ce-edge

{% endhighlight %}

**Update the yum package index**

{% highlight c %}

$  sudo yum makecache fast

{% endhighlight %}

**List the available versions of docker**

{% highlight c %}

$  yum list docker-ce.x86_64  --showduplicates |sort -r

{% endhighlight %}

![docker-install2]({{ site.url }}/images/docker/docker-install2.jpg)

**install docker-ce**

{% highlight c %}

$  yum install -y docker-ce.x86_64

{% endhighlight %}

![docker-install5]({{ site.url }}/images/docker/docker-install5.jpg)

**Start Docker**

{% highlight c %}

$ sudo systemctl start docker

{% endhighlight %}

**Verify that docker is installed correctly by running the hello-world image**

{% highlight c %}

$ sudo docker run hello-world

{% endhighlight %}

![docker-install6]({{ site.url }}/images/docker/docker-install6.jpg)

![docker-install7]({{ site.url }}/images/docker/docker-install7.jpg)

**安装遇到地坑**

selinux版本不满足docker-ce要求，selinux版本太低
![docker-install3]({{ site.url }}/images/docker/docker-install3.jpg)

查看CentOS上安装的selinux版本
![docker-install4]({{ site.url }}/images/docker/docker-install4.jpg)

**解决方法**

- 选择比较新的版本，比如CentOS-7-x86_64-XXX-1611（验证过）
- 安装低版本的docker-engine