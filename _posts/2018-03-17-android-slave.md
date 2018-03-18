---
title:  "容器化DEVOPS-弹性CI平台系列之gradle android自动化"
date:   2018-03-18 22:37:00
categories: kubernetes-docker
---

在以前的blog中，已经把SoapUI自动化放到我们的弹性Jenkins集群中，现在我们再实现一个业务需求：在弹性Jenkins集群中利用gradle运行Android自动化。

**基础镜像**

都知道Android的基础构建环境是Java+Android SDK，以前用ant来做构建工具，现在基本上都已经使用gradle这个第三代的构建工具。为了遵循我们的docker images构建原则，先构建通用的基础镜像。
这次我们换成ubuntu Linux来作为base images，下面是ubuntu-jnlp-slave的Dockerfile


{% highlight c %}

FROM jenkinsci/slave:2.62

MAINTAINER Qiusheng Zhang <fang_ruo@163.com>

USER root

RUN echo "Asia/Shanghai" > /etc/timezone && \
    dpkg-reconfigure -f noninteractive tzdata
USER jenkins

WORKDIR /home/jenkins

COPY jenkins-slave /usr/local/bin/jenkins-slave

ENTRYPOINT ["jenkins-slave"]

{% endhighlight %}

enterpoint文件如下（直接使用官方的）

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

**Gradle android jnlp slave**

 现在我们在ubuntu-jnlp-slave的基础上来构建我们的Android slave。这里要注意以下两个点：

 1. gradle跟maven类似，构建时会把网络下载的依赖包，存储到本地的.gradle目录，用来提高构建速度。我们需要把.gradle做成挂载卷，存储在本地。
 2. Android 构建依赖于Android SDK，如果我们把SDK放在docker里，整个docker images会变得无比庞大（几十G）。我的选择是把Android SDK也做成挂载卷存储到本地。
 3. Android依赖于一些类库，在某些系统上安装比较麻烦，例如alpine。所以我这里选择ubuntu系统作为基础系统。

 下面是我的Android slave Dockerfile

{% highlight c %}

 FROM fangruo/jenkins-slave-ubuntu-base
MAINTAINER Qiusheng Zhang <fang_ruo@163.com>

ENV ANDROID_HOME /home/jenkins/android-sdk

USER root

RUN dpkg --add-architecture i386 && apt-get update && apt-get install -y libc6:i386 libncurses5:i386 libstdc++6:i386 lib32z1 && apt-get clean

USER jenkins
RUN mkdir /home/jenkins/android-sdk
VOLUME /home/jenkins/android-sdk
ENV PATH ${ANDROID_HOME}/tools:${ANDROID_HOME}/platform-tools:$PATH

RUN mkdir /home/jenkins/.gradle
VOLUME /home/jenkins/.gradle

RUN mkdir /home/jenkins/android-target
VOLUME /home/jenkins/android-target

RUN mkdir /home/jenkins/.android/ && touch /home/jenkins/.android/repositories.cfgs 

ENV GRADLE_USER_HOME /home/jenkins/.gradle

WORKDIR /home/jenkins

{% endhighlight %}

把这些docker构建出来推送到register，就可以在我们的Jenkins集群中使用了。不过在Jenkins运行前，还需要做一件事情，那就是在Linux 服务器上安装Android SDK

**Android SDK install on Linux**

由于我们的集群的挂载卷全部都是放在NFS服务器上，所以我们需要在nfs服务器上安装Android，并且以nfs共享目录的方式共享出来。

1. 在https://developer.android.com/studio/index.html网站下载Android命令行工具，例如：Linux版sdk-tools-linux-3859397.zip
2. unzip 解压然后cd 到{androidsdk-dir}/tools
3. 运行以下命令安装对应版本的sdk和build-tools

{% highlight c %}

$ echo y | ./android update sdk --no-ui --all --filter android-23,android-22,android-21,android-20,android-19,android-17,android-15,android-9

$ echo y | ./android update sdk --no-ui --all --filter   build-tools-27.0.1,build-tools-23.0.2,build-tools-23.0.1,build-tools-22.0.1,build-tools-21.1.2,build-tools-20.0.0,build-tools-19.1.0,build-tools-26.0.2

{% endhighlight %}

**运行Android jobs**

一切都OK了，就可以在Jenkins UI配置Android slave和创建Jenkins jobs。（具体配置参考之前的几篇blog，这里就写怎么配置了）。下图是我的Android job构建debug包的log图

![android]({{ site.url }}/images/jenkins/android.png)



![maven4]({{ site.url }}/images/jenkins/maven4.png)

