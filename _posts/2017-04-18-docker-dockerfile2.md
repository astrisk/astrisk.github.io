---
title:  "DOCKER-Dockerfile详解(二)"
date:   2017-04-18 17:00:19
categories: DOCKER-HACKATHON
---

在上一篇blog中，介绍了Dockerfile的用法等基础知识，这篇blog来详细看看Dockerfile中的指令。

## FROM

FROM为Dockerfile中的第一条指令，是用来设置base image,Dockerfile定义的所有指令将基于FROM指定的base image进行构建。

**语法**

{% highlight c %}

FROM <image>

{% endhighlight %}


{% highlight c %}

FROM <image:tag>

{% endhighlight %}


{% highlight c %}

FROM <image>@<digest>

{% endhighlight %}

备注：
- tag或digest是可填项，如果没写，则使用默认项latest

## RUN

RUN指令定义的命令会在image构建的过程中执行。RUN指令会在当前image的基础上新建一层执行命令，并提交结果，形成一个缓存层。同时该缓存层会被下一步的Dockerfile指令使用。

**语法**

RUN指令有两种格式，如下：

{% highlight c %}

RUN <command>

{% endhighlight %}


{% highlight c %}

RUN ["executable","param1","param2"]

{% endhighlight %}

> Note: 如果不想使用'/bin/sh'，而其他指定shell，例如'/bin/bash'，可以使用数组格式的RUN命令。
> RUN["/bin/bash","-c","echo hello"]


> Note:数组格式的RUN指令会被解析成JSON数组，所以在数组里面只能使用双引号("")，而不能使用单引号('')。

> Note:在JSON数组里面转义字符是必须的，比如以下指令解析时会报语法错误，RUN["c:\windows\system32\tasklist.exe"],

> 正确写法，应该是RUN ["c:\\windows\\system32\\tasklist.exe"]


## CMD

CMD指令跟RUN指令很相似，只不过RUN是在image被构建时执行并提交结果，而CMD指令是container启动时执行。如果在Dockerfile中定义多条CMD指令，只有最后一条CMD指令被运行。

CMD指令的语法，跟RUN指令一样

{% highlight c %}

FROM ubuntu
CMD ["/usr/bin/wc","--help"]

{% endhighlight %}

备注：在运行container可以run后指定参数来覆盖默认的CMD指令；如果不希望被覆盖，可以把ENTRYPOINT和CMD指令结合使用。（ENTRYPOINT指令也是可以使用特定项来覆盖）

## LABEL

LABEL指令是一个key-value对，LABEL指令会给image增加元数据。如果base image已经存在相同的LABEL，新的LABEL值会覆盖原有的值。

可以使用docker inspect命令来查看image的所有labels。

**语法**

{% highlight c %}

LABEL <key>=<value> <key>=<value> <key>=<value> ...

{% endhighlight %}

**示例**

{% highlight c %}

LABEL "com.example.vendor"="ACME Incorporated"
LABEL com.example.label-with-value="foo"
LABEL version="1.0"
LABEL description="This text illustrates \
that label-values can span multiple lines."

{% endhighlight %}

## EXPOSE

EXPOSE指令通知Docker在container在运行时，监听指定的端口。仅定义EXPOSE指令并不会使特定端口暴露给主机。在docker run可以使用-p选项把EXPOSE监听的所有端口暴露给主机，可以-p 后指定暴露的端口。

**语法**

{% highlight c %}

EXPOSE <port> [<port>...]

{% endhighlight %}


**示例**

{% highlight c %}

# PORT
EXPOSE 8080
EXPOSE 22
EXPOSE 8009
EXPOSE 8005
EXPOSE 8443

{% endhighlight %}

## ENV

ENV指令定义和设置变量，这些变量可以在Dockerfile的后续命令中使用。使用ENV定义的环境变量会被持久化下来，同时可以使用docker run --env <key>=<value>进行重新赋值。

**语法**

{% highlight c %}

ENV <key> <value>
ENV <key>=<value> ...

{% endhighlight %}


**示例**

{% highlight c %}

