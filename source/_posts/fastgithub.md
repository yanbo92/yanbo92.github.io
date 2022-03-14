---
title: Mac电脑使用FastGithub加速并配置命令行代理、开机自动启动
date: 2021-10-24 12:26
tags: 
- github
- fastgithub
categories: 
- Mac
---




## 最终效果

浏览器、命令行低延迟访问github，刷项目、拉依赖访问正常

<!-- more -->




## 项目介绍
> FastGithub是一款github加速神器，解决github打不开、用户头像无法加载、releases无法上传下载、git-clone、git-pull、git-push失败等问题。


#### 项目主页
[FastGithub](https://github.com/dotnetcore/FastGithub)



#### 加速实现原理

>####  windows
>
>1. 客户端访问`https://github.com`
>2. 客户端向dns查询github.com的ip，FastGithub拦截dns数据包并伪造解析结果为127.0.0.1
>3. 客户端请求到FastGithub的`https://127.0.0.1:443`
>4. FastGithub使用fastgithub.cer颁发服务器证书给客户端
>5. FastGithub查询和计算github.com最快的ip
>6. FastGithub与github.com进行无sni的tls连接
>7. FastGithub将请求反向代理到`https://github.com`
>
>#### linux/osx
>
>1. 客户端访问`https://github.com`
>2. 客户端使用fagithub的代理端口38457代理请求
>3. FastGithub将代理的流量请求到自身的反向代理服务
>4. FastGithub使用fastgithub.cer颁发服务器证书给客户端
>5. FastGithub查询和计算github.com最快的ip
>6. FastGithub与github.com进行无sni的tls连接
>7. FastGithub将请求反向代理到`https://github.com`





## 配置步骤

#### 下载、解压文件

下载链接中的最新osx压缩包，命名类似`fastgithub_osx-x64.zip`

[Releases](https://github.com/dotnetcore/fastgithub/releases)





#### 安装证书

- 解压后，双击运行Unix可执行文件`fastgithub`，同目录下将会生成cert目录
- 双击cert目录中的`fastgithub.cer`证书文件，并设置为信任方式为始终信任

![fastgithub-cert](/images/fastgithub-cert.png)



#### 设置代理

系统代理

- 打开设置-网络，选择使用的网络模式，比如wifi
- 点击左下角黄色锁，解锁
- 点击高级，代理，勾选自动代理配置
- 填入URL：`http://127.0.0.1:38457`

- 点击右下角应用

![fastgithub-system-proxy](/images/fastgithub-system-proxy.png)



命令行代理环境变量

- 编辑`~/.zshrc`或者`~/.bashrc`，这取决于你的shell用哪种
- 加入一行：

```
export https_proxy=http://127.0.0.1:38457 http_proxy=http://127.0.0.1:38457
```

- 保存，并运行`source ~/.zshrc`或者`source ~/.bashrc`



#### 配置开机自启

- 打开设置-用户与群组，选择用户
- 选择登陆项，点击`+`符号，选择`fastgithub`的Unix可执行文件，并勾选隐藏。

![fastgithub-auto-boot](/images/fastgithub-auto-boot.png)

