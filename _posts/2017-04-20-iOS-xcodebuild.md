---
title:  "iOS-AutoTest-Xcodebuild"
date:   2017-04-20 17:16:34
categories: iOS
---

**简介**

Apple的命令行工具，可以通过命令行方式来编译、打包、执行测试。Xcodebuild 可以编译project中的一或多target。Xcodebuild 也支持编译指定workspace或project中scheme。

**安装**

- 在终端中下载Command LineTool

{% highlight c %}

$: xcode-select –install

{% endhighlight %}

- 下载完成后，界面step-by-step安装


**使用**

{% highlight c %}

#查看xcode 的版本号 和build 版本号
$: xcodebuild –version 

#显示当前系统的SDK及其版本	
$:xcodebuild –showsdks 

#查看xcodebuild man手册(帮助文档)	
$:man xcodebuild 

#退出man手册	
$:  : + q   

#清理目录	
$:xcodebuild clean install 

# build MyTarget

xcodebuild -target MyTarget OBJROOT=/Build/MyProj/Obj.root
              SYMROOT=/Build/MyProj/Sym.root

              Builds the target MyTarget in the Xcode project in the directory
              from which xcodebuild was started, putting intermediate files in
              the directory /Build/MyProj/Obj.root and the products of the
              build in the directory /Build/MyProj/Sym.root.

xcodebuild -sdk macosx10.6

              Builds the Xcode project in the directory from which xcodebuild
              was started against the Mac OS X 10.6 SDK.  The canonical names
              of all available SDKs can be viewed using the -showsdks option.

xcodebuild -workspace MyWorkspace.xcworkspace -scheme MyScheme

              Builds the scheme MyScheme in the Xcode workspace
              MyWorkspace.xcworkspace.

xcodebuild -workspace MyWorkspace.xcworkspace -scheme MyScheme archive

              Archives the scheme MyScheme in the Xcode workspace
              MyWorkspace.xcworkspace.

xcodebuild -workspace MyWorkspace.xcworkspace -scheme MyScheme
              -destination 'platform=OS X,arch=x86_64' test

              Tests the scheme MyScheme in the Xcode workspace
              MyWorkspace.xcworkspace using the destination described as My
              Mac 64-bit in Xcode.

xcodebuild -workspace MyWorkspace.xcworkspace -scheme MyScheme
              -destination 'platform=iOS Simulator,name=iPhone 5s'
              -destination 'platform=iOS,name=My iPad' test

              Tests the scheme MyScheme in the Xcode workspace
              MyWorkspace.xcworkspace using both the iOS Simulator device
              named iPhone 5s for the latest version of iOS, and the the iOS
              device named My iPad.  (Note that the shell requires arguments
              to be quoted or otherwise escaped if they contain spaces.)

xcodebuild -workspace MyWorkspace.xcworkspace -scheme MyScheme
              -destination generic/platform=iOS build

              Builds the scheme MyScheme in the Xcode workspace
              MyWorkspace.xcworkspace using the generic iOS Device destina-
              tion.

xcodebuild -exportArchive -exportFormat IPA -archivePath
              MyMobileApp.xcarchive -exportPath MyMobileApp.ipa
              -exportProvisioningProfile 'MyMobileApp Distribution Profile'

              Exports the archive MyMobileApp.xcarchive as an IPA file to the
              path MyMobileApp.ipa using the provisioning profile MyMobileApp
              Distribution Profile.

xcodebuild -exportArchive -exportFormat APP -archivePath
              MyMacApp.xcarchive -exportPath MyMacApp.pkg
              -exportSigningIdentity 'Developer ID Application: My Team'

              Exports the archive MyMacApp.xcarchive as a PKG file to the path
              MyMacApp.pkg using the application signing identity Developer ID
              Application: My Team.  The installer signing identity Developer
              ID Installer: My Team is implicitly used to sign the exported
              package.


{% endhighlight %}




