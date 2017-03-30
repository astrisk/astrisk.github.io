---
title:  "SHELL-TR"
date:   2017-03-30 14:09:59
categories: LINUXSHELL
---


**tr作用**

tr可以对来自标准输入的字符进行替换，删除，以及压缩。由于可以将一组字符变成另一组字符，因而通常被称为转换(translate)命令

**语法**

{% highlight c %}

\$ tr [OPTION] set1 set2

{% endhighlight %}

将stdin的输入字符从set1映射到set2，并将其输出写入stdout。Set1和set2是字符或字符集。如果两个字符集的长度不相等，那么set2会不断重复其最后一个字符，直到长度与set2相同。如果set2的长度大于set1，那么在set2中超出set1长度的那部分字符则全部被忽略。

**示例**

- a. 将输入字符由大写转换成小写

{% highlight c %}

\$ echo "HELLO WHO IS THIS" | tr 'A-Z' 'a-z'

{% endhighlight %}

- b. 用tr删除字符

{% highlight c %}

\$ echo "Hello 123 world 456" | tr -d [[ :digit: ]]

{% endhighlight %}


- c. 字符集补集

{% highlight c %}

\$ echo hello 123 world 456 | tr -d -c '0-9 \n'

{% endhighlight %}

备注:以上删除除数字,空格，回车外的字符;-c 选项表示取补集

- d. 压缩字符

{% highlight c %}

\$ echo "hello     wold     1234    6677" | tr -s ' '

{% endhighlight %}

- e. 将列数字相加

{% highlight c %}

\$ cat sum.txt
1
2
3
4
5

\$ cat sum.txt | echo $[ $(tr '\n' '+') 0 ]

{% endhighlight %}

上述命令用tr '\n' '+' 把换行替换成+，在最后的加号后补0，然后执行算数运算$[ 1+2+3+4+5+0 ],然后echo