[Xcode7 添加PCH文件](http://www.jianshu.com/p/e6e0e3bbbf38)

1.)  打开你的Xcode工程. 在Supporting Files目录下,选择 File > New > File > iOS > Other > PCH File 然后点击下一步；

2.) 假设你的项目名称为TestDemo, 你的PCH 文件的名字应该为 TestDemo-Prefix.pch,然后创建；

3.) 选择 PCH 文件(文章的示例文件为 TestDemo-Prefix.pch) ,可以看到里面的内容如下:

4.) 找到 Project > Build Settings > 搜索 “Prefix Header“；

5.) “Apple LLVM 7.0 -Language″ 栏目中你将会看到 Prefix Header 关键字；

6.) 输入: TestDemo/TestDemo-Prefix.pch (如 TestDemo/TestDemo-Prefix.pch )；

7.)，将Precompile Prefix Header为YES，预编译后的pch文件会被缓存起来，可以提高编译速度。效果如下

8.) Clean 并且 build 你的项目.

就是这样！Done！现在你可以使用你的 PCH 文件就像你使用老版本的Xcode一样了

文／默默desire（简书作者）
原文链接：http://www.jianshu.com/p/e6e0e3bbbf38
著作权归作者所有，转载请联系作者获得授权，并标注“简书作者”。


