---
title:  "SORT"
date:   2017-03-30 13:57:02
categories: LINUXSHELL
---

**sort作用**

排序sort可以对文本文件或stdin进行排序操作。

**语法**

{% highlight c %}

\#sort [OPTION] FILE1 FILE2 …

{% endhighlight %}


**示例**

- a. 按数字进行排序

{% highlight c %}

\# sort -n file.txt

{% endhighlight %}

- b. 按倒序进行排序

{% highlight c %}

\# sort -r file.txt

{% endhighlight %}

- c. 按月份进行排序

{% highlight c %}

\# sort -M file.txt

{% endhighlight %}

- d. 合并两个排过序的文件，而且不需要对合并后的文件再进行排序

{% highlight c %}

\# sort -m sorted1 sorted2

{% endhighlight %}


- e. 测试文件是否排序

{% highlight c %}

\# sort -C file.txt

{% endhighlight %}

- f.按key进行排序

{% highlight c %}

\#cat data.txt
1 mac 200
2 winxp 400
3 bsd 5634
\#sort -kn 2 data.txt

{% endhighlight %}

- g.忽略前导空格字符

{% highlight c %}

\# sort -bd unsorted.txt

{% endhighlight %}

备注: -d 按字典进行排序

{% highlight c %}

\# echo "hello     wold     1234    6677" | tr -s ' '

{% endhighlight %}

- e. 将列数字相加

{% highlight c %}

\# cat sum.txt
1
2
3
4
5

\# cat sum.txt | echo $[ $(tr '\n' '+') 0 ]

{% endhighlight %}

上述命令用tr '\n' '+' 把换行替换成+，在最后的加号后补0，然后执行算数运算$[ 1+2+3+4+5+0 ],然后echo
