---
title: Atmosphere vs. npm
order: 27
discourseTopicId: 20193
---

从零开始构建一个应用是个几乎不可能实现的任务。这也是你开始使用 Meteor 其中一个主要原因——你可以专注于编写你应用本身，而不是重新造一大堆轮子，比如用户登录系统和数据同步系统。为了提高工作效率，使用来自 [npm](https://www.npmjs.com) 和 [Atmosphere](https://atmospherejs.com) 的社区包很有必要。这些社区包中的一些在这份指南中被推荐了，你也可以在在线目录中找到更多。

**从 1.3 起，Meteor 完全支持 npm。未来我们将会把所有包都移到 npm 上，现在暂时你可以同时使用两种包。**

<h2 id="when-atmosphere">什么时候用 Atmosphere 包</h2>

Atmosphere 包是为 Meteor 专门编写的包，相比 npm 包，在用于 Meteor 时有一些优势。展开讲，Atmosphere 包可以：

- 依赖于 Meteor 核心包，如 `ddp` 和 `blaze`
- 显式导入非 JavaScript 文件，包括 CSS、Less、Sass、Stylus 和其他静态资源
- 利用 Meteor 的[构建系统](build-tool.html)自动地从其他语言（如 CoffeeScript）转译过来
- 具有明确定义的在服务器和客户端提供不同代码的方式，使其能够在不同环境中拥有不同的行为
- 直接访问 Meteor 的[包命名空间](using-atmosphere-packages.html#package-namespacing)和包的全局导出而不使用 ES2015 的 `import`
- 通过 Meteor 的 [版本限制处理器（constraint resolver）](writing-atmosphere-packages.html#version-constraints)硬性确定包之间的版本依赖关系
- 内置为 Meteor 构建系统打造的[构建插件](build-tool.html#compiles-with-build-plugins)
- 为不同的服务器架构内置预编译的二进制代码，比如 Linux 和 Windows。

如果你的包依赖于一个 Atmosphere 包（在 Meteor 1.3 里也包括 Meteor 核心包），或者需要利用[构建系统](build-tool.html)，当下最好的选择是写一个 Atmosphere 包。

<h2 id="when-npm">什么时候用 npm 包</h2>

npm is a repository of general JavaScript packages. These packages were originally intended solely for the Node.js server-side environment, but as the JavaScript ecosystem matured, solutions arose to enable the use of npm packages in other environments such as the browser. Today, npm is used for all types of JavaScript packages.

If you want to distribute and reuse code that you've written for a Meteor application, then you should consider publishing that code on npm if it's general enough to be consumed by a wider JavaScript audience. It's simple to [use npm packages in Meteor applications](using-npm-packages.html#using-npm), and possible to [use npm packages within Atmosphere packages](writing-atmosphere-packages.html#npm-dependencies), so even if your main audience is Meteor developers, npm might be the best choice.

> Meteor comes with npm bundled so that you can type `meteor npm` without worrying about installing it yourself. If you like, you can also use a globally installed npm to manage your packages.