[配置CocoaLumberjack的一些问题记录](https://my.oschina.net/ioslighter/blog/381201)
颜色日志，由于需要 XCode 插件，而 XCode 8 已禁用插件，所以没有继续验证，其它配置参照调整，可用。



想在Xcode中整一个彩色日志显示，按照GettingStarted.md 一文中的步骤将CocoaLumberjack 2.x整合进我的项目中来，遇到一些问题，当然不乏一些坑，作个记录。



整合步骤：

Drag CocoaLumberjack/Framework/{Desktop/Mobile}/Lumberjack.xcodeproj into your project

In your App target Build Settings

Add to 'User Header Search Paths' $(BUILD_ROOT)/../IntermediateBuildFilesPath/UninstalledProducts/include

Set 'Always Search User Paths' to YES

In your App target Build Phases

Add CocoaLumberjack static library target to 'Target Dependencies'

Add libCocoaLumberjack.a to 'Link Binary With Libraries'

Include the framework in your source files with

#import <CocoaLumberjack/CocoaLumberjack.h>


1、首先是编译提示Use of undeclared identifier 'LOG_LEVEL_VERBOSE'问题

这个我是按照文档XcodeTricks.md 在pch文件中加了下面的代码：

#ifdef DEBUG
  static const int ddLogLevel = LOG_LEVEL_VERBOSE;
#else
  static const int ddLogLevel = LOG_LEVEL_WARN;
#endif
踩坑1，把LOG_LEVEL_VERBOSE和LOG_LEVEL_WARN换成DDLogLevelVerbose和DDLogLevelError就好了。

修改后的代码应该是：

#ifdef DEBUG
static const int ddLogLevel = DDLogLevelVerbose;
#else
static const int ddLogLevel = DDLogLevelError;
#endif
然后在方法application:didFinishLaunchingWithOptions:中添加以下代码设置颜色显示：

[DDLog addLogger:[DDASLLogger sharedInstance]];
[DDLog addLogger:[DDTTYLogger sharedInstance]];
[[DDTTYLogger sharedInstance] setColorsEnabled:YES];
测试颜色显示的代码：

DDLogError(@"Paper jam");
DDLogWarn(@"Toner is low");
DDLogInfo(@"Warming up printer (pre-customization)");
DDLogVerbose(@"Intializing protcol x26 (pre-customization)");


2、彩色日志显示需要插件XcodeColors插件支持，这个插件我下载的是https://github.com/rvi/XcodeColors里面的，因为它支持了Xcode6.3。



3、用CocoaLumberjack Demo里自带的TextXcodeColors工程测试，pod install后打开TextXcodeColors.xcodeproj，编译提示ld: library not found for -lPods-TXC_ios-CocoaLumberjack

又是一个坑。一般来说，pod install后应该生成xcworkspace文件，但是没有生成TextXcodeColors.xcworkspace文件，我就奇怪了。后来才发现，应该是打开Demos.xcworkspace，然后在里面选择TextXcodeColors这个target，当然前提是先要进入TextXcodeColors文件夹执行pod install才行。尽量使用“pod install --verbose --no-repo-update”



4、Demo里测试日志颜色正常，在自己的项目里就不会显示颜色

坑3。奥秘在于要对Project的Scheme作了如下调整：

    In Xcode bring up the Scheme Editor (Product -> Edit Scheme...)
    Select "Run" (on the left), and then the "Arguments" tab
    Add a new Environment Variable named "XcodeColors", with a value of "YES"



5、内存暴涨问题

加入CocoaLumberjack后，模拟器测试正常，真机测试内存暴涨，导致xcode自动终结联机调试，不调用DDLog相关的代码就没有问题。最终缩小范围，发现把上面的第4步中的Environment Variable给删掉就没有问题，我真是晕了。

而另一个位置的同名项目，我测试是没有这个问题，真是奇哉怪也。

实在是没有办法，把DerivedData里的所有app文件夹全部删掉，再重新运行，就没有问题了。猜测是因为同名项目的缘故？



6、使用静态库

本来解决了上面的问题后使用CocoaLumberjack基本就没什么问题了，可是不幸又被我发现了一个问题，那就是当我要修改app图标和启动图片的时候，即在General--App Icons and Launch Images中点向右箭头时，进入的却是项目Lumberjack.xcodeproj的启动图片设置界面。这说明嵌入Lumberjack.xcodeproj后App Icons and Launch Images混乱了。

于是我想到了使用静态库，在网上找到一个现成的CocoaLumberjack静态库工程，链接是https://github.com/NachoMan/CocoaLumberjack-Static。下载后先执行﻿﻿git submodule update --init --recursive命令更新CocoaLumberjack库，再执行build.sh脚本即可生成一个zip包，里面包含了.a文件和头文件，复制到自己的工程中添加就行了。

可以用lipo -info xxxxx.a命令来检查生成的.a文件的CPU架构。

参考：
iOS开源项目之日志框架CocoaLumberjack 
利用 CocoaLumberjack 搭建自己的 Log 系统

Colorful XCode Console 






<p align="center" >
  <img src="LumberjackLogo.png" title="Lumberjack logo" float=left>
</p>

CocoaLumberjack
===============
[![Build Status](https://travis-ci.org/CocoaLumberjack/CocoaLumberjack.svg?branch=master)](https://travis-ci.org/CocoaLumberjack/CocoaLumberjack)
[![Pod Version](http://img.shields.io/cocoapods/v/CocoaLumberjack.svg?style=flat)](http://cocoadocs.org/docsets/CocoaLumberjack/)
[![Carthage compatible](https://img.shields.io/badge/Carthage-compatible-4BC51D.svg?style=flat)](https://github.com/Carthage/Carthage)
[![Pod Platform](http://img.shields.io/cocoapods/p/CocoaLumberjack.svg?style=flat)](http://cocoadocs.org/docsets/CocoaLumberjack/)
[![Pod License](http://img.shields.io/cocoapods/l/CocoaLumberjack.svg?style=flat)](http://opensource.org/licenses/BSD-3-Clause)
[![Reference Status](https://www.versioneye.com/objective-c/cocoalumberjack/reference_badge.svg?style=flat)](https://www.versioneye.com/objective-c/cocoalumberjack/references)
[![codecov](https://codecov.io/gh/CocoaLumberjack/CocoaLumberjack/branch/master/graph/badge.svg)](https://codecov.io/gh/CocoaLumberjack/CocoaLumberjack)

**CocoaLumberjack** is a fast & simple, yet powerful & flexible logging framework for Mac and iOS.

### How to get started
- install via [CocoaPods](http://cocoapods.org)

##### Swift version via CocoaPods
```ruby
platform :ios, '8.0'

# You need to set target when you use CocoaPods 1.0.0 or later.
target 'SampleTarget' do 
  use_frameworks!
  pod 'CocoaLumberjack/Swift'
end
```
Note: `Swift` is a subspec which will include all the Obj-C code plus the Swift one, so this is sufficient. 
For more details about how to use Swift with Lumberjack, see [this conversation](https://github.com/CocoaLumberjack/CocoaLumberjack/issues/405).

##### Swift Usage

If you installed using CocoaPods or manually:
```swift
import CocoaLumberjackSwift
```

```swift
DDLog.add(DDTTYLogger.sharedInstance()) // TTY = Xcode console
DDLog.add(DDASLLogger.sharedInstance()) // ASL = Apple System Logs

let fileLogger: DDFileLogger = DDFileLogger() // File Logger
fileLogger.rollingFrequency = TimeInterval(60*60*24)  // 24 hours
fileLogger.logFileManager.maximumNumberOfLogFiles = 7
DDLog.add(fileLogger)

...

DDLogVerbose("Verbose");
DDLogDebug("Debug");
DDLogInfo("Info");
DDLogWarn("Warn");
DDLogError("Error");
```

##### Obj-C version via CocoaPods

```ruby
platform :ios, '7.0'
pod 'CocoaLumberjack'
```

##### Obj-C usage
If you're using Lumberjack as a framework, you can `@import CocoaLumberjack`.

Otherwise, `#import <CocoaLumberjack/CocoaLumberjack.h>`

```objc
[DDLog addLogger:[DDTTYLogger sharedInstance]]; // TTY = Xcode console
[DDLog addLogger:[DDASLLogger sharedInstance]]; // ASL = Apple System Logs

DDFileLogger *fileLogger = [[DDFileLogger alloc] init]; // File Logger
fileLogger.rollingFrequency = 60 * 60 * 24; // 24 hour rolling
fileLogger.logFileManager.maximumNumberOfLogFiles = 7;
[DDLog addLogger:fileLogger];

...

DDLogVerbose(@"Verbose");
DDLogDebug(@"Debug");
DDLogInfo(@"Info");
DDLogWarn(@"Warn");
DDLogError(@"Error");
```

##### Installation with Carthage (iOS 8+)

[Carthage](https://github.com/Carthage/Carthage) is a lightweight dependency manager for Swift and Objective-C. It leverages CocoaTouch modules and is less invasive than CocoaPods.

To install with Carthage, follow the instruction on [Carthage](https://github.com/Carthage/Carthage)

Cartfile
```
github "CocoaLumberjack/CocoaLumberjack"
```

- or [install manually](Documentation/GettingStarted.md#manual-installation)
- read the [Getting started](Documentation/GettingStarted.md) guide, check out the [FAQ](Documentation/FAQ.md) section or the other [docs](Documentation/)
- if you find issues or want to suggest improvements, create an issue or a pull request
- for all kinds of questions involving CocoaLumberjack, use the [Google group](http://groups.google.com/group/cocoalumberjack) or StackOverflow (use [#lumberjack](http://stackoverflow.com/questions/tagged/lumberjack)).

### CocoaLumberjack 3

#### Migrating to 3.x

* To be determined

### Features

#### Lumberjack is Fast & Simple, yet Powerful & Flexible.

It is similar in concept to other popular logging frameworks such as log4j, yet is designed specifically for Objective-C, and takes advantage of features such as multi-threading, grand central dispatch (if available), lockless atomic operations, and the dynamic nature of the Objective-C runtime.

#### Lumberjack is Fast

In most cases it is an order of magnitude faster than NSLog.

#### Lumberjack is Simple

It takes as little as a single line of code to configure lumberjack when your application launches. Then simply replace your NSLog statements with DDLog statements and that's about it. (And the DDLog macros have the exact same format and syntax as NSLog, so it's super easy.)

#### Lumberjack is Powerful:

One log statement can be sent to multiple loggers, meaning you can log to a file and the console simultaneously. Want more? Create your own loggers (it's easy) and send your log statements over the network. Or to a database or distributed file system. The sky is the limit.

#### Lumberjack is Flexible:

Configure your logging however you want. Change log levels per file (perfect for debugging). Change log levels per logger (verbose console, but concise log file). Change log levels per xcode configuration (verbose debug, but concise release). Have your log statements compiled out of the release build. Customize the number of log levels for your application. Add your own fine-grained logging. Dynamically change log levels during runtime. Choose how & when you want your log files to be rolled. Upload your log files to a central server. Compress archived log files to save disk space...

### This framework is for you if:

-   You're looking for a way to track down that impossible-to-reproduce bug that keeps popping up in the field.
-   You're frustrated with the super short console log on the iPhone.
-   You're looking to take your application to the next level in terms of support and stability.
-   You're looking for an enterprise level logging solution for your application (Mac or iPhone).

### Documentation

- **[Get started using Lumberjack](Documentation/GettingStarted.md)**<br/>
- [Different log levels for Debug and Release builds](Documentation/XcodeTricks.md)<br/>
- [Different log levels for each logger](Documentation/PerLoggerLogLevels.md)<br/>
- [Use colors in the Xcode debugging console](Documentation/XcodeColors.md)<br/>
- [Write your own custom formatters](Documentation/CustomFormatters.md)<br/>
- [FAQ](Documentation/FAQ.md)<br/>
- [Analysis of performance with benchmarks](Documentation/Performance.md)<br/>
- [Common issues you may encounter and their solutions](Documentation/ProblemSolution.md)<br/>
- [AppCode support](Documentation/AppCode-support.md)
- **[Full Lumberjack documentation](Documentation/)**<br/>

### Requirements 
The current version of Lumberjack requires:
- Xcode 8 or later
- Swift 3.0 or later
- iOS 5 or later
- OS X 10.7 or later
- WatchOS 2 or later
- TVOS 9 or later

#### Backwards compability
- for Xcode 7.3 and Swift 2.3, use the 2.4.0 version
- for Xcode 7.3 and Swift 2.2, use the 2.3.0 version
- for Xcode 7.2 and 7.1, use the 2.2.0 version
- for Xcode 7.0 or earlier, use the 2.1.0 version
- for Xcode 6 or earlier, use the 2.0.x version
- for OS X < 10.7 support, use the 1.6.0 version

### Communication

- If you **need help**, use [Stack Overflow](http://stackoverflow.com/questions/tagged/lumberjack). (Tag 'lumberjack')
- If you'd like to **ask a general question**, use [Stack Overflow](http://stackoverflow.com/questions/tagged/lumberjack).
- If you **found a bug**, open an issue.
- If you **have a feature request**, open an issue.
- If you **want to contribute**, submit a pull request.

### Author
- [Robbie Hanson](https://github.com/robbiehanson)
- Love the project? Wanna buy me a coffee? (or a beer :D) [![donation](http://www.paypal.com/en_US/i/btn/btn_donate_SM.gif)](https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=UZRA26JPJB3DA)

### Collaborators
- [Ernesto Rivera](https://github.com/rivera-ernesto)
- [Dmitry Vorobyov](https://github.com/dvor)
- [Bogdan Poplauschi](https://github.com/bpoplauschi)
- [C.W. Betts](https://github.com/MaddTheSane)

### License
- CocoaLumberjack is available under the BSD license. See the [LICENSE file](https://github.com/CocoaLumberjack/CocoaLumberjack/blob/master/LICENSE.txt).

### Architecture

<p align="center" >
    <img src="Documentation/CocoaLumberjackClassDiagram.png" title="CocoaLumberjack class diagram">
</p>
