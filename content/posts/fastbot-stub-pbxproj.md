---
title: 使用pbxproj添加fastbot-stub.framework并运行测试
date: 2021-11-25
tags:
- github
- pbxproj
- ios
- 开源项目
- fastbot
- fastbot-stub
- ios-monkey
- xcode
categories:
- iOS
---


## 背景

Monkey测试一直是一种强度较高，性价比较高的测试手段，但在iOS平台上，工具用起来总有各种各样的苦难。今年字节跳动开源了一款自动化测试工具[bytedance/Fastbot_iOS](https://github.com/bytedance/Fastbot_iOS)，效果非常好。但默认实现是基于纯图像识别的，有时候难免遇到一些靠图像不好处理的地方，陷入覆盖率较低的境地。但好在项目也提供了`stub模式`，只是上手有一定的技术门槛。之前的文章[使用yololib注入fastbot-stub并重签名运行测试](https://yanbo92.site/fastbot-stub-inject/)介绍了一种逆向手法使用fastbot-stub。本文将介绍一种从Xcode编译角度加入fastbot-stub的方法，当然，这需要app的完整代码。





## 相关项目介绍

#### [Fastbot_iOS](https://github.com/bytedance/Fastbot_iOS)

> Jenkins是一个开源的、提供友好操作界面的持续集成(CI)工具，起源于Hudson（Hudson是商用的），主要用于持续、自动的构建/测试软件项目、监控外部任务的运行（这个比较抽象，暂且写上，不做解释）。Jenkins用Java语言编写，可在Tomcat等流行的servlet容器中运行，也可独立运行。通常与版本管理工具(SCM)、构建工具结合使用。常用的版本控制工具有SVN、GIT，构建工具有Maven、Ant、Gradle。



#### [mod-pbxproj](https://github.com/kronenthaler/mod-pbxproj)

>这是一个可以通过命令行修改Xcode项目依赖的python模块，便于在不使用界面的情况下增加或者删除库。



#### [Swift-30-Projects](https://github.com/soapyigu/Swift-30-Projects)

>这个项目是一个由30个iOS小项目组成的合集，本文使用的调试app就是项目中的04 ToDo，clone下来打包一个ipa即可。当然也可以换成其他脱了壳或者没上架Appstore的ipa包，我用这个只是因为体积比较小，打包和重签名都比较快。



## 实现原理

通过pbxproj工具向Xcode项目中添加fasbot-stub.framework动态库，并重新编译，达到让APP支持stub模式的效果。



## 具体步骤

#### 环境准备

此处需要准备好的东西：

- 一台Mac
- 测试APP的代码
- 开发者证书，以及可用的描述文件
- Python环境
- fastbot-stub.framework



##### Python环境安装pbxproj

```
sudo pip install pbxproj
```



##### fastbot-stub.framework

按照`Fastbot-iOS`项目的Readme打开`Fastbot-iOS.xcworkspace`，编译即可，完整命令：

```
git clone git@github.com:bytedance/Fastbot_iOS.git

cd Fastbot-iOS && pod install --repo-update

open Fastbot-iOS.xcworkspace
```

然后target选择`fastbot-stub`，连一台真机编译，得到产物`fastbot-stub.framework`





#### 添加动态库

同样的，我们用上一篇文章的todo项目，编写python文件如下：

```python
from pbxproj import XcodeProject
from pbxproj.pbxextensions.ProjectFiles import FileOptions
import time

project = XcodeProject.load('Swift-30-Projects/Project 04 - TodoTDD/ToDo.xcodeproj/project.pbxproj')
file_options = FileOptions(weak=True)
project.add_file('fastbot_stub.framework', force=False, file_options=file_options)

project.save()
```

修改代码中的路径，直接运行即可

若不想通过python脚本，也可以通过纯命令行的方式，具体可以参考[CLI · kronenthaler/mod-pbxproj Wiki (github.com)](https://github.com/kronenthaler/mod-pbxproj/wiki/CLI)



#### 构建、打包

若没有导出ipa需求，直接在Xcode中指定手机构建即可。需要导出ipa比较麻烦，建议自行搜索。





回到Fastbot-iOS工程，修改Fastbot-Runner的Scheme：

```
dataport为9797
launchenv为stubPort=9797
```

再修改`FastbotRunner/FastbotRunner.m`，将以下代码取消注释

```swift
    [fastbot_native addUIInterruptionMonitor:^CGRect(NSArray<XCUIElement *> *systemAlerts) {
        NSArray<XCUIElement*> *buttons = [systemAlerts.firstObject.buttons allElementsBoundByIndex];
        NSInteger buttonCount = [buttons count];
        CGRect btnRect = CGRectZero;
        if(buttonCount<=0)
            return btnRect;
        if(buttonCount > 2)
        {
            btnRect = [[buttons objectAtIndex:2] frame];
        }
        else
            btnRect = [buttons.lastObject frame];
        return btnRect;
    }];
```

代码大意为处理系统弹窗

像之前运行Fastbot-iOS一样在命令行传参运行

```
BUNDLEID=yigu.com.ToDo duration=240 throttle=300 xcodebuild test  -workspace Fastbot-iOS.xcworkspace -scheme FastbotRunner  -configuration Release  -destination 'platform=iOS,id=c6b0ab4fa8867c51cf1c5b6d8cd076d3957192b2' -only-testing:FastbotRunner/FastbotRunner/testFastbot
```

截取一段命令行输出：

```
[fastbot] : visit ToDo.InputViewController,UIApplicationRotationFollowingController,UIApplicationRotationFollowingControllerNoTouches,UICompatibilityInputViewController,UIInputViewController,UIInputWindowController; visited ViewController count is: 2 
[fastbot] : state visited: 30 
[fastbot] : action first visited, get reward 2.336364
[fastbot] : state is saturated, get reward 0.295547
[fastbot] : got reward: 9.3110
```

区别在哪？区别就在于有控件信息了，如果看到类似`UIInputWindowController`这样的字样，恭喜你，你成功了。







## 参考文章

[字节跳动质量利器 -- 移动端智能化稳定性测试工具 Fastbot-Android/iOS 双端重磅发布上线](https://testerhome.com/topics/31113)

[奔跑吧！智能Monkey之Fastbot跨平台 ](https://mp.weixin.qq.com/s/QhzqBFZygkIS6C69__smyQ)

[Issue #44 · bytedance/Fastbot_iOS](https://github.com/bytedance/Fastbot_iOS/issues/44)



