---
title:  "SHELL-UNIQ"
date:   2017-03-30 14:20:13
categories: LINUXSHELL
---

**uniq 作用**

通过消除重复的内容，从给定输入（stdin或命令行参数文件）找出单一的行。uniq只能用于排过序的文件，所以uniq经常跟sort一起结合使用。

**语法**

{% highlight c %}

$ sort unsorted.txt | uniq

{% endhighlight %}


***示例***

- a. 只显示唯一的行

{% highlight c %}

$ sort unsorted.txt | uniq  或 $uniq sorted.txt

{% endhighlight %}


- b. 统计各行出现的次数

{% highlight c %}

$ sort unsorted.txt | uniq -c

{% endhighlight %}

- c. 找出重复的行

{% highlight c %}

$ sort unsorted.txt | uniq -d

{% endhighlight %}


- d. 使用-s跳过前N个字符，-w指定用于比较的最大字符数

{% highlight c %}

$ sort data.txt | uniq -s 2 -w 2

{% endhighlight %}

- e. 跟xargs结合，删除file.txt定义的文件

{% highlight c %}

$ sort file.txt | uniq -z | xargs -0 rm

{% endhighlight %}