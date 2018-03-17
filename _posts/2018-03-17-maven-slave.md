---
title:  "容器化DEVOPS-弹性CI平台系列之maven slave运行SoapUI自动化用例"
date:   2018-03-17 20:37:00
categories: kubernetes-docker
---

在前面的blog中，我们已经在kubernetes中部署自动伸缩的Jenkins集群。既然基础设施已经齐全，那就让我们把业务的东西放进去，让我们的基础设施发挥作用。在这篇blog中，我们会做一个maven的slave，然后利用这个maven slave来运行我们的SoapUI自动化用例。

**jenkins jnlp maven slave**

用maven来构建Java程序时，需要先下载各种依赖的jar包，而且这个jar的量还比较大。为了利用maven的缓存，我们把.m2目录挂在在nfs服务器目录中，这样只需要在第一次构建时，从网络中下载依赖jar包，以后就可以利用本地存储的jar包，以此来提高CI的构建效率。
下面直接给出maven slave的Dockerfile

{% highlight c %}

FROM fangruo/jenkins-slave-apline-base
MAINTAINER Qiusheng Zhang <fang_ruo@163.com>

USER root

RUN apk add --no-cache curl tar bash

ARG MAVEN_VERSION=3.3.9
ARG USER_HOME_DIR="/home/jenkins"
ARG SHA=6e3e9c949ab4695a204f74038717aa7b2689b1be94875899ac1b3fe42800ff82
ARG BASE_URL=https://apache.osuosl.org/maven/maven-3/${MAVEN_VERSION}/binaries

RUN mkdir -p /usr/share/maven /usr/share/maven/ref \
  && curl -fsSL -o /tmp/apache-maven.tar.gz ${BASE_URL}/apache-maven-${MAVEN_VERSION}-bin.tar.gz \
  && echo "${SHA}  /tmp/apache-maven.tar.gz" | sha256sum -c - \
  && tar -xzf /tmp/apache-maven.tar.gz -C /usr/share/maven --strip-components=1 \
  && rm -f /tmp/apache-maven.tar.gz \
  && ln -s /usr/share/maven/bin/mvn /usr/bin/mvn

ENV MAVEN_HOME /usr/share/maven
ENV MAVEN_CONFIG "$USER_HOME_DIR/.m2"

VOLUME "$USER_HOME_DIR/.m2"


USER jenkins


{% endhighlight %}

**Jenkins配置maven slave**

在Jenkins管理界面左上角 Jenkins -> 系统管理 -> 系统设置 -> Kubernetes Cloud中增加slave

{% highlight c %}

apiVersion: "v1" 
kind: "Pod" 
metadata: 
  name: "maven-slave" 
  labels: 
    name: "maven-slave" 
spec: 
  containers: 
  - name: "maven-slave" 
    image: "fangruo/jenkins-snlp-maven:latest" 
    imagePullPolicy: IfNotPresent
    volumeMounts:
    - mountPath: /home/jenkins/.m2
      name: mavenhome
    env: 
    - name: "JENKINS_URL" 
      value: "http://jenkins.jenkins:8080"
  volumes:
    - name: mavenhome
      nfs:
        server: x.x.x.x
        path: "/home/nfs/.m2"  


{% endhighlight %}

![maven]({{ site.url }}/images/jenkins/maven.png)

**测试**

在Jenkins中新建一个job，在Restrict where this project can be run 中填写maven slave对应的label（我的是maven3.3.9）即可


![maven2]({{ site.url }}/images/jenkins/maven2.png)

![maven3]({{ site.url }}/images/jenkins/maven3.png)

![maven4]({{ site.url }}/images/jenkins/maven4.png)

