---
title: Dojo local installation
icon: cloud-download-alt
layout: tutorial
overview: Discover the basics of creating a Dojo application.
paginate: false
group: getting_started
topic: install
---

> [tutorials/000_local_installation/index.md](https://github.com/dojo/dojo.io/blob/master/site/source/tutorials/000_local_installation/index.md)
>
> commit ef8cd9d90d326549aa3e6b43c2d4b78f846144d0

# 本地安装 Dojo

## 概述

本教程介绍如何在本地安装 Dojo 环境。

## 创建 Dojo 应用程序

首先，我们需要创建一个 Dojo 项目。 Dojo 为创建应用程序提供强大和先进的工具。 它提供了一个高效的命令行工具，来简化安装过程， 使用以下命令安装此工具：

```bash
npm install -g @dojo/cli
```

该命令会安装 Dojo 命令行工具（`@dojo/cli`），它简化创建、测试和构建应用程序等相关的开发任务。 `@dojo/cli` 工具默认自带两个选项：

* `ejet` - 将一个项目与 `@dojo/cli` 工具断开，允许高阶用户自定义配置
* `version` - 显示 `@dojo/cli` 版本信息和已安装的命令

运行 `dojo` 命令将显示所有可用的子命令，即使这些子命令还没有安装。当你尝试运行一个不可用的子命令时，CLI 会打印一条消息提示需要先安装此命令。

在创建你的第一个 Dojo 应用程序前，请先全局安装 `@dojo/cli-create-app` 命令，该命令用于创建项目模板：

```bash
npm install -g @dojo/cli-create-app
```

一旦命令安装完成，请运行以下命令创建你的 Dojo 项目：

```bash
dojo create app --name first-dojo-app
```

{% aside 'Dojo create arguments' %}
传入 `dojo create app` 的很多参数都有缩写版本。所以 `dojo create app -n first-dojo-app` 等价于 `dojo create app --name first-dojo-app`。
{% endaside %}

该命令将在新建的 “first-dojo-2-app” 文件夹下创建 Dojo 应用程序的基本项目结构，并预安装的所有依赖库。

就是这么简单，此刻我们已成功创建了第一个基本的 Dojo 应用程序，并安装了它的依赖库。 现在让我们看看新建的应用程序能做什么！ 第一步，我们使用另一个 `@dojo/cli` 命令。 您不需要自己动手去安装这个命令，它会随着其他依赖一起安装。 在终端切换到 `first-dojo-app` 文件夹，然后输入 `dojo build --mode dev --watch --serve` 命令：

```bash
cd first-dojo-app
dojo build --mode dev --watch --serve
```

(或简写为 `dojo build -m dev -w -s`)

{% aside 'Production build as a default' %}
`@dojo/cli-build-app` 命令默认使用 `--mode dist` 参数创建一个可用于生产环境的构建。提供的 `--mode dev` 参数将指示该命令创建一个对开发友好的构建，方便调试和不间断的开发。
{% endaside %}

此命令将触发 Dojo 工具使用 [Webpack](https://webpack.github.io/) 构建和 transpile 项目， Webpack 是一款优秀的 JavaScript 源码优化工具。 `--watch`（或 `-w`） 标志用于监听变化，当有任何变化发生并保存到硬盘后，就立即重新构建项目；而 `--serve` （或 `-s`）标志会启动一个轻量级的 web 服务器，这样当我们修改一些源码后，可在浏览器中即时运行程序。

为了查看应用程序的功能， 请打开一款主流的 web 浏览器（如最新版的 Chrome，Edge，Firefox，Internet Explore 或 Safari）， 并导航到 [http://localhost:9999](http://localhost:9999)。 您将看到一个简陋的页面，欢迎你访问新建的应用程序。