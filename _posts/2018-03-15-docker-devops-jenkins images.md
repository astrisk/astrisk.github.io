---
title:  "容器化DEVOPS-弹性CI平台系列之Jenkins 基础images构建"
date:   2018-03-15 12:37:00
categories: kubernetes-docker
---

docker images 制作比较简单，只要熟悉一下相关的命令多操作几遍就行。具体可以参考[DOCKER-Dockerfile详解(一)](https://astrisk.github.io/docker-hackathon/2017/04/17/docker-dockerfile/),[DOCKER-Dockerfile详解(二)]（https://astrisk.github.io/docker-hackathon/2017/04/18/docker-dockerfile2/）以及[DOCKER-DOCKERFILE最佳实践](https://astrisk.github.io/docker-hackathon/2017/04/21/dockerfile-bestpractices/)。

这里简单提下最基本的几点：

1.不直接使用第三方docker images.所有的images自己构建，并且基础镜像使用权限或官方镜像。
2.尽量使用最小的docker base images。个人主要使用使用alpine Linux或ubuntu Linux。基本不使用CentOS。关于alpine，参考 [alpine linux](https://www.alpinelinux.org/about/) 或 [infoq 这篇文章](http://www.infoq.com/cn/news/2016/01/Alpine-Linux-5M-Docker)
3.docker container最小权限,尽量不用root

按照以上几点，我们就可以编写需要的几个Jenkins Dockerfile。

**Jenkins master**

{% highlight c %}

FROM jenkins:2.46.3-alpine

MAINTAINER Qiusheng Zhang <fang_ruo@163.com>

USER root

RUN apk --no-cache add tzdata  && \
    ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
    echo "Asia/Shanghai" > /etc/timezone

USER jenkins

{% endhighlight %}


**Jenkins jnlp slave base**

{% highlight c %}

FROM jenkinsci/slave:2.62-alpine

MAINTAINER Qiusheng Zhang <fang_ruo@163.com>

USER root

RUN apk --no-cache add tzdata  && \
    ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
    echo "Asia/Shanghai" > /etc/timezone

USER jenkins

WORKDIR /home/jenkins

COPY jenkins-slave /usr/local/bin/jenkins-slave

ENTRYPOINT ["jenkins-slave"]

{% endhighlight %}

**jnlp base entrypoint 文件**

FROM jenkinsci/slave:2.62-alpine

MAINTAINER Qiusheng Zhang <fang_ruo@163.com>

USER root

RUN apk --no-cache add tzdata  && \
    ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
    echo "Asia/Shanghai" > /etc/timezone

USER jenkins

WORKDIR /home/jenkins

COPY jenkins-slave /usr/local/bin/jenkins-slave

ENTRYPOINT ["jenkins-slave"]

**jnlp base image entrypoint 文件：jenkins-slave**

{% highlight c %}
#!/usr/bin/env bash

# The MIT License
#
#  Copyright (c) 2015, CloudBees, Inc.
#
#  Permission is hereby granted, free of charge, to any person obtaining a copy
#  of this software and associated documentation files (the "Software"), to deal
#  in the Software without restriction, including without limitation the rights
#  to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
#  copies of the Software, and to permit persons to whom the Software is
#  furnished to do so, subject to the following conditions:
#
#  The above copyright notice and this permission notice shall be included in
#  all copies or substantial portions of the Software.
#
#  THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
#  IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
#  FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
#  AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
#  LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
#  OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
#  THE SOFTWARE.

# Usage jenkins-slave.sh [options] -url http://jenkins SECRET SLAVE_NAME
# Optional environment variables :
# * JENKINS_TUNNEL : HOST:PORT for a tunnel to route TCP traffic to jenkins host, when jenkins can't be directly accessed over network
# * JENKINS_URL : alternate jenkins URL

if [ $# -eq 1 ]; then

    # if `docker run` only has one arguments, we assume user is running alternate command like `bash` to inspect the image
    exec "$@"

else

    # if -tunnel is not provided try env vars
    if [[ "$@" != *"-tunnel "* ]]; then
        if [ ! -z "$JENKINS_TUNNEL" ]; then
            TUNNEL="-tunnel $JENKINS_TUNNEL"
        fi
    fi

    if [ ! -z "$JENKINS_URL" ]; then
        URL="-url $JENKINS_URL"
    fi

    if [ -z "$JNLP_PROTOCOL_OPTS" ]; then
        echo "Warning: JnlpProtocol3 is disabled by default, use JNLP_PROTOCOL_OPTS to alter the behavior"
        JNLP_PROTOCOL_OPTS="-Dorg.jenkinsci.remoting.engine.JnlpProtocol3.disabled=true"
    fi

    exec java $JAVA_OPTS $JNLP_PROTOCOL_OPTS -cp /usr/share/jenkins/slave.jar hudson.remoting.jnlp.Main -headless $TUNNEL $URL "$@"
fi

{% endhighlight %}

- master和jnlp base 最为简单，直接在官方镜像基础上更改下时区（改为上海）
- 以后业务的slave镜像都是基于自己jnlp base镜像来做
- 把这两个Jenkins build出来，并推送到私有register