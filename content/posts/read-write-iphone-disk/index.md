---
title:  通过ifuse/tidevice库读写iPhone手机硬盘
date: 2021-11-07
tags:
- iOS
- libimobiledevice
- ifuse
- tidevice
- osxfuse
- macfuse
- python
categories:
- Mobile
image: "cover.png"
url: "read-write-iphone-disk"
---


## 背景

在自动化测试的交付、执行过程中，偶尔需要对iPhone手机的硬盘进行读写操作，比如删除相册，拷贝素材等等。本文介绍两个通过命令行操作iPhone手机硬盘的工具，分别是`ifuse`和`tidevice`。

<!-- more -->



## 工具介绍

#### MacFuse

> ## What is macFUSE?
>
> macFUSE allows you to extend macOS's native file handling capabilities via third-party file systems.
>
> macFUSE允许你通过第三方文件系统扩展macOS的本地文件处理能力。
>
> ### Features
>
> As a user, installing the macFUSE software package will let you use any third-party FUSE file system. Legacy MacFUSE file systems are supported through the optional MacFUSE compatibility layer.
>
> As a developer, you can use the FUSE SDK to write numerous types of new file systems as regular user space programs. The content of these file systems can come from anywhere: from the local disk, from across the network, from memory, or any other combination of sources. Writing a file system using FUSE is orders of magnitude easier and quicker than the traditional approach of writing in-kernel file systems. Since FUSE file systems are regular applications (as opposed to kernel extensions), you have just as much flexibility and choice in programming tools, debuggers, and libraries as you have if you were developing standard macOS applications.
>
> 作为一个用户，安装macFUSE软件包可以让你使用任何第三方FUSE文件系统。传统的MacFUSE文件系统通过可选的MacFUSE兼容层得到支持。
>
> 作为开发者，你可以使用 FUSE SDK 来编写众多类型的新文件系统，作为常规的用户空间程序。这些文件系统的内容可以来自任何地方：来自本地磁盘，来自整个网络，来自内存，或任何其他来源的组合。使用FUSE编写文件系统比编写内核内文件系统的传统方法要容易得多，也快得多。由于 FUSE 文件系统是普通的应用程序（而不是内核扩展），你在编程工具、调试器和库方面的灵活性和选择与开发标准的 macOS 应用程序一样多。
>
> 项目主页：[osxfuse.github.io](https://osxfuse.github.io/)
>
> 安装：
>
> ```shell
> brew install osxfuse --cask
> ```



#### ifuse

>*A fuse filesystem implementation to access the contents of iOS devices.*
>
>一个Fuse文件系统的实现，用于访问iOS设备的内容。
>
>## Features
>
>This project allows mounting various directories of an iOS device locally using the [FUSE file system interface](https://github.com/libfuse/libfuse).
>
>这个项目允许使用[FUSE文件系统接口]在本地挂载iOS设备的各种目录。
>
>Some key features are:
>
>- **Media**: Mount media directory of an iOS device locally
>- **Apps**: Mount sandbox container or document directory of an app
>- **Jailbreak**: Mount root filesystem on jailbroken devices *(requires AFC2 service)*
>- **Browse**: Allows to retrieve a list of installed file-sharing enabled apps
>- **Implementation**: Uses [libimobiledevice](https://github.com/libimobiledevice/libimobiledevice) for communication with the device
>
>主要特性如下
>
>- **媒体**。在本地安装iOS设备的媒体目录
>- **Apps**。挂载一个应用程序的沙盒容器或文档目录。
>- **越狱**。在已越狱的设备上挂载根文件系统 *（需要AFC2服务）*。
>- **浏览**。允许检索已安装的支持文件共享的应用程序的列表
>- **实施**。使用[libimobiledevice]与设备进行通信
>
>
>
>项目主页：
>
>安装：
>
>```shell
>brew install ifuse
>```
>
>如果装不上，把系统时间改到2021年4月6日之前即可。但时间改了之后又会出现443:
>
>```
>curl: (35) LibreSSL SSL_connect: SSL_ERROR_SYSCALL in connection to ghcr.io:443 
>```
>
>所以正确的操作流程是要在通过了brew校验这个包可以装之后再改回正常的时间，以通过下载包所需要的SSL校验，毫秒级操作，不过多试几次就能装上去。
>
>此项目依赖osxfuse，请先安装上面的osxfuse。





#### tidevice

> 该工具能够用于与iOS设备进行通信, 提供以下功能
>
> - 截图
> - 获取手机信息
> - ipa包的安装和卸载
> - 根据bundleID 启动和停止应用
> - 列出安装应用信息
> - 模拟Xcode运行XCTest，常用的如启动WebDriverAgent测试（此方法不依赖xcodebuild)
> - 获取指定应用性能(CPU,MEM,FPS)
> - 文件操作
> - 其他
>
> 支持运行在Mac，Linux，Windows上
>
> 
>
> 项目主页：[alibaba/taobao-iphone-device](https://github.com/alibaba/taobao-iphone-device)
>
> 安装：
>
> ```shell
> pip3 install -U "tidevice[openssl]"
> ```





## 实现原理

- 通过阅读源码可以得知，`ifuse`依赖`libimobiledevice`，同时是基于`usbmux`实现的，换言之，该库只支持通过电脑usb连接手机来通信。ifuse将手机硬盘的某一部分挂载到电脑硬盘上，再通过正常的文件读写操作去控制。
- 而`tidevice`则是把`instruments`协议整个用python实现了一遍，这是支持**无线**连接手机的。并且开发者将常用文件操作封装了一层，支持` rm cat pull push stat tree rmtree mkdir ls`  等操作，有点adb的味道。



## 具体实现代码

#### ifuse读写手机内存

首先新建一个mount_point目录用于挂载手机硬盘：

```
mkdir mount_point
```

然后用默认参数挂载，这样将会挂载整个`Media`目录，可以拿到`DCIM`以及`Downloads`之类的目录内容：

```
MacBook-Pro ~ % ifuse -u c6b0ab4fa8867c51cf1c5b6d8cd076d3957192b2 mount_point
MacBook-Pro ~ % ls mount_point
AirFair		MediaAnalysis	Purchases	afk(1).zip	rd
Books		PhotoData	Radio		afk_screenshots
DCIM		Photos		Recordings	general_storage
Downloads	PublicStaging	afk		iTunes_Control
```

可以像自己电脑上的目录一样去读写，比如`rm`，`mkdir`之类的都没问题。

卸载掉：

```
umount mount_point
```

还有另一种更强力的卸载，毕竟这个库经常会卸载不掉：

```
diskutil unmount force mount_point
```

再试试指定APP包名的挂载方式，此处用的是Alook浏览器`com.ld.TakeBrowser`：

```
MacBook-Pro ~ % ifuse --documents com.ld.TakeBrowser -u c6b0ab4fa8867c51cf1c5b6d8cd076d3957192b2 mount_point
MacBook-Pro ~ % ls mount_point
Audios		Images		Videos
Documents	Others		Zipped
```

这里挂载了APP的Documents目录，同样可以使用的参数还有`--container`





#### tidevice读写手机内存

首先新建一个txt用于调试

```
touch file.txt
```

类似的，查看`Media`下的目录：

```
MacBook-Pro ~ % tidevice fsync ls /DCIM/
['.DS_Store',
 '104APPLE',
 '._.DS_Store',
 '103APPLE',
 '102APPLE',
 '.MISC',
 '101APPLE',
 '100APPLE']
```

删除一张照片：

```
MacBook-Pro ~ % tidevice fsync rm /DCIM/104APPLE/IMG_4334.JPG
<AFCStatus.SUCCESS: 0>
```

操作特定的APP目录，需要使用`-B`参数：

ls操作：

```
MacBook-Pro ~ % tidevice fsync -B com.ld.TakeBrowser ls /Documents
['Zipped', 'Documents', 'Videos', 'Others', 'Images', 'Audios']
```

push操作：

```
MacBook-Pro ~ % tidevice fsync -B com.ld.TakeBrowser push file.txt /Documents/
pushed to /Documents/
```

类似的支持的操作还有如下几个：

```
{rm,cat,pull,stat,tree,rmtree,mkdir
```



#### 使用建议

个人认为在功能没有差异的情况下，使用`tidevice`要比`ifuse`方便得多，有如下原因

- 基于Python实现，相对于ifuse使用C的实现更适合脚本集成，可以直接用python的`import`使用，无需使用类似`os.system()`这种粗暴的方式
- `tidevice`无需挂载到本机，实际使用起来`ifuse`经常出现某个目录卸载失败，需要重启电脑这样的情况，会出现`Input/Output Error`，也就是规避掉了关闭通道异常这种风险
- `tidevice`基于`instruments`实现，支持无线连接手机，当你的测试手机USB口被占用时，`tidevice`是唯一的选择。

