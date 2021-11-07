---
title:  通过Extended-Choice-Parameter插件将Jenkins参数选项动态设置为当前连接的iOS/Android手机(二)
date: 2021-11-7 00:35
tags:
- jenkins
- jenkins插件
- 自动化
- UI自动化
- Extended-Choice-Parameter
- tidevice
- 持续集成
- CI/CD
- groovy
categories:
- Jenkins
---


## 背景

在[通过Extended-Choice-Parameter插件将Jenkins参数选项动态设置为当前连接的iOS/Android手机)](https://yanbo92.site/jenkins-extended-choice-parameter/)中，介绍了如何简单的把手机选择参数做成实时显示当前连接的设备列表，但实际使用下来，发现有以下两个小问题

- 列表只能显示手机的id，可读性极差，仍然难以避免另外去查询手机名字以对应上id
- iOS列表基于idevice_id实现，其底层连接依赖USB，也就是说无法做到识别无线设备

基于以上考虑，进行一次细节优化
<!-- more -->





## 工具介绍

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
> 安装：
>
> ```shell
> pip3 install -U "tidevice[openssl]"
> ```





## 实现原理

- Jenkins有许多插件支持动态参数列表，比如`Extended-Choice-Parameter`，又比如`Active Choice`，综合试用了一下选择了前者。
- [（一）](https://yanbo92.site/jenkins-extended-choice-parameter/)文中只在参数的Value中编写了Groovy脚本动态获取手机id列表，而Value Description也可以通过Groovy来获取手机的名字，直接显示具体的型号
- 把查询手机名称的命令，也就是`adb devices -l`或者`tidevice list`写成Groovy脚本传入`Extended-Choice-Parameter`的`Value Description`里



## 具体实现代码

#### Android

如果你的手机都插在主节点`master`上，可以用以下代码，填入`Choose Source for Value  Description - Groovy Script`

```groovy
String content = "adb devices -l".execute().text
String[] str;
str = content.split('\n'); 
def result = [];
for( String values : str ){
    if(values.contains("\tdevice")){
        result.add(values.replaceAll("\tdevice",""))
    }
}
result
```



但如果你的主节点和执行节点并不是同一个，需要在执行节点上去做动态获取，那么代码是这样的

```groovy
import hudson.util.RemotingDiagnostics
import jenkins.model.Jenkins

String agent_name = '节点名'
groovy_script = '''
println "adb devices -l".execute().text
'''.trim()

def result = []
Jenkins.instance.slaves.find { agent ->
    agent.name == agent_name
}.with { agent ->
    String[] str;
	str = RemotingDiagnostics.executeGroovy(groovy_script, agent.channel).split('\n');
	for( String values : str ){
	    if(values.contains("\tdevice")){
	        result.add(values.replaceAll("\tdevice",""))
	    }
	}
}
result
```



#### iOS

类似的，把`adb device`替换为`idevice_id -l`，当然也可以用`tidevice list`，只是要对参数做截断处理

```groovy
String content = "tidevice list".execute().text
String[] str;
str = content.split('\n'); 
def result = [];
for( String values : str ){ 
    result.add(values)
}
result
```



master-slave模式代码：

```groovy
import hudson.util.RemotingDiagnostics
import jenkins.model.Jenkins

String agent_name = '节点名'
groovy_script = '''
println "tidevice list".execute().text
'''.trim()

def result = []
Jenkins.instance.slaves.find { agent ->
    agent.name == agent_name
}.with { agent ->
    String[] str;
	str = RemotingDiagnostics.executeGroovy(groovy_script, agent.channel).split('\n');
	for( String values : str ){
	    result.add(values)   
	}
}
result
```



## 参考文章

[Jenkins 动态获取安卓设备作为参数 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/148957030)

[通过Extended-Choice-Parameter插件将Jenkins参数选项动态设置为当前连接的iOS/Android手机)](https://yanbo92.site/jenkins-extended-choice-parameter/)

