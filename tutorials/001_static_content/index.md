---
title: Your first Dojo application
icon: magic
layout: tutorial
overview: Create your first Dojo application and use it to print a simple message in the browser.
paginate: true
group: getting_started
topic: create
---

> [tutorials/001_static_content/index.md](https://github.com/dojo/dojo.io/blob/master/site/source/tutorials/001_static_content/index.md)
>
> commit 3e0f3ff1ed392163bc65e9cd015c4705cb9c586e

{% section 'first' %}

# 你的第一个 Dojo 应用程序

## 概述

通过学习本教程，你将了解如何创建你的第一个 Dojo 程序，并使用它在浏览器上打印一段简短信息。

## 前提

你可以打开 [codesandbox.io 上的教程](https://codesandbox.io/s/github/dojo/dojo.io/tree/master/site/source/tutorials/001_static_content/demo/initial/biz-e-corp) 或者 [下载](../assets/001_static_content-initial.zip) 示例项目，然后运行 `npm install`。

已全局安装 `@dojo/cli` 命令行工具。 参考 [本地安装 dojo](../000_local_installation/) 文章了解更多信息。

你也需要熟悉 TypeScript 语言， 因为 Dojo 是基于 TypeScript 语言开发的。

{% section %}

## 启动开发服务器

{% task '构建并运行应用程序。' %}

在开始修改源代码之前，我们先启动开发服务器，如此就可以实时查看修改代码变化带来的变化。 在程序的根目录运行以下命令：

`dojo build --mode dev --watch memory --serve`

或者使用缩写参数：

`dojo build -m dev -w memory -s`

现在，打开浏览器并导航到 [http://localhost:9999](http://localhost:9999) 查看当前应用程序。

接下来，我们开始定制应用程序。

{% section %}

## 页面内容

{% task '改变页面中显示的内容。' %}

为了开始定制应用程序，我们先删掉已有的内容。 需要调整两个地方。 第一行，`index.html` 文件中的 “Welcome to biz-e-core”。

{% instruction '打开 `src` 文件夹中的 `index.html` 文件，删除 `<p>` 标签及其内容 “Welcome to biz-e-corp”。' %}

注意，页面已自动刷新。 这意味着我们不需要中断我们的工作，去刷新或重新构建程序，就可以实时查看调整后的效果。

现在我们删掉 “Hello, Dojo World!” 信息。

{% instruction '打开位于 `/src` 下的 `main.ts`。' %}

你将看到以下代码：

```ts
import renderer from '@dojo/framework/widget-core/vdom';
import { v } from '@dojo/framework/widget-core/d';

const r = renderer(() =>
    v('div', [ 'Hello, Dojo World!' ])
);
r.mount({ domNode: document.querySelector('my-app') as HTMLElement });
```

可能现在看看不懂部分代码，但看完后面的教程后，你将逐步了解。 现在我们重点了解这行代码：

```ts
v('div', [ 'Hello, Dojo World!' ])
```

`v` 函数指示 Dojo 创建一个 HTML 元素，这段代码创建一个 `<div>` 元素， 并在其中添加文字 “Hello, Dojo World!”。 我们接下来将构建一个页面，在这个页面中能查看 Biz-E 公司的员工列表， 所以我们将标签和消息修改为更合适的内容。

{% instruction '使用 `<h1>` 标签替换掉 `<div>` 标签，并用 “Biz-E-Bodies” 替换掉 “Hello, Dojo World!”' %}

```ts
const r = renderer(() =>
    v('h1', [ 'Biz-E-Bodies' ])
);
```

现在，我们再回过头来看 `v` 函数。我们有意避免使用 `document.createElement` 来创建 DOM ([Document Object Model](https://en.wikipedia.org/wiki/Document_Object_Model)) 元素。这是因为我们不会直接创建 DOM 元素。相反，我们用 TypeScript 创建一个视图(view)的表现层，然后 Dojo 能高效的将这个视图转换为 DOM 元素，并在页面中渲染出来。这个渲染技术就是所谓的 *virtual* DOM。

传统的 web 程序，将 DOM 和 JavaScript 逻辑揉在一起，导致较复杂的应用程序的复杂度提高且效率低下。当构建具有大量的状态和数据更改的应用程序时， 虚拟 DOM 技术能极大简化程序逻辑并提高性能。虚拟 DOM 扮演中间人角色，处在应用程序逻辑和在页面中渲染真正 DOM 节点之间。

Dojo 使用自研的虚拟 DOM 库，提供最高效的与视图中的 DOM 元素交互方式。虚拟 DOM 的另一个好处是它促使你编写出 reactive 风格的代码，而这种风格的代码会简化你的程序。在本教程的最后部分，我们将学习如何为虚拟 DOM 节点设置属性。

{% section %}

## 虚拟 DOM 属性

{% task '为虚拟 DOM 节点设置属性。' %}

现在我们为早先创建的 `HelloWorld.ts` 文件中的 `<h1>` 元素增加属性。`v` 函数的第二个参数用于传入这些属性。

{% instruction '更新 `v` 函数调用, 传入 `title` 属性。' %}

```ts
v('h1', { title: 'I am a title!' }, [ 'Biz-E-Bodies' ])
```

{% aside '虚拟 Dom 的 Properties 和 Attributes' %}
虚拟 DOM 系统自动将 `string` 类型的 properties 作为 attribute，其余的作为 properties 添加到 DOM 节点上。
{% endaside %}

注意，我们在 tag 和内容参数中间添加了一个参数。第二个参数可以往元素中设置任意 attributes 或 properties。这种使用 JavaScript 或 TypeScript 创建 DOM 元素的方法被称为 [HyperScript](https://github.com/hyperhype/hyperscript)，这种技术应用在很多虚拟 DOM 实现中。

{% section %}

## 总结

恭喜！您在精通 Dojo 之旅中，迈出了坚实的一步。[Dojo 应用程序的组件](../002_creating_an_application/)中，我们开始了解 Dojo 应用程序中的重要组件。

你可以在 [codesandbox.io](https://codesandbox.io/s/github/dojo/dojo.io/tree/master/site/source/tutorials/001_static_content/demo/finished/biz-e-corp) 中打开完整示例或[下载](../assets/001_static_content-finished.zip)项目。

{% section 'last' %}
