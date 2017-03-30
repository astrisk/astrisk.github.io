---
title:  "SHELL-生成临时文件名"
date:   2017-03-30 14:37:03
categories: LINUXSHELL
---

**功能**

在测试的时候，经常要生成一些测试文件，有时候想要生成一些随机的文件名。

**使用当前进程ID作为临时文件名**

- 示例

{% highlight c %}

$temp_file="/tmp/var.$$"

{% endhighlight %}

备注: .$$作为添加的后缀会被扩展成当前运行脚本的进程ID

**使用tempfile命令（Debian发行版）**

- 示例

{% highlight c %}

$temp_file=$(tempfile)

{% endhighlight %}

**使用随机数作为文件名**

- 示例

{% highlight c %}

$temp_file="/tmp/file-$RANDOM"

{% endhighlight %}