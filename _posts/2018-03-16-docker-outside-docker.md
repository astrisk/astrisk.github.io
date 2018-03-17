---
title:  "容器化DEVOPS-弹性CI平台系列之docker build docker"
date:   2018-03-16 07:37:00
categories: kubernetes-docker
---

由于我们所有的技术都是基于docker,所以会经常构建docker images，之前我们都是手工构建，并推送到docker register。现在是时候把构建docker images做成自动化，并集成的CI管道。懒人的目标是：提交完代码（或者merge完代码），CI系统就会自动拉取代码，构建包括镜像，推送到docker images到docker register。
有了docker images，就可以持续部署了。关于部署，我们会用到别的技术，暂时不在这里介绍。

**选择构建docker 方式**

用docker来构建docker有以下两种方式或技术：
- docker in docker
- docker outside docker
那么这两种docker 构建docker的技术有什么区别？两种不同技术的利弊是什么呢？应该选择哪种比较合适？这些答案都可以从这篇文章里面找到[docker build docker](http://blog.teracy.com/2017/09/11/how-to-use-docker-in-docker-dind-and-docker-outside-of-docker-dood-for-local-ci-testing/)

我们主要是CI中构建docker images，所以我选择用docker outside docker的方式来构建docker images。

**DOD-jenkins slave base**

现在我们需要在jnlp slave的基础上做一个能够构建docker的Jenkins jnlp slave。具体细节大家可以自己进入docker container里，尝试install docker，我这里直接给出Dockerfile

{% highlight c %}

$ vim Dockerfile

FROM jenkinsci/slave:2.62

MAINTAINER Qiusheng Zhang <fang_ruo@163.com>

USER root

RUN echo "Asia/Shanghai" > /etc/timezone && \
    dpkg-reconfigure -f noninteractive tzdata && \
    apt-get update && \
    apt-get -y install \
     apt-transport-https \
     ca-certificates \
     curl \
     gnupg2 \
     software-properties-common && \
     curl -fsSL https://download.docker.com/linux/$(. /etc/os-release; echo "$ID")/gpg | apt-key add - && \
     add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/$(. /etc/os-release; echo "$ID") \
   $(lsb_release -cs) \
   stable" && \
     apt-get update && apt-get -y install docker-ce

VOLUME /var/run/docker.sock

WORKDIR /home/jenkins

COPY jenkins-slave /usr/local/bin/jenkins-slave

ENTRYPOINT ["jenkins-slave"]

{% endhighlight %}

**jenkins-slave文件**

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

**Jenkins中配置DOD slave**

具体如何配置，在之前的blog，Jenkins 集成kubernetes有介绍，具体如下图
![dood]({{ site.url }}/images/jenkins/dood.png)

**测试**

在Jenkins中建个测试的job，简单运行docker run hello-world 或docker ps命令，如果运行成功，就表示已经可以用CI系统来构建docker。下图我直接从gitlab repo拉代码，利用Jenkins 构建了docker images,并push到私有的docker register。

![dod1]({{ site.url }}/images/jenkins/dod1.png)

![dod2]({{ site.url }}/images/jenkins/dod2.png)