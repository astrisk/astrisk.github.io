---
title:  "Docker-Uninstall"
date:   2017-04-14 10:09:31
categories: DOCKER-HACKATHON
---

**查看已经安装的docker程序**


{% highlight c %}

$ yum list installed | grep docker

{% endhighlight %}

![docker-uninstalled1]({{ site.url }}/images/docker/docker-uninstalled1.png)

**停止所有正在运行的容器**

{% highlight c %}

$ docker stop docker ps -q 

{% endhighlight %}

备注：如果不先停止运行的container,后面在清理资源时，有些挂载的文件会因为无法删除报错。

**卸载docker**

{% highlight c %}

$ yum remove docker-engine*

{% endhighlight %}

![docker-uninstalled2]({{ site.url }}/images/docker/docker-uninstalled2.png)

**清理docker资源**

- docker安装后，相关资源的存储目录为/var/lib/docker/
![docker-uninstalled3]({{ site.url }}/images/docker/docker-uninstalled3.png)

{% highlight c %}

$ rm -rf /var/lib/docker

{% endhighlight %}

最后世界又回到原始位置；让我们重头开始
![docker-uninstalled4]({{ site.url }}/images/docker/docker-uninstalled4.png)