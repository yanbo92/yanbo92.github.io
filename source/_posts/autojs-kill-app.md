---
title:  Auto.JS中免Root杀死APP的巧妙方法
date: 2021-10-25 23:35
tags:
- auto.js
- android
- 自动化
- UI自动化
categories: 
- Android
---



## Auto.JS介绍



>#### Auto.js Pro是什么
>
>一个在Android、鸿蒙平台编写、运行JavaScript代码的集成开发环境，包括代码补全的编辑器、单步调试、图形化设计，可构建为独立apk应用，也可连接电脑开发。
>
>
>
>#### Auto.js Pro能做什么
>
>创建自动化工具、效率工具、优美界面、小应用，诸如早晨自动签到、处理文件为excel、图片批量处理、机器人、自动化测试、搭建服务器等，或解放双手，或学习编程，或制作应用。
>
>
>
>#### 为什么选择Auto.js Pro
>
>完善的文档和示例、丰富的API、增强的加密、活跃的更新，用JavaScript连接Java、Android、Node.js的生态。

注：Auto.JS与Auto.JS Pro的区别在于，前者是免费、开源的，但是已经停更多年，后者需要付费，但仍在持续更新，提供更多的功能。Auto.js 开源版本已不再维护(原因参见Auto.js Pro FAQ)，后续将只维护Auto.js Pro专业版。
<!-- more -->



#### 官网链接

[Auto.JS](https://hyb1996.github.io/AutoJs-Docs/#/)

[Auto.JS Pro](https://pro.autojs.org/)





## 杀死APP具体实现

#### 实现原理

- 调用`openAppSetting(packageName)`方法打开系统设置中对应APP的设置页
- 通过控件操作点击*强制停止*，*确认*等按钮，达到杀死APP的目的
- 该方法巧妙的规避了通过`adb shell kill`或者 `adb shell am force-stop`方法带来的权限问题



#### 完整代码

```javascript
function kill_app(packageName) {
    var name = getPackageName(packageName);
    if (!name) {
        if (getAppName(packageName)) {
            name = packageName;
        } else {
            return false;
        }
    }
    app.openAppSetting(name);
    text(app.getAppName(name)).waitFor();
    let is_sure = textMatches(/(.*强.*|.*停.*|.*结.*|.*行.*)/).findOne();
    if (is_sure.enabled()) {
        textMatches(/(.*强.*|.*停.*|.*结.*|.*行.*)/).findOne().click();
        textMatches(/(.*确.*|.*定.*)/).findOne().click();
        log(app.getAppName(name) + "应用已被关闭");
        sleep(1000);
        back();
    } else {
        log(app.getAppName(name) + "应用不能被正常关闭或不在后台运行");
        back();
    }
}

kill_app('微信')
```



