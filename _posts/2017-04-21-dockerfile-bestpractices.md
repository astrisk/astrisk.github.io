---
title:  "DOCKER-DOCKERFILE最佳实践"
date:   2017-04-21 09:27:35
categories: DOCKER-HACKATHON
---

Dockerfile提供简单的语法来构建images，如何编写优雅的Dcokerfile是有技巧的。

**使用缓存**

Dockerfile里的每条指令都会将结果提交形成新的镜像缓存，下一条指令会基于镜像环境（上一条指令提交的）构建，如果一个镜像存在相同的父镜像和指令（除ADD），Docker将会使用使用镜像缓存，而不是重新执行指令。

为了有效地利用缓存，需要Dockerfile尽可能的保持一致。如作者的Dockerfile前5行如下：

{% highlight c %}

FROM ubuntu
MAINTAINER Michael Crosby <michael@crosbymichael.com>
RUN echo "deb http://archive.ubuntu.com/ubuntu precise main universe" > /etc/apt/sources.list
RUN apt-get update
RUN apt-get upgrade -y

{% endhighlight %}

**使用标签**

除非是在做试验，否则构建镜像时都应该使用-t选项来指定新镜像的标签。使用标签提高了镜像的可读性，便于镜像的维护管理。

{% highlight c %}

docker build -t="crosbymichael/sentry" .

{% endhighlight %}

**公开端口**

可移植性和可重复性是docker的两个核心概念。镜像可以放在任何主机尽可能多地运行。Dockerfile里可以定义私有端口和共用端口的映射，但是永远不能在Dockerfile里映射共用端口，由于绑定了公共端口，由于公共端口限制在同一时刻只能运行一个镜像。

{% highlight c %}

# bad practices
# private and public mapping
EXPOSE 80:8080

# best practices
# private only
EXPOSE 80

{% endhighlight %}

如果镜像使用者需要关注容器私有端口映射哪个或哪些公共端口，可以在运行镜像时，通过-p选项指定，否者Docker会自动分配公共端口。

**CMD和ENTRYPOINT语法**

CMD和ENTRYPOINT指令都比较简单，但是这里都有个隐藏的坑，如果不注意，很有很能会踩到。这两个指令都有以下两种语法。

{% highlight c %}

CMD /bin/echo
CMD ["/bin/echo"]

ENTRYPOINT /bin/echo
ENTRYPOINT ["/bin/echo"]

{% endhighlight %}

但使用数组语法时，命令运行结果跟预期完全一样。如果使用第一种语法，Docker会在命令前面加上/bin/sh -c,如果不了解Docker修改了CMD/ENTRYPOINT命令，可能出现些意想不到的结果，并且难于理解。

使用CMD和ENTRYPOINT时，务必使用数组语法。

**CMD和ENTRYPOINT结合更好**

docker run 命令中的参数都会传递给ENTRYPOINT指令，而不用担心ENTRYPOINT指令会被覆盖（CMD指令会）。当把ENTRYPOINT和CMD指令结合起来使用，会更好。以下是Rethinkdb的 Dockerfile示例

{% highlight c %}

#Dockerfile for Rethinkdb 
#http://www.rethinkdb.com/

FROM ubuntu

MAINTAINER Michael Crosby <michael@crosbymichael.com>

RUN echo "deb http://archive.ubuntu.com/ubuntu precise main universe" > /etc/apt/sources.list
RUN apt-get update
RUN apt-get upgrade -y

RUN apt-get install -y python-software-properties
RUN add-apt-repository ppa:rethinkdb/ppa
RUN apt-get update
RUN apt-get install -y rethinkdb

#Rethinkdb process
EXPOSE 28015
#Rethinkdb admin console
EXPOSE 8080

#Create the /rethinkdb_data dir structure
RUN /usr/bin/rethinkdb create

ENTRYPOINT ["/usr/bin/rethinkdb"]

CMD ["--help"]

{% endhighlight %}

在以上的示例中，把ENTRYPOINT和CMD结合使用，在docker run过程中传递的参数将被当作参数传递给ENTRYPOINT("/usr/bin/rethinkdb")。
同时还设置了默认的CMD参数--help。这样当运行docker run时，如果没有传递参数，将把CMD["--help"]参数传递给ENTRYPOINT，给用户显示默认的帮助文档。

- 运行以下命令

{% highlight c %}

docker run crosbymichael/rethinkdb

{% endhighlight %}

- 输出

{% highlight c %}

Running 'rethinkdb' will create a new data directory or use an existing one,
  and serve as a RethinkDB cluster node.
File path options:
  -d [ --directory ] path           specify directory to store data and metadata
  --io-threads n                    how many simultaneous I/O operations can happen
                                    at the same time

