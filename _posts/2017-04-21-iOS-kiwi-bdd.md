---
title:  "iOS Kiwi-BDD安装配置"
date:   2017-04-21 15:08:12
categories: iOS
---
**Kiwi和BDD**

XCTest是Xcode自带的测试框架(Xcode5时加入的)，是基于OCUnit的传统测试框架，在书写性和可读性上都不太好。在测试用例太多的时候，由于各个测试方法是割裂的，想在某个很长的测试文件中找到特定的某个测试并搞明白这个测试是在做什么并不是很容易的事情。所有的测试都是由断言完成的，而很多时候断言的意义并不是特别的明确，对于项目交付或者新的开发人员加入时，往往要花上很大成本来进行理解或者转换。另外，每一个测试的描述都被写在断言之后，夹杂在代码之中，难以寻找。使用XCTest测试另外一个问题是难以进行mock或者stub，而这在测试中是非常重要的一部分。 

行为驱动开发（BDD）正是为了解决上述问题而生的，作为第二代敏捷方法，BDD提倡的是通过将测试语句转换为类似自然语言的描述，开发人员可以使用更符合大众语言的习惯来书写测试，这样不论在项目交接/交付，或者之后自己修改时，都可以顺利很多。如果说作为开发者的我们日常工作是写代码，那么BDD其实就是在讲故事。一个典型的BDD的测试用例包活完整的三段式上下文，测试大多可以翻译为Given..When..Then的格式，读起来轻松惬意。BDD在其他语言中也已经有一些框架，包括最早的Java的JBehave和赫赫有名的Ruby的RSpec和Cucumber。Kiwi是objc中BDD框架，同时Kiwi对Mock、Stub和异步的良好支持，使我们可以方便的写出漂亮优美的测试。

**CocoaPods**

如果已经安装CocoaPods 和 Command Line Tools，可以跳过此步骤。
- 安装CocoaPods
	- 打开Terminal

{% highlight c %}

$ [sudo] gem install cocoapods
$ pod setup

{% endhighlight %}

备注：CocoaPods 为Xcode Project 包依赖管理工具。具体资料请参考Cocoapods 官网：https://guides.cocoapods.org/

- 安装Command Line Tools
	- 打开Terminal

{% highlight c %}

$: xcode-select –install

{% endhighlight %}

下载完成后，界面step-by-step安装
备注：Command Line Tools 为Xcode的命令行工具，可以在Command Line Tools上，通过执行命令，可以完成编译、打包、测试、归档等。


**Unit Test Target**

如果Xcode Project已经包含了Unit Test Target，可以跳过以下1和2步骤。在Xcode Project上安装Kiwi前，Project中需要有Unit Test Target。如果使用Xcode5+，在创建Project时，Xcode会自动创建Unit Test Target。如果Project中，没有Unit Test Target，可以手动方式添加Unit Test Target，以下为检查和手动添加Unit Test Target步骤。

- 检查Unit Test Target
	- 在Project导航栏中，选择Project
	- 右侧的TARGETS中是否包含Unit Test Target
![kiwi1]({{ site.url }}/images/kiwibdd/kiwi1.png)

- 添加Unit Test Target
	- File->New->Target…->iOS或OS X->Other->Cocoa Touch Testing Bundle(iOS)/Cocoa Testing Bundle(OS X)->Next
	- 给Unit Test Target取个合适的名字，一般是XXXTests(XXX为projectName，例如AmazingAppTests)。
	- 选择合适的语言，并确认Unit Test Target关联的project正确。
	- 点击Finish完成

**Installing Pods**
- 首先退出Xcode
- 在包含.xcodeproj文件的目录中创建名称为Podfile的文件(如果已经存在，可以编辑文件)。文件内容如下：
![kiwi2]({{ site.url }}/images/kiwibdd/kiwi2.png)
- 在project中安装pods，CocoaPods会根据Podfile文件配置下载相应的Libs。

{% highlight c %}

$ pod install

{% endhighlight %}

- CocoaPods会在.xcodeproj的目录中生成后缀为.xcworkspace文件，以后用.xcworkspace文件打开project。

{% highlight c %}

$ open myproject.xcworkspace

{% endhighlight %}

**Unit Test Target Configuration**

- 展开project导航菜单，确认CocoaPods生成的Pods Group中，包含2个配置文件:Pods-<TargetName>debug.xcconfig  Pods-<TargetName>release.xcconfig
![kiwi3]({{ site.url }}/images/kiwibdd/kiwi3.png)
- 确认Pods-<TargetName>.xcconfig 已经关联了Configurations下的Unit Test Target
![kiwi4]({{ site.url }}/images/kiwibdd/kiwi4.png)
- 确认Unit Test Target的Build Settings中的"Other Linker Flags"配置包含-ObjC –framework “XCTest” –l “Pods-yourUnitTestTargetName-Kiwi”
![kiwi5]({{ site.url }}/images/kiwibdd/kiwi5.png)
- 指定运行测试时，通过执行application 加载unit test bundle(如果测试类库，可以跳过以下5、6两个步骤)
	- iOS: Ensure that "Bundle Loader" is set to $(BUILT_PRODUCTS_DIR)/MyProject.app/ApplicationTargetName
	![kiwi6]({{ site.url }}/images/kiwibdd/kiwi6.png)
	- OS X: Ensure that "Bundle Loader" is set to $(BUILT_PRODUCTS_DIR)/MyProject.app/Contents/MacOS/ApplicationTargetName.
- 指定Unit Test被注入到application，这与"Bundle Loader"设置相匹配。
	- 设置”Test Host”为：$(BUNDLE_LOADER)
	![kiwi7]({{ site.url }}/images/kiwibdd/kiwi7.png)

**Schemes**

- 打开Scheme 管理窗口
- Product -> Scheme -> Manage Schemes
- 选择与Project同名的Scheme
- 编辑Scheme
- Edit
- 选择”Test” action(位于左边)
- 检查右边的Tests列表中包含需要的Unuit Test Target，展开Unuit Test Target可以看到包含的”Test”列表。
![kiwi8]({{ site.url }}/images/kiwibdd/kiwi8.png)

到此安装配置已经完成，可以写个测试尝试使用下Kiwi。

**Write a Test**

- 新建测试类：File->New->File->iOS 或 OS X->Objective-c File->Next
- 输入文件名SampleSpec->Next
- Targets中确认勾选Unit Test Target->Create
- 进入SampleSpec代码编辑窗口，删除所有代码，用以下代码替换

{% highlight c %}

#import "kiwi.h"

SPEC_BEGIN(MathSpec)

describe(@"Math",^{
	it(@"is pretty cool",^{
		NSUInteger a = 16;
		NSUInteger b = 26;
		[[theValue(a+b) should] equal:theValue(43)];
	});
});

SPEC_END

{% endhighlight %}

- 执行测试：product-Test或者快捷键 command+u
![kiwi9]({{ site.url }}/images/kiwibdd/kiwi9.png)

