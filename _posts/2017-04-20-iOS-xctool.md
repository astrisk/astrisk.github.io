---
title:  "iOS-AutoTest-XCTool安装使用"
date:   2017-04-20 17:31:59
categories: iOS
---

**HomeBrew**

Homebrew作为OS X上强大的包管理器，为系统软件提供了非常方便的安装方式，独特式的解决了包的依赖问题，一键式编译，无参数困扰。Homebrew依赖于XCode，首先需要安装Xcode。同时Homebrew也依赖ruby，Mac已经自带ruby。

{% highlight c %}

#安装Homebrew:
$:ruby -e "$(curl –fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

#查看brew的帮助
$:brew –help

#安装软件(git)
$:brew install git

#卸载软件(git)
$:brew uninstall git

#搜索软件(git)
$:brew search git

#显示已经安装软件列表
$:brew list

#更新软件，把所有的Formula目录更新，并且会对本机已经安装并有更新的软件用*标明。
$:brew update

#更新某具体软件(git)
$:brew upgrade git

#查看软件信息
$:brew [info | home] [FORMULA...]
#删除程序，和upgrade一样，单个软件删除和所有程序老版删除。
$:brew cleanup git
$:brew cleanup
#查看那些已安装的程序需要更新
$:brew outdated

{% endhighlight %}

程序安装路径及文件夹
Homebrew将本地的/usr/local初始化为git的工作树，并将目录所有者变更为当前所操作的用户，以后的操作将不需要sudo。

-bin          用于存放所安装程序的启动链接（相当于快捷方式）
-Cellar       所以brew安装的程序，都将以[程序名/版本号]存放于本目录下
-etc          brew安装程序的配置文件默认存放路径
-Library      Homebrew 系统自身文件夹
+–Formula     程序的下载路径和编译参数及安装路径等配置文件存放地
+–Homebrew    brew程序自身命令集

**XCTool**

XCTool是facebook开源的一个命令行工具，用来替代苹果的xcodebuild。主要有以下功能：
- 像xcode一样跑测试用例。
- 结构化输出编译测试结果。
- 彩色且方便阅读的编译内容输出。

Xctool 安装，最简单的办法是在终端中通过homebrew安装xctool

{% highlight c %}

$:brew update
$:brew install xctool

#检查安装是否成功
$:xctool –version (终端显示xctool的版本)

{% endhighlight %}

**Xctool 使用**

在终端中进入项目目录，执行相应命令：

- 语法

{% highlight c %}

$: xctool [BASE OPTIONS] [ACTION [ACTION ARGUMENTS]] ...

ACTION:
    xctool [BASE OPTIONS] clean
    xctool [BASE OPTIONS] build
    xctool [BASE OPTIONS] build-tests [-only TARGET] [-skip-deps]
    xctool [BASE OPTIONS] run-tests [-test-sdk SDK] [-only SPEC] [-freshSimulator] [-freshInstall]
    xctool [BASE OPTIONS] test [-test-sdk SDK] [-only SPEC] [-skip-deps] [-freshSimulator] [-freshInstall]
    xctool [BASE OPTIONS] archive
Base Options:
    -help                    show help
    -workspace PATH          path to workspace
    -project PATH            path to project
    -scheme NAME             scheme to use for building or testing
    -find-target TARGET      Search for the workspace/project/scheme to build the target
    -find-target-path PATH   Path to search for -find-target.
    -find-target-exclude-pathColon-separated list of paths to exclude for -find-target.
    -sdk VERSION             sdk to use for building (e.g. 6.0, 6.1)
    -configuration NAME      configuration to use (e.g. Debug, Release)
    -jobs NUMBER             number of concurrent build operations to run
    -arch ARCH               arch to build for (e.g. i386, armv7)
    -toolchain PATH          path to toolchain
    -xcconfig PATH           path to an xcconfig
    -reporter TYPE[:FILE]    add reporter
    -showBuildSettings       display a list of build settings and values
    -version                 print version and exit
    SETTING=VALUE            Set the build 'setting' to 'value'
Options for 'build-tests' action:
    -only TARGET             build only a specific test TARGET
    -skip-deps               Only build the target, not its dependencies
Options for 'run-tests' action:
    -test-sdk SDK            SDK to test with
    -only SPEC               SPEC is TARGET[:Class/case[,Class2/case2]]
    -freshSimulator          Start fresh simulator for each application test target
    -freshInstall            Use clean install of TEST_HOST for every app test run
Options for 'test' action:
    -test-sdk SDK            SDK to test with
    -only SPEC               SPEC is TARGET[:Class/case[,Class2/case2]]
    -skip-deps               Only build the target, not its dependencies
    -freshSimulator          Start fresh simulator for each application test target
    -freshInstall            Use clean install of TEST_HOST for every app test run

{% endhighlight %}

- Build 

{% highlight c %}

$ xctool 
  -workspace YourWorkspace.xcworkspace 
  -scheme YourScheme 
   archive
build

{% endhighlight %}


- 测试

{% highlight c %}

$: xctool
  -workspace YourWorkspace.xcworkspace
  -scheme YourScheme 
  Test

{% endhighlight %}

- 生成archive文件

{% highlight c %}

$: xctool -workspace YourWorkspace.xcworkspace -scheme myScheme archive 

{% endhighlight %}


- 补充

{% highlight c %}

# 把生成文件存放到tmp目录
$ xctool -workspace YourWorkspace.xcworkspace -scheme myScheme 
 SYMROOT=./tmp build 

{% endhighlight %}









