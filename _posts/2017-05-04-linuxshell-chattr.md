---
title:  "SHELL-CHATTR"
date:   2017-05-04 15:10:27
categories: LINUXSHELL
---

**功能**
chattr命令用来修改文件属性，命令的使用跟chmod命令相似。

chattr命令有一个特别的选项i。chattr +i filename 将使得这个文件被标识为永远不变。这个文件不能被修改，连接，删除，**即使root用户也不行**。这个文件属性只能被root设置和删除。类似的a选项将会吧文件标记为只能追加数据。

不是所有的文件系统都支持这个命令，在CentOS 7-ext4中试验支持。

{% highlight c %}

\# chattr +i file1.txt

# 使用chattr命令设置的属性，不会在ls -la中显示，用lsattr 显示相关属性

\# lsattr file1.txt

{% endhighlight %}
