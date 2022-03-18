---
title: 使用Dockerfile和docker-compose搭建Sonar+PostgreSQL代码扫描服务
date: 2021-10-24 20:21
tags:
- docker
- dockerfile
- docker-compose
- sonar
- postgreSQL
categories: 
- Docker
---



## 项目背景

- 由于调试插件冲突需要，经常重装Sonar，但插件又要重新安装，于是自定义一个Dockerfile来预装我需要的插件，做成镜像
- 另外，Sonar自带的数据库较弱，官方只建议调试使用，并且会永远有一个Banner提示你更换数据库，所以使用docker-compose把Sonar和PostgreSQL数据库的镜像编排起来，一起搭建
- 默认环境：已安装docker和docker-compose
<!-- more -->



## 操作步骤



#### 新建项目目录

```shell
mkdir sonar_postgres && cd sonar_postgres
```





#### 新建Dockerfile文件和docker-compose文件

```shell
touch Dockerfile && touch docker-compose-yml
```





#### 编辑Dockerfile，预装Sonar插件

Dockerfile文件内容：

```dockerfile
FROM sonarqube:8.9-community
RUN wget -P /opt/sonarqube/extensions/plugins/ https://github.com/xuhuisheng/sonar-l10n-zh/releases/download/sonar-l10n-zh-plugin-8.9/sonar-l10n-zh-plugin-8.9.jar
RUN wget -P /opt/sonarqube/extensions/plugins/ https://github.com/tal-tech/sonar-swift/releases/download/1.5.1/tal-sonar-swift-plugin-1.5.1.jar
RUN wget -P /opt/sonarqube/extensions/plugins/ https://github.com/detekt/sonar-kotlin/releases/download/2.3.0/sonar-detekt-2.3.0.jar
```

使用最新版本的Sonar Community版，安装中文插件、swift插件以及kotlin插件。此处使用最粗暴的下载、拷贝jar文件的安装方法。通过挂载存储也可以实现相同的功能。





#### 根据Dockerfile构建镜像

```shell
docker build -t sonarqube_plugins:v1 .
```

此处`-t`后面的第一个参数为镜像名字以及版本，第二个参数`.`为Dockerfile所在目录，下面使用docker-compose时将会用到镜像名字以及版本





#### 使用docker-compose编排镜像

编辑文件docker-compose.yml：

```
version: "3"

services:
  sonarqube:
    image: sonarqube_plugins:v1
    expose:
      - 9000
    ports:
      - "9000:9000"
    networks:
      - sonarnet
    environment:
      - sonar.jdbc.url=jdbc:postgresql://db:5432/sonar
      - sonar.jdbc.username=sonar
      - sonar.jdbc.password=sonar


  db:
    image: postgres
    networks:
      - sonarnet
    environment:
      - POSTGRES_USER=sonar
      - POSTGRES_PASSWORD=sonar

networks:
  sonarnet:
```





#### 启动容器

```shell
docker-compose up -d
```

此时在浏览器访问localhost:9000就能看到Sonar了







