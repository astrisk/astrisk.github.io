---
title:  "接口集成测试框架"
date:   2017-04-21 17:06:51
categories: INTEGRATIONTEST
---

**背景**

公司大数据平台通用接口需要一个测试工具，既能方便地调用代码，实现类似UnitTest测试，又能调用RESTAPI。没有找到合适地工具，就自己动手基于Nunit写了这个框架。
框架采用数据驱动方式，把测试代码和测试数据分离，根据测试用例文件会动态生成多个case，并执行。

**实现思路**

- 分离测试代码和测试数据，把测试数据填写到外部的json文件中。
- 读取json文件的测试数据，序列化json字符串为TestCase集合。
- 遍历TestCase，生成TestCaseData集合，做为测试数据源返回给测试方法。
- 遍历测试数据源，执行测试方法。
![integrationTest1]({{ site.url }}/images/integrationTest/integrationtest1.png)

**测试数据**

- 数据格式
	- json数组，数组中一个元素对应一个测试用例。如需增加用例，只需在数组中增加一个json用例数据。
- 用例数据组成
	每个用例由TestName、Input、ExpValue三个元素组成
	- TestName: 测试用例名称
	- Input: 输入参数，供调用生产代码，需要传参使用，会被序列化成Jobject对象。调用时，按input[key]的方式取值，如果调用代码，不需要传参，可以填写为 "Input":{} 。
	- ExpValue: 预期值，用来跟实际值匹配，检查测试是否通过，格式为实际值业务model对应的json字符串。

**框架引用**

框架用到以下dll,请确保加载以下dll:
- nunit.framework：Nunit测试框架
- Newtonsoft.Json：Json序列化和反序列化组件
- TestHelper：测试帮助组件。序列化json字符串，比较函数等。

**测试代码组成**

测试代码由两部分组成：用例工厂和测试代码

- 用例工厂：读取Json测试数据，生成测试用例
	- 在测试项目中编写测试用例工厂类：TestCaseFactory
	- 工厂类中定义测试数据源属性，具体代码如下。使用中只需要更改数据源属性名称、用例数据文件路径、更改相应model类型（这个model用来序列化json数据中的expValue的内容）

- 测试代码：编写测试类，测试方法

**编写测试**

- 在测试项目中添加测试类

- 编写测试方法

方法需加属性[Test, TestCaseSource(typeof(TestCaseFactory), "XXXTest")]。
	- TestCaseFactory:测试数据源工厂类
	- XXXTest：TestCaseFactory(可以自定义类名)类中的XXXTest属性

- 配置数据库(可选)

如果测试项目需要连接数据库，则需要为测试项目增加app.config文件，并在app.config文件中定义数据库连接字符串

**执行测试**

安装完dotCover,在测试类和每个测试方法左边会有个圆圈图标，点击圆圈，执行测试、debug或进行覆盖率分析

**完整例子**

数据文件

{% highlight c %}

[
  {
    "TestName":"InstallTest260",
    "Input":{"url":"http://localhost/api/Datas/installs?pids","pids":"260"},
    "ExpValue":{
      "Count": 14481
    }
  },
  {
    "TestName":"InstallTest313",
    "Input":{"url":"http://localhost/api/Datas/installs?pids","pids":"313"},
    "ExpValue":{
      "Count": 9744
    }
  }
]

{% endhighlight %}

测试代码

{% highlight c %}


namespace DatasControllerTest
{

    [TestFixture]
    public class DatasController_installTest
    {
         [SetUp]
          public void Indata()
          {
              DateTime dt = DateTime.ParseExact("20150407", "yyyyMMdd",
                  System.Globalization.CultureInfo.CurrentCulture);
              DatasController controller = new DatasController();
              controller.PostUpdateTimes(dt);
          }

        [Test, TestCaseSource(typeof (TestCaseFactoryMethod), "installTest")]
        public void WebApiInstallCount(JObject input, List<InstallModel> expValue)
        {
            string message;
            //
            var pids = input["pids"].ToString();
            var ed = input["ed"].ToString();
            var sd = input["sd"].ToString();
            var zt = input["zt"].ToString();
            DatasController controller = new DatasController();
            List<InstallModel> actValue = controller.GetInstalls(pids, Convert.ToDateTime(sd), Convert.ToDateTime(ed),
                int.Parse(zt));
            //验证
            bool result = NunitTestHelper.CompareObject(actValue, expValue, out message);
            Assert.IsTrue(result, message);
        }
    }

    /// <summary>
    /// 测试获取指定Profile页面浏览次数前几的页面信息 - 单APP最热内容排行
    /// </summary>
    [TestFixture]
    public class DatasController_GetPagesTest
    {
        [Test, TestCaseSource(typeof (TestCaseFactoryMethod), "GetPagesTest")]
        public void WebApiGetPagesTest(JObject input, List<PageInfoModel> expValue)
        {
            string message;
            //
            var pids = input["pids"].ToString();
            var pn = input["pn"].ToString();
            DatasController controller = new DatasController();
            List<PageInfoModel> actValue = controller.GetPages(pids, int.Parse(pn));
            //验证
            //验证
            bool result = NunitTestHelper.CompareObject(actValue, expValue, out message);
            Assert.IsTrue(result, message);
        }


    }

    ///   测试用例工厂,生成测试用例
    /// 
    #region
    public class TestCaseFactoryMethod
    {
        public static IEnumerable installTest
        {

            get
            {
                var fileinfos = new[]
                {
                    new FileInfo(@"..\..\TestData\DatasController1.json")
                };
                return NunitTestHelper.TestCaseGenerater<List<InstallModel>>(fileinfos).Cast<object>();
            }
        }

        public static IEnumerable GetPagesTest
        {
            get
            {
                var fileinfos = new[]
               {
                    new FileInfo(@"..\..\TestData\GetPagesTest.json")
                };
                return NunitTestHelper.TestCaseGenerater<List<PageInfoModel>>(fileinfos).Cast<object>();
            }
        }
        #endregion
    }
}

{% endhighlight %}

**不足**

测试代码编写有些复杂，后期看看能否把两部分（生成测试数据和测试代码）整合成一块，在一个公共地地方调用一次，减少复杂度。