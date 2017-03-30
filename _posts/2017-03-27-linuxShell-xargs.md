---
title:  "SHELL-XARGS"
date:   2017-03-30 14:25:03
categories: LINUXSHELL
---


**xargs作用**

有些命令只能以命令行参数形式接收数据，而无法通过stdin接受数据流，在这种情况下，就没有办法使用管道来提供那些只能通过命令行参数才能提供的数据。
xargs命令的一个非常大的作用就是将标准输入的数据转换成命令行参数。xargs也可以将单行或多行文本输入转换成其他格式，例如将单行变成多行。


**语法**

{% highlight c %}

Command | xargs

{% endhighlight %}



** 示例**

- a. 将多行输入转换成单行输出

{% highlight c %}

$ cat example.txt | xargs

{% endhighlight %}

- b. 将单行输入转换成多行输出

{% highlight c %}

 $ cat examle.txt | xargs -n 2

{% endhighlight %}

- c. 指定分隔符

{% highlight c %}

$ echo "splitXsplitXsplitXsplit" | xargs -d x

{% endhighlight %}

- d. 指定分隔符并输出多行

{% highlight c %}

$ echo "splitXsplitXsplitXsplit" | xargs -d x -n 2

{% endhighlight %}

**读取stdin,将格式化参数传递给命令**

- 一次提供全部参数

{% highlight c %}

$ cat args.txt | xargs ./ccat.sh

{% endhighlight %}


- 一次提供一个参数


{% highlight c %}

$ cat args.txt | xargs -n 1 ./ccat.sh

{% endhighlight %}

- 一次提供多个参数

{% highlight c %}

$ cat args.txt | xargs -n 2 ./ccat.sh

{% endhighlight %}

- 命令后跟多个选项，并且参数是唯一可变的，如下命令：

{% highlight c %}

$ ./ccat.sh -p arg1 -1
$ ./ccat.sh -p arg2 -1
$ ./ccat.sh -p arg3 -1

{% endhighlight %}


- 利用xargs -I选项，具体命令如下：

{% highlight c %}

$ cat arg.txt | xargs -I {} ./ccat.sh -p {} -l

{% endhighlight %}

{}在命令执行时，会被替换成对应的参数，在这里起到占位符作用。

**结合find 和xargs**

{% highlight c %}

$ find . -type f -name ".txt" -print0 | xargs -0 rm -f

{% endhighlight %}

备注：无法预测find 命令输出的结果的定界符时 '\n'还是 ' '。很多文件名中间都有空格符，而xargs很可能会误认为它们是定界符。主要我们把find的输出作为xargs的输入，就必须将-print0与find结合，以字符null作为分割输出。Xargs -0将\0作为输入定界符


**统计当前目录下所有c程序文件的行数**

{% highlight c %}

$ find ./ -type f -name "*.c" -print0 | xargs -0 wc -l


{% endhighlight %}