ENV myName="John Doe" myDog=Rex\ The\ Dog \
    myCat=fluffy

ENV myName John Doe
ENV myDog Rex The Dog
ENV myCat fluffy

{% endhighlight %}

## ARG

ARG指令定义变量，在使用docker build时可以通过--build-arg <varname>=<value> 标识对ARG变量进行赋值。比如在编写nodejs基础镜像的Dockerfile时，就可以把VERSION定义成ARG，在docker build时指定具体要构建的nodejs版本。

**示例**

{% highlight c %}

FROM resin/rpi-raspbian
ARG NODE_VER
ADD ./${NODE_VER:-node-v5.9.1-linux-armv7l} /
RUN ln -s /${NODE_VER} /node
ENV PATH=/node/bin:$PATH
CMD ["node"]

{% endhighlight %}

在Dockerfile中指定了一个默认值，如果构建image未指定NODE_VER，就使用默认值。同时也可以在构建image时指定NODE_VER值，如下：

{% highlight c %}

docker build --build-arg NODE_VER=node-v5.9.0-linux-armv7l .

{% endhighlight %}

## ADD

ADD指令把文件，目录（可以是URL指定的网络上的目录，文件）拷贝到image文件系统的目标位置上。

备注：
- 如果源路径是以/结尾，docker会认为是一个目录，并把源文件拷贝到该目录下。如果目标目录不存在，则自动创建目标目录。
- 如果源路径是个文件，且不以/结尾，docker会认为是一个文件。如果目标路径不存在，会把文件拷贝到目标路径下。如果目标文件存在，则会覆盖文件内容。如果目标路径是个存在的目录，则把文件拷贝到目标目录下。
- 如果源文件是归档，压缩文件，ADD指令在拷贝的同时自动进行解压展开，支持的压缩格式gzip, bzip2 ,xz。
- ADD指令添加到image的文件/目录权限为755，uid,gid都为0.
- 源文件可以使用模式匹配写法，例如: ADD home* /mydir/  ;  ADD hom?.txt  /mydir/
- 目标目录可以用绝对路径，也可以用相对路径（相对于WORKDIR）

## COPY

COPY指令跟ADD指令相似，跟ADD不同的是，COPY仅支持本地文件的拷贝，不支持URL和解压。


## VOLUME

VOLUME指令在container中创建一个挂载点，用来挂载主机或其他container的文件系统。

**语法**

{% highlight c %}

VOLUME ["/data"]

{% endhighlight %}


**示例**

{% highlight c %}

FROM ubuntu
RUN mkdir /myvol
RUN echo "hello world" > /myvol/greeting
VOLUME /myvol

{% endhighlight %}

在docker run 时可以把本地的donwload目录挂载到

{% highlight c %}

docker run -it -v /home/Downloads:/myvol ubuntu64 /bin/bash

{% endhighlight %}

{% highlight c %}

docker run -it -v /home/Downloads:/myvol:ro ubuntu64 /bin/bash

{% endhighlight %}

备注：
- 默认container对于挂载目录是可读可写权限，如果须要指定只读，可指定权限为ro。
- -v 指定的本地目录，须使用绝对路径。

## USER

USER指令用来指定运行Dockerfile定义的RUN,CMD,ENTRYPOINT指令定义命令的用户，可以使用用户名或UID。当服务不需要管理员权限时，可以通过该命令指定运行用户。并且可以在之前创建所需要的用户，例如： RUN groupadd -r postgres && useradd -r -g postgres postgres 。要临时获取管理员权限可以使用 gosu ，而不推荐 sudo 

**示例**

{% highlight c %}

USER apache

{% endhighlight %}

## WORKDIR

WORKDIR指令用来设置运行RUN,CMD,ENTRYPOINT指令时所在的工作目录。如果WORKDIR设置的目录不存在，则会创建。WORKDIR指令在Dockerfile中可以多次设置。如果WORKDIR设置时使用的是相对路径，新的路径是相对于上一条WORKDIR指令设置的路径。

**示例**

{% highlight c %}

WORKDIR /usr/local/sonarqube

{% endhighlight %}

## ENTRYPOINT
