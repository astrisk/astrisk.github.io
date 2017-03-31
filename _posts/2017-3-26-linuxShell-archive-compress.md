---
title:  "SHELL-压缩归档"
date:   2017-03-31 09:28:50
categories: LINUXSHELL
---

**压缩归档概念**

首先来看下压缩和归档，对应的英文单词应该是compress和archive。压缩就是把体积大的物品压制成小的，对应到文件压缩，可理解为更换一种占用硬盘空间小的格式，具体压缩比则取决于压缩算法和选择的压缩率；归档可以理解为把一堆文件归在一起，归档后的文件大小可能比源文件还大。就像一堆散乱的东西，放到放到一个纸箱中，因为包含了纸箱，所以总大小反而更大。

**压缩率**

Linux平台有众多压缩工具，各种压缩工具的用法基本上都大同小异。每种压缩工具都有个压缩率的选项，取值范围都是[0,9]，0表示不压缩，9表示最大压缩率，默认都是6。压缩率越大，压缩后的文件越小，但是压缩过程中消耗的CPU时钟周期越长。

**压缩和解压**

Linux上的压缩工具都是成套的，一个压缩工具都会有一个配套的解压工具和不解压缩查看文件内容的工具。

**compress/uncompress**

compress/uncompress 是比较老的的压缩工具，压缩后的文件以.Z作为后缀，现在已经基本上很少见了。

**gzip/gunzip/zcat**

gzip/gunzip为比较常见的压缩工具，压缩文件以.gz做为后缀，gzip默认是不保留源文件（压缩后生成压缩文件并删除源文件），以下是gzip的常见用法。

- 语法

{% highlight c %}

$  gzip [OPTION] FILE ...

{% endhighlight %}

- 常用选项
	- d :解压，相当于gunzip
	- c :将输出写到标准输出，并保留源文件
	- r：递归的查找指定目录并压缩所有文件或解压文件
	- \$ : $ 代表压缩率，取值范围 [0,9]
	- l: 显示压缩文件列表

- 示例

**压缩但是不删除源文件**

{% highlight c %}
	
$gzip -c FILE > FILE.gz
	
{% endhighlight %}

**不解压，显示压缩文件列表**

{% highlight c %}

$gzip -l FILE.gz
	
{% endhighlight %}

**压缩目录中的文件**

{% highlight c %}

$gzip -r ./scripts

{% endhighlight %}

**解压文件**

{% highlight c %}

$gzip -d FILE.gz

{% endhighlight %}

备注：相当于gunzip

**不显示展开的前提下，查看文件内容**

{% highlight c %}

$zcat FILE.gz

{% endhighlight %}

**bzip2/bunzip2/bzcat**

摘抄百科的介绍:
	bzip2 是一个基于Burrows-Wheeler 变换的无损压缩软件，压缩效果比传统的LZ77/LZ78压缩算法来得好。它是一款免费软件。可以自由分发免费使用。它广泛存在于UNIX && LINUX的许多发行版本中。bzip2能够进行高质量的数据压缩。它利用先进的压缩技术，能够把普通的数据文件压缩10%至15%，压缩的速度和解压的效率都非常高！支持大多数压缩格式，包括tar、gzip 等等。
	压缩后的文件后缀为.bz2

- 语法

{% highlight c %}

$  bzip2 [OPTION] FILE ...

{% endhighlight %}

- 常用选项
	- d :解压，相当于bunzip2
	- k :默认压缩并删除源文件，用-k保留源文件
	- q：静默模式
	- \$ : $ 代表压缩率，取值范围 [0,9]
	- f: 压缩或解压时，若输出有同名文件，默认不会覆盖，如果指定此选项则覆盖

- 示例

**压缩并保留源文件**

{% highlight c %}

$  bzip2 -k FILE

{% endhighlight %}

**选择最高的压缩比**

{% highlight c %}

$  bzip2 -9 FILE

{% endhighlight %}

**解压**

{% highlight c %}

$  bzip2 -d FILE.bz2

{% endhighlight %}

**不显式展开的前提下查看文件内容**

{% highlight c %}

$  bzcat FILE.bz2

{% endhighlight %}

**xz/unxz/xzcat**

	XZ Utils 是为 POSIX 平台开发具有高压缩率的工具。它使用 LZMA2 压缩算法，生成的压缩文件比 POSIX 平台传统使用的 gzip、bzip2 生成的压缩文件更小，而且解压缩速度也很快。最初 XZ Utils 的是基于 LZMA-SDK 开发，但是 LZMA-SDK 包含了一些 WINDOWS 平台的特性，所以 XZ Utils 为以适应 POSIX 平台作了大幅的修改。XZ Utils 的出现也是为了取代 POSIX 系统中旧的 LZMA Utils。压缩的文件名后缀为.xz
    CentOS7已经默认安装了xz,如果是CentOS6或以下的，需要另外安装。

- 语法

{% highlight c %}

$  xz [OPTION] FILE ...

{% endhighlight %}

- 常用选项
	- d :解压，unxz
	- k :默认压缩并删除源文件，用-k保留源文件
	- q：静默模式
	- \$ : $ 代表压缩率，取值范围 [0,9]
	- f: 压缩或解压时，若输出有同名文件，默认不会覆盖，如果指定此选项则覆盖

- 示例

**压缩并保留源文件**

{% highlight c %}

$  xz -k FILE

{% endhighlight %}

**解压**

{% highlight c %}

$  xz -d FILE.xz

{% endhighlight %}

**不解压，查看文件内容**

{% highlight c %}

$  xzcat FILE.xz

{% endhighlight %}

**zip/unzip**
