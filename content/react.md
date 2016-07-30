---
title: React
order: 23
description: 如何在 Meteor 中使用 React（Facebook 的前端渲染库）
discourseTopicId: 20192
---

读完这篇文章，你将学到：

1. 什么是 React，为什么你会考虑在 Meteor 中使用 React。
2. 如何在你的 Meteor 应用中安装 React 并正确地使用它。
3. 如何将 React 与 Meteor 的实时数据层进行整合。
4. 如何在一个 React-Meteor 应用中进行路由（页面跳转）。

<h2 id="introduction">介绍</h2>

[React](https://facebook.github.io/react/) 是由 Facebook 团队开发并维护，用于构建响应式用户界面的 JavaScript 库。React 是 Meteor 所支持的三种渲染库之一，其它两者分别是 [Blaze](blaze.html) 和 [Angular](angular.html)。若有兴趣，你可以在这里查看他们[三者的比较](ui-ux.html#view-layers)。

React 有着一个活跃并且仍在不断壮大的生态系统，并且，搭配着各种不同的框架，React 已经被广泛地应用于生产实践中。

想了解更多关于 React 的知识并且快速上手，请参考 [React documentation](https://facebook.github.io/react/docs/getting-started.html)。[thinking in React](https://facebook.github.io/react/docs/thinking-in-react.html) 这篇文章更是很好地阐明了 React 背后的哲学思想。

想要在 Meteor 中上快速上手使用 React，你可以参考 [React tutorial](https://www.meteor.com/tutorials/react/creating-an-app)。若想要进一步了解如何使用 React 开发更加完整的 Meteor 应用，请参考  [Todos 应用示例](https://github.com/meteor/todos/tree/react)。这篇文章会在一些地方引用这个示例的部分代码。

<h3 id="using-with-meteor">React 的安装与使用</h3>

在 Meteor 1.3 中安装 React，只需将它添加到 npm 依赖：

```sh
npm install --save react react-dom
```

这将会在你的项目中安装 `react` 并允许你在项目文件中通过 `import React from 'react'` 来使用它。绝大部分 React 代码是用 [JSX](https://facebook.github.io/react/docs/jsx-in-depth.html) 格式来编写的，不过你不用担心编译问题，如果你在项目中添加了 `ecmascript` 这个 Atmosphere 包，那么你可以[在 Meteor 项目中直接编写并使用JSX文件](http://guide.meteor.com/build-tool.html#react-jsx)，而这个包，则是所有 Meteor 应用默认安装的。

```jsx
import React from 'react';

export default class HelloWorld extends React.Component {
  render() {
    return (
      <h1>Hello World</h1>
    );
  }
}
```

你可以使用 `react-dom` 包在 DOM 中渲染一个组件：

```jsx
import { Meteor } from 'meteor/meteor';
import React from 'react';
import { render } from 'react-dom';
import HelloWorld from './HelloWorld.js';

Meteor.startup(() => {
  render(<HelloWorld />, document.getElementById('app'));
});
```

当然，为了使上述代码生效，你需要在你的 HTML 文件的 body 中的某一位置添加 `<div id="app"></div>`。

每一个新创建的 Meteor 应用都自带 Meteor 的默认模板系统 - Blaze。如果你不打算在项目中[同时使用 React 和 Blaze](#using-with-blaze)，你可以通过以下命令从你的项目中删除 Blaze：

```sh
meteor remove blaze-html-templates
meteor add static-html
```

<h3 id="using-third-party-npm-packages">使用第三方 npm 包</h3>

如果你想使用在 [npm 上发布的第三方 React 组件](https://www.npmjs.com/search?q=react)，你可以通过 `npm install --save` 来安装它们，并在应用中通过 `import` 来导入他们。

例如，为了使用超棒的 React 包 [Griddle](http://griddlegriddle.github.io/Griddle/) 来制作表格，你可以运行如下命令：

```sh
npm install --save griddle-react
```

然后，你可以像使用其他 [npm 包](using-packages.html#npm)一样，在应用中导入组件：

```jsx
import React from 'react';
import Griddle from 'griddle-react';

export default class MyGriddler extends React.Component {
  render() {
    return (<Griddle ..../>);
  }
}
```

如果你想编写一个 Atmosphere 包来封装一个组件，你需要做[更多工作](#atmosphere-packages)。

<span id="using-with-blaze"><!-- don't break old links --></span>
<h3 id="react-in-blaze">在 Blaze 中使用 React </h3>

有时候，你可能会需要在 [Blaze](#blaze.html) 中使用 React，例如，将一个使用 Blaze 构建的大型应用，逐步过渡到 React 上来。这时，你可以使用 [`react-template-helper`](https://atmospherejs.com/meteor/react-template-helper) 组件，它将帮助你在一个 Blaze 模板中渲染一个 React 组件。首先，运行 `meteor add react-template-helper`，然后，在你的模板中使用 `React` helper：

```html
<template name="userDisplay">
  <div>Hello, {{username}}</div>
  <div>{{> React component=UserAvatar userId=_id}}</div>
</template>
```

你需要通过一个 helper 来传递一个 React 组件类：

```js
import { Template } from 'meteor/templating';

import './userDisplay.html';
import UserAvatar from './UserAvatar.js';

Template.userDisplay.helpers({
  UserAvatar() {
    return UserAvatar;
  }
})
```

`component` 这个参数的值正是需要被渲染的 React 组件，它应该通过一个 helper 进行传入。

其他所有的参数都将各自作为一个 React 组件的 prop 传入这个组件。

请注意：

- React 组件必须是这个包裹元素的唯一内容。 由于 React 的一些限制（参考 facebook/react [#1970](https://github.com/facebook/react/issues/1970), [#2484](https://github.com/facebook/react/issues/2484)），一个 React 组件必须是渲染它的父节点的唯一子节点，也就是说，它不能有其他兄弟节点。

- 这也意味着，一个 React 组件不能是一个 Blaze 模板的唯一内容，因为我们无法预知这个模板将会在哪里被使用。

<h4 id="passing-callbacks-from-blaze">向一个 React 组件传递回调函数</h4>

如果你使用这个 helper 引入了一个 React 组件，为了向它传递一个回调函数，你只需要创建一个[返回一个函数的模板 helper](http://guide.meteor.com/blaze.html#pass-callbacks)，并将它作为一个 prop 传递给 React 组件，例如：

```js
Template.userDisplay.helpers({
  onClick() {
    const instance = Template.instance();

    // 从这个 helper 中返回一个函数
    // 注意，instance 变量处在一个闭包中
    return () => {
      instance.hasBeenClicked.set(true)
    }
  }
});
```

在 Blaze 中使用它：

```html
<template name="userDisplay">
  <div>
    {{> React component=UserAvatar userId=_id onClick=onClick}}
  </div>
</template>
```

<h3 id="blaze-in-react">在 React 中使用 Blaze</h3>

正如我们可以在 Blaze 模板中使用 React 组件一样，我们也可以在 React 组件中使用 Blaze 模板。类似地，这种特性对于从 React 逐步过渡到 Blaze 的策略非常有用，然而更棒的是，它使得我们能够在 React 项目中继续使用当初为 Blaze 构建的海量的 Atmosphere 包，就连核心包 `accounts-ui` 也不例外。

一个简单的方式是使用 [`gadicc:blaze-react-component`](https://atmospherejs.com/gadicc/blaze-react-component) 包。首先，运行 `meteor add gadicc:blaze-react-component`，然后，按如下方式在你的组件中导入并使用它：

```jsx
import React from 'react';
import Blaze from 'meteor/gadicc:blaze-react-component';

const App = () => (
  <div>
    <Blaze template="itemsList" items={items} />
  </div>
);
```

`<Blaze template="itemsList" items={items} />` 这一行同你在 Blaze 模板中写的 `{% raw %}{{> itemsList items=items}}{% endraw %}` 是一个意思。如需更多信息，请参考这个包的[项目页](https://github.com/gadicc/meteor-blaze-react-component)。

<h2 id="data">使用 Meteor 的数据系统</h2>

React 是一个前端渲染库，因此，它并不关心数据是如何流入及流出组件的。另一方面，Meteor 对于数据的处理方式非常执着。Meteor 应用通过 [publications](data-loading.html) 和 [methods](methods.html) 来实际运作，而它们分别被用于订阅和修改应用数据。

为了集成这两大系统，我们开发了 [`react-meteor-data`](https://atmospherejs.com/meteor/react-meteor-data) 包，利用 Meteor 的响应式系统 [Tracker](https://www.meteor.com/tracker)，使得 React 组件能够响应数据的改变。

<h3 id="using-createContainer">使用 `createContainer`</h3>

一旦你运行了 `meteor add react-meteor-data`，你就可以导入 `createContainer` 函数来创建 [container 组件](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0#.by86emv9b)，并用它向表现型组件 (presentational component) 提供其所需的数据。

> 译者注：presentational 有表现、展现的意思，表现型组件 (presentational component) 即只负责表现样式的组件，它们不关心需要渲染的数据从哪里来。而相对应地，container 组件则只关心如何获取数据并传递给表现型组件。

> 提一下，「container 组件」 和 「表现型组件」 分别类似于我们在一种设计模式中提到的 「智能组件」 和 「可复用组件」，如果你想了解更多关于这种思想和 Meteor 之间的关系，请参考这篇文章 [UI/UX article](http://guide.meteor.com/ui-ux.html#components)。

例如，在 Todos 示例应用中，我们有一个 `ListPage` 组件，它负责渲染一个 list 的基本信息以及这个 list 中的所有 tasks。为了做到这些，它需要 [订阅](data-loading.html#subscriptions) `todos.inList` 这个publication，检查这个subscription是否就绪，然后从 `Todos` 数据集中查询数据。

它还需要实时响应相关操作所带来的数据变化（例如，因为另一个用户的某种操作而导致一个 todo 发生改变）。所有这些数据加载的复杂性都是要求 container-presentational 组件分离的典型用例，而 `createContainer()` 函数则使得这种分离变得非常容易。

我们先简单地将 `ListPage` 组件定义为一个 presentational 组件，它期望需要的数据以 React `props` 的形式传入：

```jsx
import React from 'react';

export default class ListPage extends React.Component {
  ...
}

ListPage.propTypes = {
  list: React.PropTypes.object,
  todos: React.PropTypes.array,
  loading: React.PropTypes.bool,
  listExists: React.PropTypes.bool,
};
```

然后，我们创建一个名为 `ListContainer` 的 container 组件来包裹 `ListPage` 并向它提供数据：

```js
import { Meteor } from 'meteor/meteor';
import { Lists } from '../../api/lists/lists.js';
import { createContainer } from 'meteor/react-meteor-data';
import ListPage from '../pages/ListPage.js';

export default ListPageContainer = createContainer(({ params }) => {
  const { id } = params;
  const todosHandle = Meteor.subscribe('todos.inList', id);
  const loading = !todosHandle.ready();
  const list = Lists.findOne(id);
  const listExists = !loading && !!list;
  return {
    loading,
    list,
    listExists,
    todos: listExists ? list.todos().fetch() : [],
  };
}, ListPage);
```

最好给 container 和它包裹的组件起一样的名字，并在最后添加「Container」这个单词。这样一来，当你想在代码中追踪问题时，你将很容易定位到可能的文件或类。

一旦在函数中访问过的[响应式数据源](https://atmospherejs.com/meteor/tracker)发生任何变化，这个由 `createContainer()` 创建的 container 组件就会相应地重新渲染它所包裹的组件。

虽然，这个 `ListContainer` 的设计使用场景是将其放在 React Router 之中并根据路由自动传入参数，不过，如果你想手动实例化这个 container 组件，你只需要将参数作为 props 传入即可：

```jsx
<ListPageContainer params={{id: '7'}}/>
```

<h3 id="preventing-rerenders">防止重新渲染</h3>

有时候，一些数据变化会导致数据的重新运算，但是你很清楚这些变化不会影响到你的UI。虽然通常来说，React 在防止无用的重新渲染方面做得足够有效，但如果你确实需要进行更精确的控制，上述模式让你可以很容易地在表现型组件上使用 React 的 [`shouldComponentUpdate`](https://facebook.github.io/react/docs/component-specs.html#updating-shouldcomponentupdate) 函数来避免重新渲染。

<h2 id="routing">路由</h2>

在 Meteor-React 应用的路由方面，这里有两个主流选项。无论选择哪一个，我们都建议你在开发应用前，先查阅我们的 [「路由」章节](routing.html) 来了解一些关于 Meteor 路由的通用原则。

- [`kadira:flow-router`](https://atmospherejs.com/kadira/flow-router) 是一个面向 Meteor 的路由器。它可以支持 React 和 Blaze。更多信息可参考 [「路由」章节](routing.html)。

- [`react-router`](https://www.npmjs.com/package/react-router) 是一个面向 React 的路由器，它在 React 社区中非常流行，你也能很方便地在 Meteor 应用中使用它。

<h3 id="using-flow-router">Flow Router</h3>

Flow Router + React 的组合使用起来与 Flow Router + Blaze 的组合非常类似。唯一的区别是，在你的路由动作中，你需要使用 [`react-mounter`](https://www.npmjs.com/package/react-mounter) 包来挂载组件和布局。当你 `npm install --save react-mounter` 之后，便可以：

```js
import React from 'react';
import { FlowRouter } from 'meteor/kadira:flow-router';
import { mount } from 'react-mounter';

import AppContainer from '../../ui/containers/AppContainer.js';
import ListPageContainer from '../../ui/containers/ListPageContainer.js';


FlowRouter.route('/lists/:_id', {
  name: 'Lists.show',
  action() {
    mount(AppContainer, {
      main: <ListPageContainer/>,
    });
  },
});
```

注意，`react-mounter` 会将布局组件挂载到 `#react-root` 这个DOM节点下。你可以通过 `withOptions()` 函数来更改这个设定。

在下面这个例子中，你的 `App` 组件会通过 `main` prop 收到一个实例化后的 React 组件并渲染它：

```js
const App = (props) => (
  <div>
    <section id="menu"><..></section>
    {props.main}
  </div>
);

export default AppContainer = createContainer(props => {
  // 这里的 props 中会有一项叫做 `main`，它是由 router 传入的
  // 我们返回的所有东西都会被*添加*到 props 中
  return {
    user: Meteor.user(),
  };
}, App);
```

<h3 id="using-react-router">React Router</h3>

使用 React Router 也是非常直观的。一旦你 `npm install --save react-router` 之后，你就可以像在其他 React Router 驱动 React 应用中一样，简单地导出一系列嵌套路由：

```js
import React from 'react';
import { Router, Route, browserHistory } from 'react-router';

// route components
import AppContainer from '../../ui/containers/AppContainer.js';
import ListPageContainer from '../../ui/containers/ListPageContainer.js';
import AuthPageSignIn from '../../ui/pages/AuthPageSignIn.js';
import AuthPageJoin from '../../ui/pages/AuthPageJoin.js';
import NotFoundPage from '../../ui/pages/NotFoundPage.js';

export const renderRoutes = () => (
  <Router history={browserHistory}>
    <Route path="/" component={AppContainer}>
      <Route path="lists/:id" component={ListPageContainer}/>
      <Route path="signin" component={AuthPageSignIn}/>
      <Route path="join" component={AuthPageJoin}/>
      <Route path="*" component={NotFoundPage}/>
    </Route>
  </Router>
);
```

使用 React Router，你需要在 startup 函数中显式地渲染导出的路由。

```js
import { Meteor } from 'meteor/meteor';
import { render } from 'react-dom';
import { renderRoutes } from '../imports/startup/client/routes.js';

Meteor.startup(() => {
  render(renderRoutes(), document.getElementById('app'));
});
```

当你在 Meteor 应用中使用 React Router 时，你可以像使用 Flow Router 一样大致遵守[简单的规则](routing.html)即可。但是，你也应当适当参考 React Router [文档](https://github.com/reactjs/react-router/blob/latest/docs/Introduction.md)所强调的风格、约定与惯例。

其中有一些应当注意的不同点，例如：
 - React Router 鼓励你在路由定义中耦合你的URL设计与布局层次。而 Flow Router 则更加灵活，虽然这样可能会导致更多的样板代码。
 - React Router 拥抱 React 独有的特性，例如将 router 实例通过 React [context](https://facebook.github.io/react/docs/context.html) 进行传递。虽然，Flow Router 并没有默认这样做，但你也同样可以显式地将你的 Flow Router 实例作为 context 进行传递（事实上，这很可能正是最佳实践）。

<h2 id="meteor-and-react">Meteor 和 React</h2>

<h3 id="atmosphere-packages">在 Atmosphere 包中使用 React</h3>

如果你在编写一个 Atmosphere 包并且需要依赖 React 或一个依赖 React 的 npm 包，你不能使用 `Npm.depends()` 和 `Npm.require()`，因为这样会导致你的应用安装 *2* 个 React 副本（除此之外，`Npm.require()` 只能在服务端使用）。

正确的做法是，你应当要求你的用户在应用中自行安装正确的 npm 包。这就能保证客户端中仅有一个 React 副本，因此也就不存在版本冲突问题。

为了检查用户是否已经安装正确版本的 npm 包，你可以使用 [`tmeasday:check-npm-versions`](https://atmospherejs.com/tmeasday/check-npm-versions) 包，以在运行时检查所有依赖的版本。
