---
title:  "SHELL-CSPLIT"
date:   2017-03-30 14:35:03
categories: LINUXSHELL
---


**功能**

csplit工具跟split一样都是用来切割文件，但是csplit的功能更加强大，能够依据指定的条件和字符串匹配选项对文件进行分割。

**语法**

{% highlight c %}

$ csplit [OPTION] ... FILE PATTERN ...

{% endhighlight %}


**选项**

- b: 指定后缀名
- f: 指定前缀名
- s: 静默模式，不打印输出
- n: 指定分割后文件名后缀的数字个数
- /|REGEX|/: 用来匹配行，分割从匹配处开始。
- {*}: 表示根据匹配重复分割，知道文件未为止。可以用{整数}的形式来指定分割的次数。

**示例**

{% highlight c %}

$ csplit server.log /SERVER/ -n 2 -s {*} -f server -b "%2d.log" ; rm server00.log

{% endhighlight %}


