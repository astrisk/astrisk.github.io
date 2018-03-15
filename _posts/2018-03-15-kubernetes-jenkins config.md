---
title:  "容器化DEVOPS-弹性CI平台系列之Jenkins kubernetes集成"
date:   2018-03-15 23:37:00
categories: kubernetes-docker
---

之前的blog中已经把Jenkins master 部署到kubernetes集群中，在浏览器输入http://nodeip:38080 就可以看到如下界面

![start]({{ site.url }}/images/jenkins/getstart.png)

在nfs服务器或者进入Jenkins master的container中获取secret，就可以继续安装通用插件

![secrets]({{ site.url }}/images/jenkins/secrets.png)


**安装kubernetes plugin**

进入Jenkins管理界面，左上角， Jenkins -> 系统管理 -> 插件管理 -> 可选插件 tab,在搜索中输入“kubernetes”,安装插件：ElasticBox Kubernetes CI/CD。安装完成后，重启Jenkins。

![k8splugin]({{ site.url }}/images/jenkins/k8splugin.png)

**配置kubernetes cloud**

在Jenkins管理界面，左上角Jenkins -> 系统管理 -> 云 -> 新增一个云

![cloud]({{ site.url }}/images/jenkins/cloud.png)

- Endpoint URL: kubernetes master endpoint

**配置slave**

在 Configuration of Pod slave nodes处 增加 一个slave，并保存,如下图

![slave]({{ site.url }}/images/jenkins/slave.png)

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


**测试**

在Jenkins中新建一个job。在job配置->General中勾选 “Restrict where this project can be run”，并  Label Expression 中填写刚新增的slave的label（对应上图的label maven3.3.9），保存后触发执行job。
可以看到一开始显示没有slave在线，job处于挂起状态。过一会有一个在线的slave，并执行job。当job完成后，slave被销毁，资源被kubernetes回收。

![slave2]({{ site.url }}/images/jenkins/slave2.png)
![slave3]({{ site.url }}/images/jenkins/slave3.png)

到此基于kubernetes docker的弹性伸缩CI Jenkins集群就已经部署成功。后面我会做一个业务的slave在构建maven项目执行SoapUI自动化case。并且做一个构建docker的slave来自动化构建docker images.
而且后面还会把代码扫描平台，持续部署平台也放到kubernetes中来，最后做到提交代码后，自动构建，扫描代码，运行自动化case，自动部署等等。而一切都是基于kubernetes和docker,并且完全使用开源的工具链来打造的。