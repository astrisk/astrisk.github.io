---
title:  "用Shell生成临时文件名"
date:   2017-03-30 13:27:28
categories: LINUXSHELL
---

**功能**

在测试的时候，经常要生成一些测试文件，有时候想要生成一些随机的文件名。

**使用当前进程ID作为临时文件名**

- 示例

{% highlight c %}

temp_file="/tmp/var.$$"

{% endhighlight %}

备注: .$$作为添加的后缀会被扩展成当前运行脚本的进程ID

**提取扩展名**

- 原理
	- 使用#操作符，${VAR#*.}
	- 从$VAR中删除位于#右边的通配符(*.)所匹配的的字符串，通配符从左向右进行匹配


- 示例

{% highlight c %}

VAR=hack.fun.book.txt
echo ${VAR#*.}  //结果fun.book.txt
echo ${VAR##*.} //结果是txt

{% endhighlight %}
