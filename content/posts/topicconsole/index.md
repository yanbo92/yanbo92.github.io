---
title: "TopicConsole：一个轻量的 Apache Pulsar 桌面客户端"
date: 2026-10-28
tags:
- Pulsar
- 开源项目
- 桌面应用
- Wails
- Go
- React
categories:
- 开源项目
url: "topicconsole"
---

## 背景

平时查 Pulsar 消息，最常见的路子都不太轻：要么拉一个带服务端的 Web 管理台，要么装几百 MB、还要配 JVM 或 Docker 的重型客户端。只是想看一眼某个 topic 里跑了什么消息，却要先起一整套环境，实在没必要。

TopicConsole 就是为这个场景做的：一个跨平台桌面客户端，**单个可执行文件约 15 MB**，直接连 Pulsar broker，不用 Docker、不用 JVM、不用云账号。打开就能看。

项目地址：官网 [topicconsole.vercel.app](https://topicconsole.vercel.app/)（有中文版，[topicconsole.vercel.app/zh-cn/](https://topicconsole.vercel.app/zh-cn/)），源码在 [yanbo92/topicconsole](https://github.com/yanbo92/topicconsole)

## 它能做什么

### 实时看消息，两种模式

- **Observer 模式**：用 Pulsar Reader 读消息，不创建持久订阅。只想看、不想留副作用的时候用这个，查完就走，不会在 broker 上留下游标。
- **订阅模式**：走正常订阅消费，自己控制什么时候 ack，适合需要确认消费进度的场景。

消息流跑起来之后，可以边跑边用正则按 key / value 过滤；多个 topic 可以开多个标签页并排看；消息缓冲能导出成 JSON 或 CSV。列表做了虚拟化，5 万条以上消息也不会卡。

### 管理操作

在主界面可以直接浏览租户和命名空间，管理 topic：

- 创建非分区或分区持久化 topic，删除非分区持久化 topic
- 看命名空间级别的 topic、生产者、消费者、订阅和积压数量
- 从 Pulsar Admin 列出 topic 的订阅
- 生产消息：填 key、value、headers 就能发；如果本地有 Avro schema，JSON 值会自动按 schema 编码

### 主题组织

topic 一多就乱，项目里做了三样东西对付这个：

- **置顶 topic**：常用的 star 一下，永远排在最前面
- **Topic 分组**：按名字归到文件夹里
- **高级搜索**：从头扫描，限定匹配数量，支持正则；按官网的说法，还能按消息 key、载荷文本、JSONPath / JMESPath 和发布时间过滤

### Broker 连接

多个 broker 放在一个列表里管理，顶部栏或 ⌘K / Ctrl+K 就能瞬间切换，正在跑的会话不受影响。每个 broker 可以存多套凭证，切换在 dashboard 里点一下就行；配置（包括分组和置顶）可以导出成 JSON 备份，也能再导回来。

## 技术栈

| 层 | 技术 |
|---|---|
| 原生窗口与 RPC 桥 | Wails v2 |
| Pulsar 客户端 | pulsar-client-go |
| UI 组件 | shadcn/ui + Tailwind CSS |
| 状态管理 | Zustand |
| 前端构建 | Vite + React 18 + TypeScript |
| 列表虚拟化 | @tanstack/react-virtual |

整体结构也简单：`main.go` 启动 Wails 并绑定一个 `App`，`app.go` 是前后端 RPC 的边界，`internal/` 放后端领域逻辑，`frontend/src/` 是 React 界面。想改功能的话，先看仓库里的 `ARCHITECTURE.md`，包结构和事件流都写在里面了。

## 上手

去 [Releases](https://github.com/yanbo92/topicconsole/releases) 页下载对应平台的包：

| 平台 | 文件 |
|---|---|
| macOS（Apple Silicon） | `topicconsole-darwin-arm64.zip` |
| macOS（Intel） | `topicconsole-darwin-amd64.zip` |
| Windows | `topicconsole-amd64-installer.exe`（需要 WebView2，Win11 自带） |
| Linux | `topicconsole-linux-amd64.tar.gz` |

macOS 第一次打开如果被 Gatekeeper 拦住，右键点「打开」即可；如果提示「已损坏」，清掉隔离属性：

```bash
xattr -cr /path/to/TopicConsole.app
```

打开之后按 ⌘,（macOS）或 Ctrl+, 打开设置，点 Add Broker 填 Pulsar 的 service URL 和 admin URL，Test Connection 通过后保存；左侧选租户和命名空间，点一个 topic，选 Observe 或 Consume 就能开始看消息了。

从源码构建也不麻烦，需要 Go 1.25+、Node.js 18+ 和 Wails CLI：

```bash
git clone https://github.com/yanbo92/topicconsole.git
cd topicconsole

make dev        # 开发模式，热重载
make build      # 编译当前平台
```

本地起一个 Pulsar standalone 做测试可以用 `make pulsar-up`（走 Docker）。

## 两点提醒

一是项目目前还是 `v0.1.x`，早期版本，API 和界面在 1.0 之前都可能变，遇到问题欢迎去仓库提 issue。

二是配置里的 Pulsar token 是**明文**存在 `.topicconsole.json` 里的（在当前工作目录有这个文件就用它，否则放 `$HOME/.topicconsole.json`）。备份和查看都方便，但也要清楚：能读到这个文件的人就能读到你的 token，别把它随手发给别人。

## 最后

这个项目是从 [kafkalet](https://github.com/sneiko/kafkalet)（一个 Kafka 的轻量客户端）衍生来的，思路一样：看消息这件事，本来就该轻一点。如果你也在用 Pulsar，又受够了重型客户端，可以下来试试；觉得哪里不好用，直接去仓库提 issue 或者 PR 就行。

项目用 AGPLv3 开源。
