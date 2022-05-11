---
title: 使用yololib注入fastbot-stub并重签名运行测试
date: 2021-11-11
tags:
- github
- yololib
- iOS
- 开源项目
- resign
- 重签名
- 逆向
- iOS逆向
- 砸壳
- fastbot
- fastbot-stub
- ios-monkey
categories:
- Reverse
image: "cover.png"
url: "fastbot-stub-inject"
---


## 背景

Monkey测试一直是一种强度较高，性价比较高的测试手段，但在iOS平台上，工具用起来总有各种各样的苦难。今年字节跳动开源了一款自动化测试工具[bytedance/Fastbot_iOS](https://github.com/bytedance/Fastbot_iOS)，效果非常好。但默认实现是基于纯图像识别的，有时候难免遇到一些靠图像不好处理的地方，陷入覆盖率较低的境地。但好在项目也提供了`stub模式`，只是上手有一定的技术门槛。刷了一下Github的issue，在开发者[geron-cn](https://github.com/geron-cn)的启发下了解到了[yololib](https://github.com/KJCracks/yololib)，本文将提供一种通过`yololib`注入方式使用`fastbot-stub`的方案。
<!-- more -->




## 相关项目介绍

### Fastbot_iOS
[项目链接](https://github.com/bytedance/Fastbot_iOS)

> 2019 年字节跳动 Quality Lab 在自动测试生成方面进行了比较深入的探索，并研发了针对 Android、iOS 的稳定性测试工具 Fastbot。Fastbot 的核心技术主要包括：
>
>智能遍历：使用基于模型的测试生成（MBT），并提供多种算法策略，以获得较高的 Activity 覆盖率及问题发现能力；
>多机协同：最高支持数百台长时间多机协同遍历，同一个目标彼此协作；
>个性化的专家系统：业务方可以进行多种个性化配置，比如：限定测试在指定的 Activity 运行，屏蔽测试某些场景；
>模型复用：基于强化学习利用历史测试经验数据学习改进当次测试策略；
>复杂用例生成：对人工用例进行模仿学习，遍历过程中混合复杂用例的组合生成；
>精准定向：根据代码调用链变更自动生成针对变更场景的定向测试。


#### yololib
[项目链接](https://github.com/KJCracks/yololib)
>yololib是Kim Jong Cracks（Clutch 砸壳的作者）小组搞出来的一个dylib注入工具，利用这个工具，大大方便我们修改Mach-O 文件的 Load Command。以达到注入动态库的目的。

用法如下

```shell
yololib [binary] [dylib file]
```



#### LTResign
[项目链接](https://github.com/gltwy/LTResign)

>LTResign是一个用Python编写的重签名工具，这是iOS逆向绕不开的东西，但同类脚本有很多，作者通常用这个以及[GQResign](https://github.com/g763007297/GQResign)。



#### Swift-30-Projects
[项目链接](https://github.com/soapyigu/Swift-30-Projects)
>这个项目是一个由30个iOS小项目组成的合集，本文使用的调试app就是项目中的04 ToDo，clone下来打包一个ipa即可。当然也可以换成其他脱了壳或者没上架Appstore的ipa包，我用这个只是因为体积比较小，打包和重签名都比较快。



## 实现原理

注入和重签名都是iOS逆向老生常谈的话题，基于这两个技术也产生了大量围绕iOS开发者证书的黑灰产，例如光环助手等。`Fastbot-stub`要求在App中加入`fastbot-stub.framework`，合入依赖重新编译打包，就是注入动态库重签名这样的方案了。

- 注入：逆向修改三方应用,让三方应用执行我们的代码，这就是代码注入，动态库注入是一种方式。其中动态库注入分为**framework注入**与**dylib注入**。此处`fastbot-stub`编译产物为`.framework`。
- 重签名：说白了重签名是一个偷天换日的过程，需要真机编译一个别的工程得到一份描述文件，再把需要重签名app的MachO以及frameworks都重新上一遍签名，当然，成熟的重签名脚本Github上有很多了，原理什么的有兴趣可以细看。



## 具体步骤

#### 环境准备

此处需要准备好的东西：

- 一台Mac
- 一个砸壳后的ipa包
- 开发者证书，以及可用的描述文件
- yololib可执行文件
- LTResign可执行文件
- fastbot-stub.framework



##### 砸壳的ipa：

建议找你家开发直接打一个adhoc包，顺便要一份开发者证书，以及可用的描述文件。或者有代码权限的测试老哥们自己动手打一个，实在想用线上APP的需要砸壳，可以看看这个工具[iOS App 自动砸壳平台](https://www.dumpapp.com/)，或者通过第三方平台下ipa包，比如爱思助手、PP助手，最折腾但能学到东西的方案：搞一台越狱的iPhone用`clutch`自己砸



##### yololib可执行文件

```shell
git clone https://github.com/KJCracks/yololib

cd yololib && open yololib.xcodeproj
```

直接build即可，给构建产物可执行权限

```shell
chmod +x yololib
```

建议直接放入`/usr/local/bin`，毕竟这个东西执行的路径有点讲究

懒人方案：直接下载别人编译好的

>[KJCracks/yololib: dylib injector for mach-o binaries (github.com)](https://github.com/niexiaobo/yololib)



##### LTResign可执行文件

作者打包好了，可以直接下载，改权限

```shell
git clone https://github.com/gltwy/LTResign

cd LTResign && chmod +x LTResign
```



##### fastbot-stub.framework

按照`Fastbot-iOS`项目的Readme打开`Fastbot-iOS.xcworkspace`，编译即可，完整命令：

```shell
git clone git@github.com:bytedance/Fastbot_iOS.git

cd Fastbot-iOS && pod install --repo-update

open Fastbot-iOS.xcworkspace
```

然后target选择`fastbot-stub`，连一台真机编译，得到产物`fastbot-stub.framework`





### 动态库注入

将ipa包改名为zip包，其实下面这些步骤都可以在界面操作，看个人习惯

```shell
cp ToDo.ipa ToDo.zip
```

解压zip，得到Payload文件夹

```shell
unzip ToDo.zip
```

将`fastbot-stub.framework`复制到Payload/Todo.app/Frameworks中

```shell
cp -r fastbot_stub.framework  Payload/ToDo.app/Frameworks
```

运行yololib注入

```shell
cd Payload/ToDo.app && yololib Todo Frameworks/fastbot_stub.framework/fastbot-stub
```

这一步正常的输出是这样的：

>Reading binary: ToDo
>
>2021-11-11 22:09:37.127 yololib[20629:7615745] Thin 64bit binary!
>
>2021-11-11 22:09:37.128 yololib[20629:7615745] dylib size wow 88
>
>2021-11-11 22:09:37.128 yololib[20629:7615745] mach.ncmds 40
>
>2021-11-11 22:09:37.128 yololib[20629:7615745] mach.ncmds 41
>
>2021-11-11 22:09:37.128 yololib[20629:7615745] Patching mach_header..
>
>2021-11-11 22:09:37.128 yololib[20629:7615745] Attaching dylib..
>
>
>
>2021-11-11 22:09:37.128 yololib[20629:7615745] size 87
>
>2021-11-11 22:09:37.128 yololib[20629:7615745] complete!

这个作者打的日志十分的可爱，当你注入一个大点的库，第二行将会变成Fat Binary，胖的库

实际上yololib做的事情是让App运行的时候加载需要注入的动态库，只是写一条链接，而不会把这个库给带进`Frameworks`里面，所以这两步都是必要的。

重新把`Payload`打包为zip

```shell
cd ../.. && zip -r Payload.zip Payload
```

改名ipa

```shell
mv Payload.zip Payload.ipa
```

此时你拥有了一个被注入的ipa包，但他装不上去手机。



### 重签名

进到LTResign项目目录

```shell
mv Payload.ipa LTResign && cd LTResign
```

用`-l`参数运行`ltresign`获取证书id

```shell
./ltresign -l
```

输出类似这样子

>\1) 24D0F12312312312312312312312300E2CC990355 "Apple Development:XXXXXXXXXXXX"
>
> \2) 98D4E12312312312312312312312312312F9B013 "Apple Development: XXX XXX(XXXXXXXX)

这里面每一项的第一个字符串就是id了，比如`24D0F12312312312312312312312300E2CC990355`

将描述文件改名为`embedded.mobileprovision`，也放到这个目录

运行重签名脚本

```shell
ltresign -s /Payload.ipa -d 24D0F12312312312312312312312300E2CC990355 -m embedded.mobileprovision
```

正常的输出会是这样结尾：

>...........
>
>glt_tmp/glt_test/Payload/ToDo.app: replacing existing signature
>
>
>
>重新签名完成，可以去安装了！（2021-11-11 22:26:54）

此时这个目录将会生成一个重签名过的ipa `glt_output.ipa`



### 装包测试

```shell
ideviceinstaller -i glt_output.ipa
```

回到Fastbot-iOS工程，修改Fastbot-Runner的Scheme：

```shell
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

```shell
BUNDLEID=yigu.com.ToDo duration=240 throttle=300 xcodebuild test  -workspace Fastbot-iOS.xcworkspace -scheme FastbotRunner  -configuration Release  -destination 'platform=iOS,id=c6b0ab4fa8867c51cf1c5b6d8cd076d3957192b2' -only-testing:FastbotRunner/FastbotRunner/testFastbot
```

截取一段命令行输出：

```shell
[fastbot] : visit ToDo.InputViewController,UIApplicationRotationFollowingController,UIApplicationRotationFollowingControllerNoTouches,UICompatibilityInputViewController,UIInputViewController,UIInputWindowController; visited ViewController count is: 2 
[fastbot] : state visited: 30 
[fastbot] : action first visited, get reward 2.336364
[fastbot] : state is saturated, get reward 0.295547
[fastbot] : got reward: 9.3110
```

区别在哪？区别就在于有控件信息了，如果看到类似`UIInputWindowController`这样的字样，恭喜你，你成功了。



## 心得

注入搞fastbot其实就是个玩，有完整代码权限正儿八经打包进去才是正道。当然很多时候这套逆向方案也够用了，文中的命令也能很好的持续集成起来。



## 参考文章

[字节跳动质量利器 -- 移动端智能化稳定性测试工具 Fastbot-Android/iOS 双端重磅发布上线](https://testerhome.com/topics/31113)

[奔跑吧！智能Monkey之Fastbot跨平台 ](https://mp.weixin.qq.com/s/QhzqBFZygkIS6C69__smyQ)

[Issue #44 · bytedance/Fastbot_iOS](https://github.com/bytedance/Fastbot_iOS/issues/44)

[iOS逆向工具09-yololib注入framework](https://www.jianshu.com/p/0e80958eb1d8)

