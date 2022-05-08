---
title:  Auto.JS中杀死其他脚本的方法
date: 2021-10-28
tags:
- auto.js
- android
- 自动化
- UI自动化
- 测试开发
- javascript
categories: 
- Mobile
image: "cover.png"
url: "autojs-kill-scripts"
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

- 调用`engines.all()`方法获取当前正在运行的所有引擎对象（返回对象数组）
- 再调用`engines.myEngine()`方法获取当前正在前台运行这个方法的对象（返回单个对象）
- 进行对象比对后再调用`engine.forceStop()`方法杀死非当前引擎对象
- 该方法能避免用户反复运行脚本，导致脚本互相干扰、资源占用等情况



#### 完整代码

```javascript
function kill_scripts() {
    allNgs = engines.all()
    myNg = engines.myEngine()
    for (var i = 0; i < allNgs.length; ++i) {

        if (!(allNgs[i] === myNg)) {
            allNgs[i].forceStop()
        }

    }
}


kill_scripts()
```



