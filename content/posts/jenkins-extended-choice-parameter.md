---
title:  通过Extended-Choice-Parameter插件将Jenkins参数选项动态设置为当前连接的iOS/Android手机
date: 2021-11-1 23:35
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

在自动化测试的交付、执行过程中，很难绕开当下大热的CI/CD工具Jenkins。而Jenkins中原生的参数类型比较单一，难以实现动态化。为了交付一些UI自动化脚本，测试工程师需要先选择执行手机、执行版本等等内容，其中选择执行手机这一步骤如果只用文本参数来实现，用户的使用场景类似这样：

>1. 开命令行，执行`adb device `或者`idevice_id -l`
>2. 将手机id粘贴到Jenkins执行参数中，运行

本文提供一种一步到位的选择参数实现。
<!-- more -->



## 工具介绍

#### Jenkins

> Jenkins是一个开源的、提供友好操作界面的持续集成(CI)工具，起源于Hudson（Hudson是商用的），主要用于持续、自动的构建/测试软件项目、监控外部任务的运行（这个比较抽象，暂且写上，不做解释）。Jenkins用Java语言编写，可在Tomcat等流行的servlet容器中运行，也可独立运行。通常与版本管理工具(SCM)、构建工具结合使用。常用的版本控制工具有SVN、GIT，构建工具有Maven、Ant、Gradle。



#### Extended-Choice-Parameter

>Extended-Choice-Parameter是一款Jenkins插件，主要功能为对选择参数进行扩展，用于实现多选参数、实现选项动态获取等更强大的功能



#### Groovy

>Groovy是一种基于[JVM](https://baike.baidu.com/item/JVM)（[Java虚拟机](https://baike.baidu.com/item/Java虚拟机)）的敏捷开发语言，它结合了[Python](https://baike.baidu.com/item/Python)、[Ruby](https://baike.baidu.com/item/Ruby/11419)和[Smalltalk](https://baike.baidu.com/item/Smalltalk)的许多强大的特性，Groovy 代码能够与 Java 代码很好地结合，也能用于扩展现有代码。由于其运行在 JVM 上的特性，Groovy也可以使用其他非Java语言编写的库。



## 实现原理

- Jenkins有许多插件支持动态参数列表，比如`Extended-Choice-Parameter`，又比如`Active Choice`，综合试用了一下选择了前者。
- 把查询手机id的命令，也就是`adb devices`或者`idevice_id -l`写成Groovy脚本传入`Extended-Choice-Parameter`的`Value`里



## 具体实现代码

#### Android

如果你的手机都插在主节点`master`上，可以用以下代码，填入`Choose Source for Value - Groovy Script`

```groovy
String content = "adb devices".execute().text
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
println "adb devices".execute().text
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
String content = "idevice_id -l".execute().text
String[] str;
str = content.split('\n'); 
def result = [];
for( String values : str ){ 
    result.add(values.replaceAll("\tdevice",""))
}
result
```



## 参考文章

[Jenkins 动态获取安卓设备作为参数 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/148957030)
