---
title:  "UNITTEST-VS+NUNIT+DOTCOVER搭建UNITTEST环境"
date:   2017-04-20 17:47:21
categories: UNITTEST
---

**工具安装**

Visual Studio中执行Nuint用例，需要安装NUnit Test Adapter或dotCover(推荐使用)

- vs安装NUnit Test Adapter：
	- 工具—扩展和更新—搜索” NUnit Test Adapter”—下载—安装
	![unittest1]({{ site.url }}/images/unittests/unit tests1.png)
	- Visual Studio 安装dotCover（推荐使用，后面图用dotCover执行测试）： 
备注：dotCover为商业插件，需要购买License或者…。

**新建单元测试项目**

	- 解决方案-右键-添加-新建项目-单元测试项目
	![unittest2]({{ site.url }}/images/unittests/unit tests2.png)
	![unittest3]({{ site.url }}/images/unittests/unit tests3.png)
备注：单元测试和被测试代码放在同一个解决方案下，便于测试代码的管理。
 
 
**编写单元测试类和方法**

	- 引用中，添加Nunit.Framework 和被测试dll的引用，删除MSTest framework dll引用：Microsoft.VisualStudio.QualityTools.UnitTestFramework.dll
	![unittest4]({{ site.url }}/images/unittests/unit tests4.png)
	
 	- 编写测试类，用TestFixture属性标识测试类。
 	![unittest5]({{ site.url }}/images/unittests/unit tests5.png)

	- 编写测试方法，用Test属性标识。
 	![unittest6]({{ site.url }}/images/unittests/unit tests6.png)

**执行、Debug、生成覆盖率**

	- 代码编辑界面执行
	![unittest7]({{ site.url }}/images/unittests/unit tests7.png)

	- 工具栏菜单执行
	![unittest8]({{ site.url }}/images/unittests/unit tests8.png)

	- 解决方案资源管理器中执行
 	![unittest9]({{ site.url }}/images/unittests/unit tests9.png)

**测试结果和代码覆盖率**

	- 在Session窗口中查看执行结果(执行测试时，会自动打开Session窗口)。
 	![unittest10]({{ site.url }}/images/unittests/unit tests10.png)

	- 在Session窗口查看覆盖率
 	![unittest11]({{ site.url }}/images/unittests/unit tests11.png)

	- 双击上图右侧中Coverage Tree里的方法名，跳转到代码编辑窗口，查看语句覆盖：绿色的为已覆盖；红色的为未覆盖。
 	![unittest12]({{ site.url }}/images/unittests/unit tests12.png)

	- 在测试类代码编辑窗口查看结果：绿色勾为通过；红色圆(中间白色线条)为失败。
	![unittest13]({{ site.url }}/images/unittests/unit tests13.png)