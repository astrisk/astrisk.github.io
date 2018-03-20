---
title:  "容器化DEVOPS-弹性CI平台系列之Jenkins Gitlab集成"
date:   2018-03-19 08:37:00
categories: kubernetes-docker
---

我们已经部署好了Jenkins cluster，现在是时候把Jenkins和代码做集成。这样开发提交代码，就可以自动触发Jenkins的自动化构建jobs。
我这里的代码仓库选用的是Gitlab（现在很多公司都部署了内网的Gitlab）.

**Jenkins插件安装配置**

- Jenkins中Gitlab插件安装

在Jenkins左上角： Jenkins -> 系统管理 -> 插件管理页面, 搜索中输入：gitlab, 
然后找到gitlab插件安装即可。

![jenkins2]({{ site.url }}/images/jenkins/jenkins2.png)

![gitalb]({{ site.url }}/images/jenkins/gitalb.png)

- Jenkins中配置Gitlab

jenkins中Gitlab的配置在Jenkins ->系统管理 -> 系统设置

![gitalb2]({{ site.url }}/images/jenkins/gitalb2.png)

完成后，可以点击TestConnection，如果显示Success，即可。

**Jenkins新建job**

在JenkinsUI中新建一个job，在构建触发器部分，勾选：Build when a change is pushed to GitLab. GitLab CI Service URL: http://x.x.x.x:8080/project/test（拷贝这个URL，等会配置gitlab用）,保存退出。

![jenkins3]({{ site.url }}/images/jenkins/jenkins3.png)

**Gitlab repo配置**


Jenkins和Gitalb的集成，是通过Gitlab的webhooks触发来Jenkins job的。配置webhook只需要以下几步：

- 需要生成一个Secret Token

![token]({{ site.url }}/images/jenkins/token.png)

- 所以需要进入对应的代码repo中，配置触发Jenkins job的webhook（需要gitlab repo的master权限）

![gitalb3]({{ site.url }}/images/jenkins/gitalb3.png)

![jenkins4]({{ site.url }}/images/jenkins/jenkins4.png)


**Gitlab触发策略**

这里只是讲如何配置gitlab触发Jenkins job，后面会专门写一篇blog来深入介绍如何根据git工作流来实现不同的触发策略。




