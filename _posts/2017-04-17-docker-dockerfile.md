---
title:  "DOCKER-DOCKERFILE详解"
date:   2017-04-14 12:47:10
categories: DOCKER-HACKATHON
---

构建docker image有两种方式：

1. 用交互方式进入container，执行各自操作（更新，安装应用包，启动应用程序等）后，用exit退container,最后使用docker commit提交，生成目标docker image;

2. 使用Dockerfile,通过在Dockerfile定义各种指令，然后通过docker build来构建docker image。

这两种方式本质上都是一样的，即在在base container中添加(更新)各种应用程序或脚本等操作，生成一个新的docker container，最后利用docker commit把docker container变成docker image。不过利用Dockerfile有利于管理维护和共享，所以基本上我们都使用Dockerfile来构建docker image。下面让我们来详细了解Dockerfile指令。

## Usage

docker build根据Dockerfile和context构建docker image。build context是位于本地或网络上的文件，可以通过PATH或URL来指定context，PATH为本地文件系统的目录，URL为git repo位置。

context是以递归的方式处理的，所以如果指本地目录PATH，则会包含所有的子目录；同理，URL会包含所有git repo的子模块。

镜像的构建是在Docker daemon上执行，而不是通过CLI。构建的第一个阶段就是把完整的context(递归)发送给docker daemon。在大多数情况下，做好新建一个空目录做为context，把Dockerfile和仅构建所需的文件添加到构建目录中。


{% highlight c %}

$ docker build  .

{% endhighlight %}

可以通过-f选项指定Dockerfile文件位置，-t选项指定repo和tag

{% highlight c %}

$ docker build  -f /path/to/Dockerfile

{% endhighlight %}


{% highlight c %}

$ docker build -t shykes/myapp .

{% endhighlight %}
