---
title:  "SCALA-单元测试"
date:   2017-04-21 15:52:54
categories: SCALA
---

**Scala项目结构**
- Scala项目结构一般是使用Maven约定的项目结构。

**IDEA安装Scala插件**
- File->Settings->Plugins->Install JetBrains plugin->输入”scala”->搜索->下载->安装
![scalaTest]({{ site.url }}/images/scala/scala2.png)

**配置Scalastyle**

- Scalastyle 官网：http://www.scalastyle.org/
- [Scalastyle 默认配置文件](http://www.scalastyle.org/scalastyle_config.xml)
- Scalastyle 默认60条规则说明：http://www.scalastyle.org/rules-0.7.0.html
- IDEA中配置规则：把scalastyle_config.xml 放到项目根目录或项目中.idea目录中

**Sacla UnitTest**

Scala 的单元测试框架主要有三个：ScalaTest、ScalaCheck和Spec2。其中ScalaTest是比较全面的测试框架。后面也主要使用ScalaTest编写Scala单元测试。以下介绍在各种构建环境下如何导入ScalaTest。同时我们公司开发测试都使用[ScalaTest](http://www.scalatest.org/)。不同的构建工具，导入ScalaTest的方式稍有区别。

- Maven,在pom.xml添加以下内容

{% highlight c %}

<dependency>
  <groupId>org.scalatest</groupId>
  <artifactId>scalatest_2.11</artifactId>
  <version>3.0.1</version>
  <scope>test</scope>
</dependency>

{% endhighlight %}

- Sbt

{% highlight c %}

libraryDependencies += "org.scalatest" % "scalatest_2.11" % "3.0.1" % "test"

{% endhighlight %}

**编写ScalaUnitTest**

- 选择测试风格
	- ScalaTest支持多种测试书写风格，我们使用推荐的FlatSpec编写单元测试，FeatureSpec编写验收类测试。[各种测试风格介绍](http://www.scalatest.org/user_guide/selecting_a_style)
- 定义测试抽象类，推荐命名为：unitSpec

{% highlight c %}

package com.mycompany.myproject

import org.scalatest._

abstract class UnitSpec extends FlatSpec with Matchers with
  OptionValues with Inside with Inspectors

{% endhighlight %}

- 使用自定义测试基类，编写测试类

{% highlight c %}

package com.mycompany.myproject

import org.scalatest._

class MySpec extends UnitSpec {
  // Your tests here
}

{% endhighlight %}
 
- 测试类的命名约束

	- 单元测试类的命名:被测试类名+UnitSpec,例如：生产代码中有个需要写单元测试的类为：InMemoryResitory.scala，则对应的单元测试类名为：InMemoryResitoryUnitSpec.scala

- 编写测试主题和测试
	在FlatSpec中，用主题来表达一组验证相同功能的测试。用it来表示单个测试，测试可以写成：”A should B”, ”A must B”, ” A can B”, 后面跟in闭包表示需要被测试的代码块。

	- Behavior of 表达主题
	
{% highlight c %}

behavior of "A Stack (with one item)"

it should "be non-empty" in {}

it should "return the top item on peek" in {}

it should "not remove the top item on peek" in {}

it should "remove the top item on pop" in {}

{% endhighlight %}

	- Shorthand notation 表达主题

{% highlight c %}

"A Stack (with one item)" should "be non-empty" in {}

it should "return the top item on peek" in {}

it should "not remove the top item on peek" in {}

it should "remove the top item on pop" in {}

{% endhighlight %}	 

	- 测试例子

{% highlight c %}

import collection.mutable.Stack
import org.scalatest._

class StackSpec extends FlatSpec {

  "A Stack" should "pop values in last-in-first-out order" in {
    val stack = new Stack[Int]
    stack.push(1)
    stack.push(2)
    assert(stack.pop() === 2)
    assert(stack.pop() === 1)
  }

  it should "throw NoSuchElementException if an empty stack is popped" in {
    val emptyStack = new Stack[String]
    assertThrows[NoSuchElementException] {
      emptyStack.pop()
    }
  }
}

{% endhighlight %}

	- assert（一般断言）

{% highlight c %}

assert(a == b || c >= d)
// Error message: 1 did not equal 2, and 3 was not greater than or equal to 4

assert(xs.exists(_ == 4))
// Error message: List(1, 2, 3) did not contain 4

assert("hello".startsWith("h") && "goodbye".endsWith("y"))
// Error message: "hello" started with "h", but "goodbye" did not end with "y"

assert(num.isInstanceOf[Int])
// Error message: 1.0 was not instance of scala.Int

assert(Some(2).isEmpty)
// Error message: Some(2) was not empty

{% endhighlight %}

	- Expected results（预期结果等于实际结果）

{% highlight c %}

val a = 5
val b = 2
assertResult(2) {
  a - b
}

{% endhighlight %}
	
	- exceptions(断言异常)

{% highlight c %}

val s = "hi"
try {
  s.charAt(-1)
  fail()
}
catch {
  case _: IndexOutOfBoundsException => // Expected, so continue
}

{% endhighlight %}

	- Share Fixture：before – after

{% highlight c %}

package org.scalatest.examples.flatspec.beforeandafter

import org.scalatest._
import collection.mutable.ListBuffer

class ExampleSpec extends FlatSpec with BeforeAndAfter {

  val builder = new StringBuilder
  val buffer = new ListBuffer[String]

  before {
    builder.append("ScalaTest is ")
  }

  after {
    builder.clear()
    buffer.clear()
  }

  "Testing" should "be easy" in {
    builder.append("easy!")
    assert(builder.toString === "ScalaTest is easy!")
    assert(buffer.isEmpty)
    buffer += "sweet"
  }

  it should "be fun" in {
    builder.append("fun!")
    assert(builder.toString === "ScalaTest is fun!")
    assert(buffer.isEmpty)
  }
}

{% endhighlight %}
 
**运行ScalaUnitTest**

ScalaTest支持多种方式运行测试，同样支持 Junit4 Runner方式运行，例子如下：
 
**Run, Debug, Coverage**

前提：安装IDEA scala 插件

- IDEA 运行测试
	- 测试类->右键点击->选择 Run ‘XXX’
- IDEA Degug 测试
	- 测试类->右键点击->选择 Debug ‘XXX’
- IDEA生成覆盖率
	- 测试类->右键点击->选择 Run ‘XXX’ with Coverage
