---
title:  "如何完全更新Node.js"
date:   2017-03-30 13:13:32
categories: example
---

**背景**
Nodejs 更新速度非常快，几天没注意就发布了一个版本，而且还是大版本。对于有更新强迫症的人来说，这也是个不小的事。那下面来看看如何进行Nodejs更新

**更新NPM库**

{% highlight c %}
npm update -g
{% highlight c %}

**更新nodejs**

  - 源码方式更新：下载最新的源码，然后make install
  - npm方式：
	{% highlight c %}
		npm install -g n [stable/v0.10.26] # n代表n模块，后面可以跟最新稳定版或具体版本号
	{% highlight c %}