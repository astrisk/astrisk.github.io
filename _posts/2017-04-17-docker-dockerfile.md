---
title:  "DOCKER-DOCKERFILE详解"
date:   2017-04-17 17:55:09
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

Docker daemon从上而下一条条运行Dockerfile中的指令，如果指令运行成功，Docker daemon会提交构建结果，生成缓存。如果所有指令都运行成功，在构建结束前会生成一个最终的docker image，并输出相应的docker image id。同时Docker daemon会自动清理构建中使用的context。

每条指令都是独立运行（如果运行成功），会生成一个缓存的镜像。为了加速构建，docker会尽可能利用缓存的镜像。

## Format

Dockerfile的格式如下：

> \# comment
> INSTRUCTION arguments

Dockerfile指令不是大小写敏感，但是约定指令是建议大写。第一条指令必须是以FROM 开头指定一个基础镜像，如下：

> FROM ubuntu

> ENTRYPOINT ["top", "-b"]

> CMD ["-c"]

## Parser directives

Parser directive，即解析器指令，为可选项。解析器指令在构建过程中不会生成缓存层，也不会作为构建步骤在构建过程中显示。解析器指令作为一种特殊的注释。单个解析器指令只能被使用一次。

以下是支持的解析器指令
- escape

**escape**
escape指令是用来设置Dockerfile中的转移字符。如果没有设置，则使用默认的，默认的转移字符为 \.

在window环境中，以下Dockerfile在构建时会出现问题。

> FROM microsoft/nanoserver

> COPY testfile.txt c:\\

> RUN dir c:\

使用escape指令把转移符设置成 \` ,可以解决以上问题，Dockerfile内容如下

> \# escape=`

> FROM microsoft/nanoserver

> COPY testfile.txt c:\

> RUN dir c:\

## Environment replacement

在Dockerfile中可以定义和使用环境变量。在Dockerfile中，环境变量可以使用$var和${var}两种方式表示，其中${var}主要用在不带空格的变量名上，如${var}_bar.

${var}语法还支持标准bash修饰语，如下
- ${variable:-word}：如果设置variable值，采用设置的值，如果没有设置，最终值为word
- ${variable:+word}：如果设置variable值，则值为word,不然最终为空字符串。

在Dockerfile中，支持环境变量的指令如下

- ADD
- COPY
- ENV
- EXPOSE
- LABEL
- USER
- WORKDIR
- VOLUME
- STOPSIGNAL
