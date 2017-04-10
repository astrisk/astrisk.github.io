---
title:  "SHELL-压缩归档(二)"
date:   2017-04-10 09:23:29
categories: LINUXSHELL
---

**TAR作用**

tar是一个归档工具，把多个文件放到一个文件包里；同时tar命令可以跟压缩命令(gz，bz2等)结合使用，实现一次性压缩归档。

**语法**

{% highlight c %}

$ tar [OPTION] FILE

{% endhighlight %}

**常用选项**

- c: 创建新的归档文件
- f：指定归档后的文件名
- t: 列出tar包中包含的文件列表
- v: 输出归档过程
- x: 展开tar包中的文件
- A: 把tar文件追加到另一个tar文件中
- r：把文件追加到tar文件中
- d: diff文件系统的文件和tar包中的文件
- z: 调用gzip
- j: 调用bzip2
- J: 调用xz

**示例**

- a. 用foo创建归档文件

{% highlight c %}

$ tar -cf archive.tar foo

{% endhighlight %}

- b. 列出tar包中包含的文件列表

{% highlight c %}

$ tar -tvf archive.tar

{% endhighlight %}

- c. 展开tar包中的文件

{% highlight c %}

$ tar -xvf archive.tar

{% endhighlight %}

- d. diff文件系统的文件和tar包中的文件

{% highlight c %}

$ tar -dvf archive.tar

{% endhighlight %}

- e. 把文件追加到另一个tar文件中

{% highlight c %}

$ tar -tvf archive.tar file.txt

{% endhighlight %}

- f.解压和展开tar.gz文件

{% highlight c %}

$ tar -xzvf archive.tar.gz

{% endhighlight %}

- g.解压和展开tar.bip2文件

{% highlight c %}

$ tar -xjvf archive.tar.gz

{% endhighlight %}

- h. 解压和展开tar.bip2文件

{% highlight c %}

$ tar -xJvf archive.tar.xz

{% endhighlight %}

- i. 压缩并归档目录（含子目录/子子目录）

{% highlight c %}

$ tar -cJvf archive.tar.xz

{% endhighlight %}

- j. 创建压缩归档文件

{% highlight c %}

$ tar -czvf archive.tar.gz  ./dir

{% endhighlight %}

备注：指定选项时，选项顺序要注意，c要放在前面，如果这样写-vfcz 会报错