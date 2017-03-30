---
title:  "动态指定NPM源"
date:   2017-03-30 13:03:03
categories: NODE
---


**问题**

由于某种原因，在下载npm依赖包时，经常失败或者下载的时间超级长，导致build一直进行不了。之前都是使用默认的npm源，在一阵折磨后，尝试各种方式来解决这种问题。CI上，特别需要通过动态指定npm源，这样能够缩短build时长，提升构建效率。

**代码**

{% highlight c %}

npm install --registry https://registry.npm.taobao.org

{% endhighlight %}

备注：通过选项--registry动态指定npm源

