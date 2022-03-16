---
title: 重签名解决光环助手频繁掉签问题
date: 2022-2-11
tags:
- github
- yololib
- ios
- 开源项目
- 光环助手
- 重签名
- 逆向
- iOS逆向
- 砸壳
- itms
- 破解
- 游戏
categories:
- iOS
weight: 1
---



## 最终效果

- 付费证书只要仍在有效期内，账号内的设备都能不掉签使用游戏的`ipa`包
- 免费证书七天一次签名，可以用[爱思助手](i4.cn)之类的工具完成这个过程。



## 背景介绍

>光环助手，隶属于广州加兔网络科技有限公司，是一款基于Android平台的多功能卡牌游戏助手，由光环团队制作，致力于为众手游玩家打造最优质的游戏氛围，成就最强卡牌管家。光环助手不但能节省手游玩家繁杂的游戏时间，带来更好更畅快的游戏体验；更会为玩家实时更新最新游戏资讯，搜罗大量游戏攻略以及“小编带你玩”等诸多精品栏目。
>
>光环修改版的游戏提供以下悬浮窗功能
>
>- 内置攻略
>- 内置15倍可调节加速
>- 内置连点器，支持调节频率，次数以及保存脚本功能



## 操作教程

### 手动找出ipa地址

1. 使用电脑或者将手机浏览器`UA`设置为`PC`进入[iOS光环助手下载页](https://www.ghzs6.com/web/ios_column_h5/index.html#/)

2. 找到想要下载的游戏，点击下载
3. 此时地址栏会暴露一个`itms-services`的`url`，这是一种无需经过`App Store`来分发`ipa`的服务
4. 比如`url`为

```
itms-services://?action=download-manifest&url=https://ios-api.ghzs.com/install-plist/61f36f94ef0f37e259a8be73.plist
```

5. 取**参数**中的`url`部分，也就是这个`plist`文件的地址

```
https://ios-api.ghzs.com/install-plist/61f36f94ef0f37e259a8be73.plist
```

6. 直接用浏览器访问，将得到一个`xml`页面

```xml
This XML file does not appear to have any style information associated with it. The document tree is shown below.
<plist version="1.0">
<dict>
<key>items</key>
<array>
<dict>
<key>assets</key>
<array>
<dict>
<key>kind</key>
<string>software-package</string>
<key>url</key>
<string>https://ios-d.ghzs.com/ipa/61f36ec83feb2f4224555b8a.ipa</string>
</dict>
<dict>
<key>kind</key>
<string>display-image</string>
<key>url</key>
<string>https://ios-image.ghzs.com/app-icon/61f36f7d3feb2f4224555b8e.png</string>
</dict>
<dict>
<key>kind</key>
<string>full-size-image</string>
<key>url</key>
<string>https://ios-image.ghzs.com/app-icon/61f36f7d3feb2f4224555b8e.png</string>
</dict>
</array>
<key>metadata</key>
<dict>
<key>bundle-identifier</key>
<string>com.gh.snsgz2appstore3</string>
<key>bundle-version</key>
<string>1.36.89</string>
<key>kind</key>
<string>software</string>
<key>title</key>
<string>少年三国志2（安装完成后，请到设置->通用->描述文件与设备管理，信任企业级应用）</string>
</dict>
</dict>
</array>
</dict>
</plist>
```

7. `xml`中的`url`即`ipa`下载地址的值了，再贴回地址栏，即可下载到一个被光环助手注入的游戏

### 重签名

麻烦移步我写的另一篇文章，有详细步骤，这里就不重复写了。

[重签名Fastbot-sub](https://yanbo92.site/fastbot-stub-inject)

## 参考链接

[光环助手官网](https://www.ghzs.com/)

[iOS光环助手下载页](https://www.ghzs6.com/web/ios_column_h5/index.html#/)

[爱思助手官网](i4.cn)

[签名工具Altstore](altstore.io)
