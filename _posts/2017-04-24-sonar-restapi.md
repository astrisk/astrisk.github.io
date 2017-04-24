---
title:  "质量可视化之SONARQUBE-RESTAPI"
date:   2017-04-24 10:40:07
categories: SONARQUBE
---

**背景**

之前为了在公司推质量可视化项目，把公司持续集成平台，sonarqube平台的代码项目过程质量数据做收集并集中展示，以此来使得项目过程质量更加透明和可度量，以此来推动研发团队重视质量。
在这个过程中，涉及到几个系统的相关质量数据获取，其中SONARQUBE上的数据就是很大一块。但是由于SONAR相关的文旦资料看起来有些晦涩，搞了段时间才弄明白。关于webapi，gitlab的相关文档就比sonar要好很多，而且每个api都有对应的示例，简洁明了。


**SONAR API**

- SONARQUBE上个指标数据主要通过 /api/measures/component API接口来获取。
- [详细的指标文档](https://docs.sonarqube.org/display/SONAR/Metric+Definitions)

**示例**

通过以下方式可以获取到sonarqube某项目的测试覆盖率，代码重复度，代码复杂度等指标，其中componentId为projectId


{% highlight c %}

http://local_sonarqube.com:9001/api/measures/component?componentId=AVcKM1HJvgI7BWJwTflx&metricKeys=ncloc,complexity,coverage,duplicated_lines_density

{% endhighlight %}


通过以下方式可以获取到sonarqube中的所有项目列表
{% highlight c %}

http://local_sonarqube.com:9001/api/projects

{% endhighlight %}


