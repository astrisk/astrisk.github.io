---
title:  "SHELL-提取文件名和文件扩展名"
date:   2017-03-30 15:07:02
categories: LINUXSHELL
---

**功能**

经常需要提取文件名，或者以xx后缀的扩展名，使用shell可以很方便得实现这个功能。

**提取文件名**

- 原理

使用%操作符，\${VAR%.*}
从$VAR中删除位于%右边的通配符(.*)所匹配的的字符串，通配符从右向左进行匹配
%属于非贪婪操作，只匹配最短结果，另一个操作符%%，属于贪婪操作，会匹配符合条件的最长结果。

- 示例

{% highlight c %}

$ VAR=hack.fun.book.txt
$ echo ${VAR%.*} //结果是hack.fun.book
$ echo ${VAR%%.*} //结果是hack

{% endhighlight %}

**提取扩展名**

- 原理

使用$操作符，${VAR$*.}
从$VAR中删除位于$右边的通配符(*.)所匹配的的字符串，通配符从左向右进行匹配

- 示例

{% highlight c %}

$ VAR=hack.fun.book.txt
$ echo ${VAR$*.}  //结果fun.book.txt
$ echo ${VAR$$*.} //结果是txt

{% endhighlight %}
