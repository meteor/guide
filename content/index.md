---
title: 介绍
order: 0
description: 这是使用 Meteor，一个用于开发现代网页和移动应用的全栈 JavaScript 平台。
---

<!--  XXX: note that this content is somewhat duplicated on the docs, and should be updated in parallel -->
<h2 id="what-is-meteor">什么是 Meteor</h2>

Meteor 是一个用于开发现代网页和移动应用的全栈 JavaScript 平台。Meteor 包含了一些用来构建连接客户端的实时应用的关键技术，一个构建工具，和一套精心维护的、来自 Node.js 和广泛 JavaScript 的社区的包。

- Meteor 使你能使用 **一种语言** 来开发，即 JavaScript，在所有的环境中：应用服务器、网页浏览器和移动设备。

- Meteor 在网络上传输 **数据**，而不是 HTML，并由客户端来渲染。

- Meteor **拥抱整个生态系统**，以一种谨慎的方式将相当活跃的 JavaScript 社区中最棒的部分带给你。

- Meteor 提供 **全栈响应性**，使你无需耗费太大精力就能让你的 UI 无缝地反应世界的实际状态。

<h2 id="quickstart">快速上手</h2>

Meteor 支持 [OS X、Windows 和 Linux](https://www.meteor.com/install).

Windows？  [在这里下载官方 Meteor 安装包](https://install.meteor.com/windows)。

OS X 或 Linux？从你的终端安装最新的官方 Meteor 发行版。

```bash
curl https://install.meteor.com/ | sh
```

> 译注：若上述地址无法使用，可以尝试使用 `curl http://install-cn.ourmeteor.com/ | sh`。我们将保持它更新到最新版。

Windows 安装包支持 Windows 7、Windows 8.1、Windows Server 2008 及 Windows Server 2012。命令行安装器支持 Mac OS X 10.7 (Lion) 或更高版本，及 x86 和 x86_64 架构的 Linux。

当你安装了 Meteor 后，创建一个项目：

```bash
meteor create myapp
```

本地运行它：

```bash
cd myapp
meteor npm install
meteor
# Meteor server running on: http://localhost:3000/
```

> Meteor 自带了一个 npm，所以你可以输入 `meteor npm` 而不需自己安装一个。当然，如果你愿意，可以使用一个全局安装的 npm 来管理包。
> 译注：过高版本的全局 npm 在现在可能引起一定问题。建议使用 `meteor npm`。你也许希望使用一个 npm 镜像，但是 Meteor 自带的尚不支持使用镜像。

<h2 id="learning-more">Meteor 资源</h2>

1. 开始上手 Meteor 的地方是[官方教程](https://www.meteor.com/tutorials/blaze/creating-an-app)。

2. [Stack Overflow](http://stackoverflow.com/questions/tagged/meteor) 是问技术问题最好的地方（当然答也是）。记得加上 meteor 标签。

3. 访问 [Meteor 论坛](https://forums.meteor.com) 来宣布项目、获得帮助、与社区讨论或讨论 Meteor 核心的改动。

4. [Meteor 文档](https://docs.meteor.com)是找到 Meteor API 文档最好的地方。

5. [Atmosphere](https://atmospherejs.com) 是社区为 Meteor 设计的包的专属仓库。

6. [Awesome Meteor](https://github.com/Urigo/awesome-meteor) 是一个社区维护的[包](https://github.com/Urigo/awesome-meteor#getting-started)与[资源](https://github.com/Urigo/awesome-meteor#resources)的列表。

<h2 id="what-is-it">什么是 Meteor 指南？</h2>

这是一组关于Meteor平台最佳实践的概要介绍，我们会覆盖到现代网站和移动应用开发中的共通部分，有许多这里提到的概念并不是Meteor特有的，这些概念可被应用到任何聚焦于现代的，用户界面可交互的程序中去。

在创建Meteor的应用时，这些指南建议**并不是必要的**，你可以完全不遵从指南中的原则和建议。但是，这篇指南主要是试图将那些最佳实践和社区共识文档化，我们希望Meteor社区中的大多数人，能从这些文档化的实践中汲取到有益的东西。

Meteor的具体API可以在[文档页](https://docs.meteor.com/)获取到，你也可以在[atmosphere](https://atmospherejs.com/)浏览到社区贡献的Meteor包。

<h3 id="audience">目标受众</h3>

这篇指南面向那些有JavaScript或者Meteor或者web开发相关经验的中级开发者，如果你刚刚开始接触Meteor，建议你先从[官方教程](https://www.meteor.com/tutorials/blaze/creating-an-app)开始。

<h3 id="example-app">示例应用</h3>

许多文章都使用Todos作为示例应用，这个应用围绕着本指南开发，你可以在它的[github仓库](https://github.com/meteor/todos)里获取到最新的源代码，当然你也可以通过发起请求(pull request)来提交议题(issues)或建议。

<h2 id="guide-concepts">指南发展</h2>

<h3 id="contributing">贡献</h3>

该指南的持续开发被**公开**在[github](https://github.com/meteor/guide)上，我们鼓励发起请求(pull requests)和议题(issues)来讨论和修改这里面有问题的内容。我们希望保持我们在进度的的开放和忠实，这样可以将该指南的计划和Meteor将来在版本上的更新表述的更加清晰。

<h3 id="goals">项目目的</h3>

在该指南中做出的决定或者概括的实践都必须是**武断的**。确定的最佳实践将被高亮，其他合理的建议将不会。我们将致力于将社区的共识放到指南主要的决议中，但是当编写应用程序时总能找到其他方法来解决问题。我们认为在找到其他方式之前知道“标准”的解决问题的方式很重要的。如果证实其他方式更好的解决问题，它应该在下一版本的指南中被采用。

这个还有一个重要的功能的就是充当Meteor开发的风向标。通过文档化这些最佳实践，这些指南将聚焦于说明这个平台将变得更好，更简单，更高的性能，因此我们将更多的聚焦于平台未来的选在。

同样的，除了在指南中被重点提到的解决方式，经常需要社区的包来提供插件化支持；我们希望当你发现可以通过编写一个包来增强Meteor的工作流，那你就去做吧！如果你不确定怎样的更好的设计或组织你的包，到社区里去获取帮助跟讨论。
