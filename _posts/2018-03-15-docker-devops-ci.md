---
title:  "容器化DEVOPS-弹性CI平台系列前言"
date:   2018-03-15 08:37:00
categories: kubernetes-docker
---

最近跟朋友聊这两年自己利用kubernetes-docker在企业做的CI/CD平台，朋友不甚了解，建议写成blog，而且自己也想把这一整套系统/技术整理出来，做些总结。这些blog都会以问题-方案的思路来做，希望这样更能够让大家理解为什么要这样做，以及如果实现。

**问题**

一般企业的技术栈都比较多，比如常见有PHP，Java，node,Java android,GOlang等等。而我们在搭建内部CI的时候，大部分都是用master-多slave的分布式架构，同时为了提高资源利用率，一台slave都会部署多种技术栈的构建/依赖环境。这样就需要在新的服务器上安装所有的依赖环境，越来越多复杂的环境会导致我们的CI环境维护起来比较麻烦，比如会造成相互之间的干扰，升级一个把另一个给搞坏了。

**解决方案**

CI还是选择开源的Jenkins来做。但是换种方式，就是把整个CI的各个部分，master和各种slave做成一个个独立的docker images，比如Golang有自己的slave docker images ,node有自己的slave images。这样每种技术栈单独维护自己的构建环境docker images，构建的slave如果升级只需要自己的docker images。然后把整个的CI系统放到kubernetes中，让kubernetes来做容器的调度，当没有构建jobs触发时，只运行一个Jenkins master。当某个环境的jobs被触发，kubernetes会自动拉取对应环境的docker images，并启动container,并执行job，而完成job后kubernetes销毁container，回收资源。

**实现**

下面先把这套docker化的CI系统实现步骤列出来，然后按照步骤一步步搭建起来。

1. 搭建kubernetes集群（不在这里介绍）
2. [部署内网docker register](https://astrisk.github.io/kubernetes-docker/2018/03/13/Harbor-Install-And-Https-Configure/)
3. [部署kubernetes共享存储nfs服务器](https://astrisk.github.io/kubernetes-docker/2018/03/11/kubernetes-%E5%AD%98%E5%82%A8NFS%E9%83%A8%E7%BD%B2/)
4. 构建Jenkins master base images
5. 把Jenkins master部署到kubernetes,并安装相关插件
6. 构建Jenkins slave snlp base images
7. 在第5步的基础上，构建各个技术栈的base images。比如node snlp images
8. 在Jerkins master UI中配置kubernetes和相应snlp
9. 在Jerkins master UI中新建测试job，测试

