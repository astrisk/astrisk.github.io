---
title:  "TEAMCITY-调整JVM_MEM参数"
date:   2017-04-12 09:14:11
categories: TEAMCITY
---

**背景**

前段时间公司的Teamcity Server经常报outofmemory,原因是teamcity的JVM内存参数值太小（安装时用的是默认值，没有更改过。）。之前让同事修改，但是由于她不知道怎么改，一直也没改，只好自己来。
本来是挺简单的，但是中间掉进坑，导致teamcity server service起不来，所以记录下详细的过程，以供后续参考。


**修改须知**

Teamcity的官方文档挺详细，但是太多，并且没有说明详细的傻瓜式的步骤，而且都是英文的，读起来有些费劲。总结下修改前必须清楚的细节。

- teamcity 可以支持32位和64位的jre,但是默认使用的是teamcity内置的jre(<Teamcity dir>/jre)
- 使用32位jre时，最大的jvm_mem_opts值最大位1.2g；如果超过这个值，需使用64位jre环境，不然会导致teamcity server service起不来。
- 使用64位jre时，jvm_mem_opts的值一般为32位时的两倍，最大支持10g。
- 如果使用64位jre,需要先安装64位jdk或jre，推荐用oracle的jdk。

**修改步骤**

- 停止服务器上teamcity server 和所有agent的service。
- 进入<teamcity-dir>,把jre目录，重命名为OLD_jre，这样teamcity就找不到内置的32位jre。
- 确定设置了JAVA_HOME环境变量，并且指向64位的JDK。
- 新增一个环境变量TEAMCITY_SERVER_MEM_OPTS，值为 -Xmx10g -XX:ReservedCodeCacheSize=512m
- 重启服务器
- 重启后，进入teamcity ui界面，进入Administration-Diagnostics-Memory usage可以看到JVM_MEM已经更新成修改后的值。
- 最后效果如下图
![teamcity-jvm]({{ site.url }}/images/teamcity-jvm.png)