Machine name options:
  -n [ --machine-name ] arg         the name for this machine (as will appear in
                                    the metadata).  If not specified, it will be
                                    randomly chosen from a short list of names.

Network options:
  --bind {all | addr}               add the address of a local interface to listen
                                    on when accepting connections; loopback
                                    addresses are enabled by default
  --cluster-port port               port for receiving connections from other nodes
  --driver-port port                port for rethinkdb protocol client drivers
  -o [ --port-offset ] offset       all ports used locally will have this value
                                    added
  -j [ --join ] host:port           host and port of a rethinkdb node to connect to
  .................
  

{% endhighlight %}

- 传入参数--bind all执行docker run 

{% highlight c %}

docker run crosbymichael/rethinkdb --bind all

{% endhighlight %}

- 输出

{% highlight c %}

info: Running rethinkdb 1.7.1-0ubuntu1~precise (GCC 4.6.3)...
info: Running on Linux 3.2.0-45-virtual x86_64
info: Loading data from directory /rethinkdb_data
warn: Could not turn off filesystem caching for database file: "/rethinkdb_data/metadata" (Is the file located on a filesystem that doesn't support direct I/O (e.g. some encrypted or journaled file systems)?) This can cause performance problems.
warn: Could not turn off filesystem caching for database file: "/rethinkdb_data/auth_metadata" (Is the file located on a filesystem that doesn't support direct I/O (e.g. some encrypted or journaled file systems)?) This can cause performance problems.
info: Listening for intracluster connections on port 29015
info: Listening for client driver connections on port 28015
info: Listening for administrative HTTP connections on port 8080
info: Listening on addresses: 127.0.0.1, 172.16.42.13
info: Server ready
info: Someone asked for the nonwhitelisted file /js/handlebars.runtime-1.0.0.beta.6.js, if this should be accessible add it to the whitelist.

{% endhighlight %}


**禁止开机初始化**

容器模型跟虚拟机不一样，不需要执行开机初始化。即使你认为需要，也是错误的。

**信任构建**

在推送镜像之前，一定要先本地构建。Docker可以确保本地的构建和运行与推送到任何地方的构建和运行是一致的。本地开发，测试，提交和推送以及等待索引上的官方镜像都是建立在可信任构建的基础上。

**不要在容器中更新(upgrade)**

在基础镜像中，而不需要在容器内执行apt-get upgrade进行更新。因为在隔离的环境下，如果更新时试图修改init或改变容器内的设备，很可能导致更新失败。同时这样的做法，还可能会产生不一致的镜像，因为在容器内升级可能导致运行应用程序的依赖程序版本跟镜像中的不一致。
如果需要更新基础镜像，可以让上游的基础镜像更新，以确保构建的一致性。

**使用小镜像**

尽量使用轻量级的基础镜像，例如debian:jessie，alpine linux作为基础镜像。

**使用指定标签**

基础镜像非常重要。Dockerfile中FROM指令应该始终包含依赖的基础镜像完整的仓库名和标签，比如使用FROM debian:jessie而不仅是FROM debian，后者会默认使用latest的标签。

**常用指令组合**

apt-get update 和apt-get install 应该进行组合，同时应使用\来分割多行进行安装，提高可读性。

{% highlight c %}

#Dockerfile for https://index.docker.io/u/crosbymichael/python/ 
FROM debian:jessie

RUN apt-get update && apt-get install -y \
git \
libxml2-dev \
python \
build-essential \
make \
gcc \
python-dev \
locales \
python-pip

RUN dpkg-reconfigure locales && \
locale-gen C.UTF-8 && \
/usr/sbin/update-locale LANG=C.UTF-8

ENV LC_ALL C.UTF-8

{% endhighlight %}

层和缓存都是很不错的，不要害怕使用多层，因为有缓存的存在。同时应该尽量使用上游的包。

**使用自己的基础镜像**

如果需要一个基础镜像来运行python程序，有以下的Dockerfile示例。

{% highlight c %}

FROM crosbymichael/python

RUN pip install butterfly
RUN echo "root\nroot\n" | passwd root

EXPOSE 9191
ENTRYPOINT ["butterfly.server.py"]
CMD ["--port=9191", "--host=0.0.0.0"]

FROM crosbymichael/python

RUN pip install --upgrade youtube_dl && mkdir /download
WORKDIR /download
ENTRYPOINT ["youtube-dl"]
CMD ["--help"]

{% endhighlight %}

这样可以使得基础镜像非常小，并把精力集中的自己的应用程序中。

**参考资料**

- [dockerfile-best-practices](http://crosbymichael.com/dockerfile-best-practices.html)
- [dockerfile-best-practices2](http://crosbymichael.com/dockerfile-best-practices-take-2.html)
- http://dockone.io/article/131
- http://dockone.io/article/132