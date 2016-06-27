---
title: 使用 npm 包
order: 30
discourseTopicId: 20193
---

<h2 id="npm-searching">查找 npm 包</h2>

在 [npmjs.com](https://www.npmjs.com/) 查找 npm 包是最好的方法。 当然对于具有明显倾向的特殊种类的包，在其他的网站也有提供，例如站如其名的 [react-components.com](http://react-components.com/) 就提供了 React 的相关 npm 组件包。

<h2 id="client-npm">在客户端使用 npm 包</h2>

为了在客户端浏览器的环境中调用这些为服务端设计的包，开发者们设计并提供了很多工具来在客户端模拟 Node 环境，开发者们设计并提供了很多工具，包括 [browserify](http://browserify.org) 和 [webpack](https://webpack.github.io) 等。Meteor 的 ES2015 模块系统就可以让开发这在没有任何额外配置的情况下，在客户端调用现成的 npm 包。大多数情况下，你都可以像在服务端一样从客户端文件中导入 npm 依赖。

> 在创建一个新的应用的时候，Meteor 会安装 `meteor-node-stubs` 包来解决在客户端的浏览器兼容问题。如果你的应用要升级到Meteor 1.3，那你可能必须手动运行 `meteor npm install --save meteor-node-stubs` 命令了。

<h2 id="installing-npm">安装 npm 包</h2>

npm 包配置放在项目根目录的 `package.json` 文件中。如果你创建了一个新的 Meteor 项目，Meteor 会自动在这里为你生成该文件。旧项目升级的话，运行 `meteor npm init` 手动新建一个。

你可以通过在项目根目录运行带 `--save` 选项的 `npm install` 命令来安装某个 npm 包。

```bash
meteor npm install --save moment
```

这个操作将会在 `package.json` 文件中添加相关的 npm 依赖并且下载包到本地的 `node_modules/` 目录中。通常，不用在版本控制系统中检入 `node_modules/` 文件夹，当相关 npm 包有更新的时候，你的团队成员可以通过运行 `meteor npm install` 来保持 npm 处于最新状态。

```bash
meteor npm install
```

如果这只是一个开发阶段的依赖包（例如：它是用来测试，代码格式检查等等），那么你就应该是用`--save-dev`标识。那样的话，如果你用脚本构建，可以通过 `npm install --production` 命令来避免安装不需要的包。

关于 `npm install`，请参考[ Meteor 官方文档](https://docs.npmjs.com/getting-started/installing-npm-packages-locally)

> Meteor 自带了一个 npm，所以你可以输入 `meteor npm` 而不需自己安装一个。当然，如果你愿意，可以使用一个全局安装的 npm 来管理包。
> 译注：过高版本的全局 npm 在现在可能引起一定问题。建议使用 `meteor npm`。你也许希望使用一个 npm 镜像，但是 Meteor 自带的尚不支持使用镜像。

<h2 id="using-npm">使用 npm 包</h2>

通过简单地 `import` 包名可以向应用里面导入 npm 包。

```js
import moment from 'moment';

// 这个就等同于标准的 node require 用法：
const moment = require('moment');
```

以上的脚本展示的是如何从包里面导入一个默认的导出量到 `moment` 变量。

通过[解构赋值](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)（ destructuring assignment ）的语法，你也可以导入包中某些函数。

```js
import { isArray } from 'lodash';
```

你也可以从包中 import 其他文件或者JS 入口点

```js
import { parse } from 'graphql/language';
```

<h3 id="npm-styles">如何从 npm 包导入样式</h3>

利用任何一个 Meteor 支持的 CSS 预处理器，你就可以从 npm 包环境中通过相对和绝对路径导入样式表文件。

使用 `{}` 语法从 npm 包中通过绝对路径引用，以下是 Less 的示例：

```less
@import '{}/node_modules/npm-package-name/button.less';
```

从 npm 包中通过相对路径引用

```less
@import '../../node_modules/npm-package-name/colors.less';
```

如果你已经安装了 `ecmascript` 包，你可以直接在 JavaScript 文件中控制加载顺序并导入 CSS 文件。

```js
import 'npm-package-name/stylesheets/styles.css';
```

> 当你从 JavaScript 文件中导入 CSS 的时候，这个 CSS 样式是没有和其他通过 Meteor Build tool 处理过的 CSS 放在一起的，而是添加在你的应用下面的 `<head>` 标记下，`<style>...</style>` 里面，紧跟在这里面其他 CSS 后面的。

<h2 id="npm-shrinkwrap">npm Shrinkwrap</h2>

在 `package.json` 中，npm 包的版本号通常是一个版本区间，所以，如果这个时候新的版本也刚发布的话，每次调用 `npm install` 命令的时候有时候可能会安装不一样的版本。为了保证你的团队成员都是使用同一个包的同一版本号，在对 `package.json` 作出版本更新之后，最好运行一下 `npm shrinkwrap` 命令。

```bash
# 安装之后
meteor npm install --save moment
meteor npm shrinkwrap
```

这将会生成一个 `npm-shrinkwrap.json` 文件，里面确定了每个包依赖的特定版本号，如果你想更加精细的控制（特定版本的包也有可能不一样），你需要把这个文件也检入你的版本控制系统里面去。同时为了避免在部署过程中对 npm 服务器的依赖，你应该考虑使用 [`npm shrinkpack`](#npm-shrinkpack) 。

<h2 id="async-callbacks">异步回调</h2>

很多 npm 包代码风格是异步的，带回调参数的或者基于 Promise 的。由于一些原因，Meteor 当前是构建在同步框架内的，但使用 [Fibers](https://github.com/laverdet/node-fibers) 仍然可以编写非阻塞的代码。

Meteor 服务器的全局环境上下文，每一个方法和发布都会初始化一个新的 fiber，如此我们才可以并行处理这么多脚本。很多 Meteor 的 API ，例如：数据集的内部都是依赖在一个 fiber 下的。它们同时也依赖于内部的一套跟踪服务器 「environment」 状态的机制。这意味着，如果你想在 Meteor app 内部运行异步的 Node 代码的话，你需要初始化自己的 fiber 和 environment。让我们来看一个无法运行的例子中的一小段代码。来自于 [node-github 仓库](https://github.com/mikedeboer/node-github) 的代码示例：

```js
// Meteor 方法体内部
updateGitHubFollowers() {
  github.user.getFollowingFromUser({
    user: 'stubailo'
  }, (err, res) => {
    // 在这里使用数据集将会抛出异常 
    // 因为异步代码段没有放在在 fiber 环境里面
    Followers.insert(res);
  });
}
```
再来看看几种解决方案。

<h3 id="bind-environment">`Meteor.bindEnvironment`</h3>

大多数情况下，简单的使用 `Meteor.bindEnvironment` 封装一个回调就可以做到。这个函数同时也会包含在一个 fiber 里面，并且做了需要维护 Meteor 服务端 environment 监测的事情。下面是使用 `Meteor.bindEnvironment` 的示例：

```js
// Meteor 方法体内部
updateGitHubFollowers() {
  github.user.getFollowingFromUser({
    user: 'stubailo'
  }, Meteor.bindEnvironment((err, res) => {
    // 一切运行正常
    Followers.insert(res);
  }));
}
```
然而，并不是所有情况的可以运行。如果你想通过它，来调用带返回值 API 方法并提取返回值，那是不可能做到的，因为这段代码是异步运行的。这个时候，你就需要一些别的方法，来把异步的API转换成一个带返回值的同步方法。

<h3 id="wrap-async">`Meteor.wrapAsync`</h3>

许多 npm 包方法都沿袭了能接受 `(err, res)` 回调参数的惯例。如果你的异步 API 恰好满足以上描述，就如同上面示例，你可以使用 `Meteor.wrapAsync` 把它转换成 fiberized fiber 化的 API ，它会返回数据和异常，而不是传入回调，方法如下：

```js
// 设置 sync API
const getFollowingFromUserFiber =
  Meteor.wrapAsync(github.user.getFollowingFromUser, github.user);

// Meteor 方法体内部
updateGitHubFollowers() {
  const res = getFollowingFromUserFiber({
    user: 'stubailo'
  });

  Followers.insert(res);

  // 返回有多少个关注者（followers）
  return res.length;
}
```

如果你想重构并且创建一个完全 fiber-wrapper 的 GitHub 客户端，你需要写脚本找出所有的可用的方法并调用 `Meteor.wrapAsync` 处理它，这样才可以做出一个调用方式类似的更好地兼容 Meteor 的API。

<h3 id="promises">Promises</h3>

最近，许多的 npm 包都抛弃了回调的使用方式，转而采用 Promises 的写法。这意味着你其实可以从一个异步方法中拿到返回值，但它只是一个没有值的空壳，之后也会填入真实的值。

有一个好消息，最新的 ES2015 `async/await` 语法（ Meteor 1.3 `ecmascript` 包中包含了这个用法）将会在 Promises 得到应用。如果真这样，那么就可以在客户端和服务端使用统一的，自然的和具有同步化风格的写法。

如果声明一个带 `async` 关键字的函数（意味着该函数会返回 Promise 本身），然后在其他 Promise 里面，使用 `await` 关键字来等待。这样序列化的调用其他基于 Promise 的库就会比较简单了。


```js
async function sendTextMessage(user) {
  const toNumber = await phoneLookup.findFromEmail(user.emails[0].address);
  return await client.sendMessage({
    to: toNumber,
    from: '+14506667788',
    body: 'Hello world!'
  });
}
```

<h2 id="npm-shrinkpack">Shrinkpack</h2>

相对于单独使用 [`npm shrinkwrap`](#npm-shrinkwrap) 而言，[Shrinkpack](https://github.com/JamieMason/shrinkpack) 是一个更加安全的，可重复的打包包依赖的工具。

本质上，它会复制每个 npm 包依赖的内容，生成对应的 tarball 压缩包文件到你应用的源码库里面。它其实就是一个 shrinkwrap 工具，只是生成一个更加健壮的 `npm-shrinkwrap.json` 版本库文件而已。这样你的应用就可以在不依赖 npm 服务器的情况下进行编译了。这样的好处就是可以在部署的时候产生重复的特定的 build 。

首先你需要全局地安装 shrinkpack 才可以使用它。

```bash
npm install -g shrinkpack
```

在把包 shrinkwrap 之后，就可以直接使用了。

```bash
meteor npm install moment
meteor npm shrinkwrap
shrinkpack
```

同时你应该检入已生成的 `node_shrinkwrap/` 到版本控制系统中，确保它不会被你的文本编辑器忽略掉。

**备注**: 在项目中使用 npm 包是一个好的想法，它不会影响其他 Atmosphere 包依赖，甚至很多 Atmosphere 包都直接引入 npm 包依赖。
