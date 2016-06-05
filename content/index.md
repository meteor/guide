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

Windows？  [在这里下载官方 Meteor 安装包。](https://install.meteor.com/windows).

OS X 或 Linux？从你的终端安装最新的官方 Meteor 发行版。

```bash
curl https://install.meteor.com/ | sh
```

> 译注：若上述地址无法使用，可以尝试使用 `curl http://install-cn.ourmeteor.com/1.3.2.4 | sh`。其中 `1.3.2.4` 是版本号，我们将保持它更新到最新版。

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
> 译注：过高版本的全局 npm 在现在可能引起一定问题。建议使用 `meteor npm`。

<h2 id="learning-more">Meteor 资源</h2>

1. 开始上手 Meteor 的地方是[官方教程](https://www.meteor.com/tutorials/blaze/creating-an-app)。

2. [Stack Overflow](http://stackoverflow.com/questions/tagged/meteor) 是问技术问题最好的地方（当然答也是）。记得加上 meteor 标签。

3. 访问 [Meteor 论坛](https://forums.meteor.com) 来宣布项目、获得帮助、与社区讨论或讨论 Meteor 核心的改动。

4. [Meteor 文档](https://docs.meteor.com)是找到 Meteor API 文档最好的地方。

5. [Atmosphere](https://atmospherejs.com)是社区为 Meteor 设计的包的专属仓库。

6. [Awesome Meteor](https://github.com/Urigo/awesome-meteor) 是一个社区维护的[包](https://github.com/Urigo/awesome-meteor#getting-started)与[资源](https://github.com/Urigo/awesome-meteor#resources)的列表。

<h2 id="what-is-it">What is the Meteor Guide?</h2>

This is a set of articles outlining opinions on best-practice application development using the [Meteor](https://meteor.com) platform. Our aim is to cover patterns that are common to the development of all modern web and mobile applications, so many concepts documented here are not necessarily Meteor specific and could be applied to any application built with a focus on modern, interactive user interfaces.

Nothing in the Meteor guide is *required* to build a Meteor application---you can certainly use the platform in ways that contradict the principles and patterns of the guide. However, the guide is an attempt to document best practices and community conventions, so we hope that the majority of the Meteor community will benefit from adopting the practices documented here.

The APIs of the Meteor platform are available at the [docs site](https://docs.meteor.com), and you can browse community packages on [atmosphere](https://atmospherejs.com).

<h3 id="audience">Target audience</h3>

The guide is targeted towards intermediate developers that have some familiarity with JavaScript, the Meteor platform, and web development in general. If you are just getting started with Meteor, we recommend starting with the [official tutorial](https://www.meteor.com/tutorials/blaze/creating-an-app).

<h3 id="example-app">Example app</h3>

Many articles reference the Todos example application. This code is being actively developed alongside the guide. You can see the latest source code for the app, and file issues or make suggestions via pull request at its [GitHub repository](https://github.com/meteor/todos).

<h2 id="guide-concepts">Guide development</h2>

<h3 id="contributing">Contributing</h3>

Ongoing Meteor Guide development takes place **in the open** [on GitHub](https://github.com/meteor/guide). We encourage pull requests and issues to discuss problems with and changes that could be made to the content. We hope that keeping our process open and honest will make it clear what we plan to include in the guide and what changes will be coming in future Meteor versions.

<h3 id="goals">Goals of the project</h3>

The decisions made and practices outlined in the guide must necessarily be **opinionated**. Certain best practices will be highlighted and other valid approaches ignored. We aim to reach community consensus around major decisions but there will always be other ways to solve problems when developing your application. We believe it's important to know what the "standard" way to solve a problem is before branching out to other options. If an alternate approach proves itself superior, then it should make its way into a future version of the guide.

An important function of the guide is to **shape future development** in the Meteor platform. By documenting best practices, the guide shines a spotlight on areas of the platform that could be better, easier, or more performant, and thus will be used to focus a lot of future platform choices.

Similarly, gaps in the platform highlighted by the guide can often be plugged by **community packages**; we hope that if you see an opportunity to improve the Meteor workflow by writing a package, that you take it! If you're not sure how best to design or architect your package, reach out on the forums and start a discussion.
