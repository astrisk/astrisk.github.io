---
title:  "SHELL-SPLIT"
date:   2017-03-30 14:36:03
categories: LINUXSHELL
---

**功能**

经常需要把一个大文件切分成多个小文件，split命令就是用来实现这个功能

**语法**

{% highlight c %}

$ split [OPTION] /path/to/file PREFIX

{% endhighlight %}

**选项**

1. -b:指定分割后的文件大小
2. -d:以数字作为文件名后缀
3. -a:指定文件名后缀长度
4. PREFIX：最后一个参数PREFIX，作为文件名前缀
5. -l:不按块大小分割文件，以行数来分割

**示例**

- 把文件分割成以10k大小的若干文件

{% highlight c %}

$ split -b 10k file.txt

{% endhighlight %}

- 把文件分割成10k大小，分割后的文件名以4位数字作为后缀

{% highlight c %}

$ split -b 10 data.txt -d -a 4

{% endhighlight %}


- 把文件分割成10k大小，分割后的文件名以4位数字作为后缀,同时前缀为split_file

{% highlight c %}

$ split -b 10k data.txt -d -a 4 split_file

{% endhighlight %}


- 按每10行分割文件

{% highlight c %}

$ split -l 10  file.txt

{% endhighlight %}



