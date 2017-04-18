---
title:  "DOCKER-DOCKERFILE详解二"
date:   2017-04-18 14:00:19
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

