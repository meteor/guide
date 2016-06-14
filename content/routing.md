---
title: "URL 和路由"
order: 20
description: 如何使用 FlowRouter 通过 URL 来驱动你的 Meteor 应用的 UI。
discourseTopicId: 19663
---

读完本章，你将学到：

1. URL 在客户端渲染应用中扮演着怎样的角色，以及与在传统的服务端渲染应用中相比，到底有何不同。
2. 如何使用 Flow Router 在应用的客户端和服务端定义路由。 
3. 如何使你的应用根据 URL 的变化来展示不同的内容。
4. 如何构建指向某个路由的链接，以及如何通过程序控制来跳转到某个路由。

<h2 id="client-side">客户端路由</h2>

在网页应用中，_路由_ 是指通过 URL 来驱动用户界面（UI）的过程。URL 是每个浏览器都具有的重要特性，并且在用户看来，它具有几个重要的作用：

1. **书签** - 用户可以将 URL 收藏在浏览器中以保存那些他们需要回顾的内容。
2. **分享** - 用户可以通过发送某个页面链接的方式来和他人分享内容。
3. **导航** - URL 被用于驱动浏览器的前进和后退功能

在传统的网页应用架构中，服务器一次性渲染一整个 HTML 页面，这时，URL 是用户访问这个应用的基本入口。每当用户在应用中点击一个 URL 链接，浏览器就会向服务器发送一个 HTTP 请求，而服务器则会通过服务端路由器做出正确的响应（向用户返回请求的页面），于是，用户便可以通过这种方式在应用中到处跳转。

相较之下，Meteor 则是以_在连接上传递数据_的方式运作。在这种方式之下，服务器并不关心 URL 和 HTML 页面。客户端应用通过 DDP 协议与服务器进行沟通。举一个典型的例子，在应用加载时，它会初始化一系列 _subscriptions_，并通过它们获取渲染应用所需要的数据。当用户和应用进行交互时，可能会加载不同的 subscriptions，但从技术上来说，这个过程并不需要 URL 的参与 - 构建一个 URL 完全不会改变的 Meteor 应用并非难事。

然而，即使是面对 Meteor 应用，用户依然十分关注上述那些 URL 特色功能。由于服务器不再由 URL 驱动，URL 成为了用户的客户端浏览状态的重要表现形式。与传统的服务端渲染应用不同的是，URL 不再需要描述完整的当前用户状态；它只需要包含那部分你希望可以链接过去的状态即可。例如，URL 可以包含页面的搜索筛选关键字，但是却没必要包含一个下拉菜单或弹窗的状态。

<h2 id="flow-router">使用 Flow Router</h2>

安装 [`kadira:flow-router`](https://atmospherejs.com/kadira/flow-router) 包，即可向应用增加路由功能：

```
meteor add kadira:flow-router
```

Flow Router 是由社区提供的 Meteor 路由包。编写这篇文章时，它的最新版本是2.X。想了解更多关于 Flow Router 的特性和功能，请参考  [Kadira Meteor routing guide](https://kadira.io/academy/meteor-routing-guide)。

<h2 id="defining-routes">定义一个简单的路由</h2>

路由器的基本作用是匹配特定的 URL 并执行相应的动作，这些都发生在客户端，包括用户的浏览器内或移动应用容器中。以 Todos 示例应用中的一段代码为例：

```js
FlowRouter.route('/lists/:_id', {
  name: 'Lists.show',
  action(params, queryParams) {
    console.log("Looking at a list?");
  }
});
```

这个路由动作会在两种情况下执行：页面最初就从与预设模式匹配的 URL 进行加载，或在页面打开时，URL 发生改变并与预设模式相匹配。需要注意的是，URL 可以在不向服务器发送任何其他请求的情况下发生改变，这与服务端渲染应用不同。

当路由匹配时，`action` 方法会被执行，此时，你可以执行任何你需要的动作。路由的 `name` 属性是可选的，但它可以让我们在以后更容易地使用这个路由。

<h3 id="url-pattern-matching">URL 模式匹配</h3>

思考一下这段上述示例中使用到的 URL 模式：

```js
'/lists/:_id'
```

这个模式会匹配特定的 URL。你可能已经注意到，这个 URL 中有一部分以 `:` 作为开头 - 这表示这个部分是一个 *url 参数*，并且会匹配一个出现在路径的相同部分的任意字符串。Flow Router 会将 URL 中的这个部分保存到当前路由的 `params` 属性中。

除此之外，URL 可以包含一个 HTTP [*查询字符串*](https://en.wikipedia.org/wiki/Query_string) （跟在可选的 `?` 之后的部分）。如果是这样，Flow Router 会将它们拆分到一个叫做 `queryParams` 的命名参数中。

这里展示了一些示例 URL 以及与它们对应的 `params` 与 `queryParams`：

| URL           | 是否匹配？ | params          | queryParams
| ---- | ---- | ---- | ---- |
| /             | no | | |
| /about        | no | | |
| /lists/        | no | | |
| /lists/eMtGij5AFESbTKfkT | yes | { _id: "eMtGij5AFESbTKfkT"} |  { }
| /lists/1 | yes | { _id: "1"} | { }
| /lists/1?todoSort=top | yes | { _id: "1"} | { todoSort: "top" }

请注意，`params` 和 `queryParams` 中的所有值都是字符串，因为 URL 无法编码数据类型。例如，如果你想用一个参数来表示一个数字，你可能需要在使用前通过 `parseInt(value, 10)` 将它转换为数字。

<h2 id="accessing-route-info">获取路由信息</h2>

Flow Router 可以将 URL 中的参数作为函数参数传递给路由中的 `action` 函数，除此之外，它还在全局单例 `FlowRouter` 中以函数（或响应式函数）的形式提供了各种有用的信息。当用户在应用中到处跳转时，这些函数的返回值也会随之改变（或在一些情况下响应式地改变）。

如同应用中的其他全局单例一样（参考[「数据加载」章节](data-loading.html#stores)以了解 stores），你最好限制对 `FlowRouter` 的使用。这样，你的应用的各部分才能保持模块化与独立。对于 `FlowRouter` 来说，你最好只在应用的组件体系的最顶层使用它，包括「页面」组件或包裹它的布局组件。你可以参考  [UI article](ui-ux.html#components) 以了解更多关于获取数据的知识。

<h3 id="current-route">当前路由</h3>

在你的代码中，当前路由的相关信息常常十分有用。这里列举了一些你可以使用的响应式函数：

* `FlowRouter.getRouteName()` 获取路由名字
* `FlowRouter.getParam(paramName)` 返回一个 URL 参数的值
* `FlowRouter.getQueryParam(paramName)` 返回一个 URL 查询字符串的值

我们曾以 Todos 应用中的列表页作为示例，其中，我们通过 `FlowRouter.getParam('_id')` 来获取当前列表的 id （在后面，我们将看到更多信息）。

<h3 id="active-route">突显激活的路由</h3>

当你希望在组件体系里嵌套较深的组件中通过导航组件来渲染链接时，你可能的确需要使用到全局的 `FlowRouter` 单例来获取当前路由的相关信息。通常，「激活的」路由需要被以某种方式突显出来（以表示网站的这个路由或区域是用户当前正在浏览的地方）。

[`zimme:active-route`](https://atmospherejs.com/zimme/active-route) 包能帮助你方便地做到这件事：

```bash
meteor add zimme:active-route
```

在 Todos 示例应用中，我们在 `App_body` 模板中为用户知道的每一个列表设置一个链接：

```html
{{#each list in lists}}
  <a class="list-todo {{activeListClass list}}">
    ...

    {{list.name}}
  </a>
{{/each}}
```

对于每一个列表，我们可以通过 `activeListClass` helper 来确定用户是否正在浏览它。

```js
Template.App_body.helpers({
  activeListClass(list) {
    const active = ActiveRoute.name('Lists.show')
      && FlowRouter.getParam('_id') === list._id;

    return active && 'active';
  }
});
```

<h2 id="rendering-routes">根据路由进行渲染</h2>

现在，我们知道了如何定义路由以及获取当前路由的相关信息。当用户访问一个路由时，我们已经准备好了去做该做的事——在屏幕上渲染用户界面。

*在这一节中，我们会讨论如何使用 Blaze 作为 UI 引擎来渲染路由。如果你使用 React 或 Angular 来构建应用，你将学到相似的概念，但实现代码差别很大。*

使用 Flow Router 时，在页面上根据不同 URL 展示不同视图的最简单方法，是使用作为补充的 Blaze Layout 包。首先，你需要安装 Blaze Layout 包：

```bash
meteor add kadira:blaze-layout
```

为了使用这个包，我们需要定义一个「布局」组件。在 Todos 示例应用中，这个组件叫做 `App_body`：

```html
<template name="App_body">
  ...
  {{> Template.dynamic template=main}}
  ...
</template>
```

（这里展示的不是完整的 `App_body` 组件，我们仅突出了其中最重要的部分）。
这里，我们使用了一个被称作 `Template.dynamic` 的 Blaze 功能来渲染数据环境中的 `main` 属性所指的模板。使用 Blaze Layout，我们可以在一个路由被访问时改变 `main` 属性。

我们在 `Lists.show` 路由定义的 `action` 函数中做这些事：

```js
FlowRouter.route('/lists/:_id', {
  name: 'Lists.show',
  action() {
    BlazeLayout.render('App_body', {main: 'Lists_show_page'});
  }
});
```

这段代码表示，当用户访问一个形如 `/lists/x` 的 URL 时，`Lists.show` 路由将会使用 `BlazeLayout`
来设置 `App_body` 组件的 `main` 属性。

<h2 id="page-templates">组件即页面</h2>

请注意，我们将被渲染的组件称作 `Lists_show_page` （而不是 `Lists_show`）。这表示，这个模板会直接被一个 Flow Router 动作渲染，并且会成为这个 URL 渲染体系中的「最顶层」。

`Lists_show_page` 模板*无需*参数即可渲染——模板本身需要负责从当前路由获取信息，并将其传递给自己的子模板。
由于 `Lists_show_page` 模板同渲染它的路由绑定得十分紧密，它必然是一个智能组件。如需了解更多关于智能组件和可复用组件，请参考[「UI/UX」章节](ui-ux.html) 。

像 `Lists_show_page` 这样的智能「页面」组件通常需要负责：

1. 收集路由信息，
2. 订阅相关 subscriptions，
3. 从 subscriptions 获取数据，并且
4. 将数据传递给子组件。

在这种情况下，`Lists_show_page` 的大部分逻辑都写在了 JavaScript 代码中，而它的 HTML 模板则显得十分简洁：

```html
<template name="Lists_show_page">
  {{#each listId in listIdArray}}
    {{> Lists_show (listArgs listId)}}
  {{else}}
    {{> App_notFound}}
  {{/each}}
</template>
```

（`{% raw %}{{#each listId in listIdArray}}{% endraw %}}` 是一种动画技巧，请参考[「页面过渡」章节](ui-ux.html#animating-page-changes)）

```js
Template.Lists_show_page.helpers({
  // 我们在只有一项的数组中使用 #each，因此，当lists发生改变时，「list」模板会被删除，
  // 而一个新的副本将被添加，这对于我们需要的动画来说非常重要。
  listIdArray() {
    const instance = Template.instance();
    const listId = instance.getListId();
    return Lists.findOne(listId) ? [listId] : [];
  },
  listArgs(listId) {
    const instance = Template.instance();
    return {
      todosReady: instance.subscriptionsReady(),
      // 为了对数据响应进行控制，我们将「list」（它包含了整个 list 以及所有的字段）作为一个函数
      // 进行传递。当你勾选一个 todo 项时，`list.incompleteCount` 将会改变。如果我们不这样做，
      // 无论你勾选了哪一项，整个 list 都会重新渲染。通过将 list 的数据响应隔离在关注它的区域，
      // 我们可以阻止这样的事发生。
      list() {
        return Lists.findOne(listId);
      },
      // 由于在查询 list 时只设置了 `_id` 字段，我们没有建立对 `list.incompleteCount` 的依赖，
      // 因此避免了在它发生改变时重新渲染整个 todos
      todos: Lists.findOne(listId, {fields: {_id: true}}).todos()
    };
  }
});
```

`listShow` 组件（一个可复用组件）承担了渲染页面内容的实际工作。由于页面组件会将数据作为参数传递给可复用组件，它便可以相当无脑地进行工作，并且，同路由器进行沟通以及渲染页面的这两个关注点也得以分离。

<h3 id="route-rendering-logic">在用户退出登录时更新页面</h3>

这里有一些渲染逻辑不仅和路由相关，还和用户界面的渲染相关。一个典型的例子就是授权；例如，对于部分页面，如果用户没有登录，你可能希望展示一个登录表单。

最好始终以「应该在组件体系（即渲染后的组件组成的树结构）中渲染什么」的方式来进行思考。因此，授权应该在一个组件中进行。假如我们希望在之前提到的 `Lists_show_page` 中添加这个功能。一开始我们可能会这样做：

```html
<template name="Lists_show_page">
  {{#if currentUser}}
    {{#each listId in listIdArray}}
      {{> Lists_show (listArgs listId)}}
    {{else}}
      {{> App_notFound}}
    {{/each}}
  {{else}}
    Please log in to edit posts.
  {{/if}}
</template>
```

当然，我们可能会发现我们需要在许多需要访问控制的页面共享这项功能。我们可以创造一个具有我们所需要的行为的「布局」组件，并将其他模板包裹在其中。这样，我们便可以很容易地共享功能。

你可以使用 Blaze 的「模板块 helper」来创造包裹组件（参考[「Blaze」章节](blaze.html#block-helpers)）。这里演示了如何编写一个授权模板：

```html
<template name="App_forceLoggedIn">
  {{#if currentUser}}
    {{> Template.contentBlock}}
  {{else}}
    Please log in see this page.
  {{/if}}
</template>
```

当添加这个模板之后，我们便可以简单地包裹我们的 `Lists_show_page`：

```html
<template name="Lists_show_page">
  {{#App_forceLoggedIn}}
    {{#each listId in listIdArray}}
      {{> Lists_show (listArgs listId)}}
    {{else}}
      {{> App_notFound}}
    {{/each}}
  {{/App_forceLoggedIn}}
</template>
```

使用这种方法的最大好处是，当查看 `Lists_show_page` 的代码时，你将很直观清晰地了解到，如果一个用户访问这个页面将会发生什么。

通过将一个模板包裹在多个包裹组件中，或创建一个组合了多个包裹模板的元包裹组件，你可以将多个这种类型的行为或功能组合起来。

<h2 id="changing-routes">改变路由</h2>

如果你不能用一种方式跳转到一个新的路由，那么根据新路由来更新 UI 这件事又从何说起呢？想要跳转路由，最简单的方式是提供一个可靠的 `<a>` 标签和一个 URL。虽然，你可以使用 `FlowRouter.path` 来生成 URL，但更方便的方法是使用 [`arillo:flow-router-helpers`](https://github.com/arillo/meteor-flow-router-helpers/) 包。它定义了一些有用的 helpers：

```
meteor add arillo:flow-router-helpers
```

当你安装好了这个包，你就可以在你的模板中使用这些 helpers 来展示跳转到特定路由的链接。例如，在 Todos 示例应用中，我们的导航链接定义如下： 

```html
<a href="{{pathFor 'Lists.show' _id=list._id}}" title="{{list.name}}"
    class="list-todo {{activeListClass list}}">
```

<h3 id="routing-programmatically">使用程序来控制路由</h3>

在某些时候，你可能想通过除了点击一个链接以外的其他某种用户行为来进行路由跳转。比如说，在示例应用中，在用户创建了一个新的列表时，我们希望将用户跳转到刚刚创建的列表处。为了做到这一点，我们在获得新列表的 id 时，调用了 `FlowRouter.go()` 函数：

```js
import { insert } from '../../api/lists/methods.js';

Template.App_body.events({
  'click .js-new-list'() {
    const listId = insert.call();
    FlowRouter.go('Lists.show', { _id: listId });
  }
});
```

通过使用 `FlowRouter.setParams()` 和 `FlowRouter.setQueryParams()`，你也可以只改变当前路由的某个部分。比如，当我们正在浏览某个列表时，如果我们想跳转到另一个列表，可以这样写代码：

```js
FlowRouter.setParams({_id: newList._id});
```

当然，无论哪种情况，调用 `FlowRouter.go()` 总是有效的，因此，除非你是在尝试为某种特定的用例进行优化，否则，最好使用 `FlowRouter.go()`。 

<h3 id="storing-data-in-the-url">在 URL 中保存数据</h3>

正如我们在介绍中讨论的一样，URL 不过是当前页面的一部分客户端状态的序列化结果。虽然，URL 只能是字符串，但是任何种类的数据实际上都可以被序列化为字符串。

一般来说，如果你想将任意可序列化的数据保存在 URL 参数中，你可以使用 [`EJSON.stringify()`](http://docs.meteor.com/#/full/ejson_stringify) 来将这些数据转变为字符串。你还需要使用 [`encodeURIComponent`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/encodeURIComponent) 对这个字符串进行 URL 编码以移除任何在 URL 有特殊含义的符号：

```js
FlowRouter.setQueryParams({data: encodeURIComponent(EJSON.stringify(data))});
```

你可以通过使用 [`EJSON.parse()`](http://docs.meteor.com/#/full/ejson_parse) 来从 Flow Router 中重新获取这些数据。请注意，Flow Router 已经自动为你对字符串进行了 URL 反编码：

```js
const data = EJSON.parse(FlowRouter.getQueryParam('data'));
```

<h2 id="redirecting">重定向</h2>

有时候，用户会跳转到它们不该访问的地方。有可能是他们正在查看的数据已经移动，也可能是他们正处在管理控制台页面但是却退出了登录，还可能是他们创建了一个新东西，而你想让他们跳转到展示这个东西的页面。

通常，我们可以根据用户行为进行重定向，只需要调用 `FlowRouter.go()`函数，或如同上文创建列表的例子中那样，调用其他相关函数即可。但有时，用户会直接访问到一个不存在的 URL，这时，我们需要知道如何直接进行重定向。

如果一个 URL 已经过时（有时你可能改变应用的 URL 定义），你可以在这个路由的 `action` 函数中进行重定向：

```js
FlowRouter.route('/old-list-route/:_id', {
  action(params) {
    FlowRouter.go('Lists.show', params);
  }
});
```

> 译者注：使用 `FlowRouter.go()` 进行重定向可能不是一个好方法，例如，如果你使用这种方式进行重定向，那么当用户将无法进行页面回退（思考一下为什么）。建议通过[在 trigger 中使用 redirect 函数](https://github.com/kadirahq/flow-router#redirecting-with-triggers)的方式进行重定向。

<h3 id="redirecting-dynamically">动态重定向</h3>

上述方法仅适用于静态重定向。有时候，你需要加载更多数据才知道应该重定向到什么地方。这是，你需要渲染一些组件来订阅你需要的数据。例如，在 Todos 示例应用中，我们希望将根路由 (`/`) 重定向到能够被查询到的第一个列表。为了做到这一点，我们需要渲染一个特殊的 `App_rootRedirector` 组件：

```js
FlowRouter.route('/', {
  name: 'App.home',
  action() {
    BlazeLayout.render('App_body', {main: 'App_rootRedirector'});
  }
});
```

`App_rootRedirector` 被渲染在 `App_body` 布局中，而这个布局组件则会在渲染子组件*之前*订阅用户可以查询到的所有列表，因此，当 `App_rootRedirector` 创建时，我们可以确定这里至少有一个列表会被加载，所以我们可以这样做：

```js
Template.App_rootRedirector.onCreated(() => {
  // 我们需要设置 timeout 函数，因为我们不能在一个重定向中进再进行重定向
  // 这是当前版本的 Flow Router 的一个限制
  Meteor.setTimeout(() => {
    FlowRouter.go('Lists.show', Lists.findOne());
  });
});
```

如果你希望使用某些数据，而这些数据在组件创建时还没有被订阅，你可以使用 `autorun` 和 `subscriptionsReady()` 来等待订阅完成：

```js
Template.App_rootRedirector.onCreated(() => {
  // 在这里进行订阅
  this.subscribe('lists.public');

  // 现在我们需要等待上述订阅。在等待期间，我们也需要模板展示一些加载状态。
  this.autorun(() => {
    if (this.subscriptionsReady()) {
      FlowRouter.go('Lists.show', Lists.findOne());
    }
  });
});
```

<h3 id="redirecting-after-user-action">在用户行为之后进行重定向</h3>

通常，你希望在用户完成某个动作之后，立刻通过程序控制跳转到一个新的路由。在上述创建列表的示例中，我们对它进行了*优化*——即在获得服务端关于 method 的成功响应之前就执行重定向。我们之所以能这样做，是因为我们有理由相信在绝大部分情况下，这个 method 会执行成功（参考[「UI/UX」章节](ui-ux.html#optimistic-ui)以了解更加深入的讨论）。

然而，如果我们希望等待服务端对 method 的返回结果，我们可以将重定向放在 method 的回调函数中：

```js
Template.App_body.events({
  'click .js-new-list'() {
    lists.insert.call((err, listId) => {
      if (!err) {
        FlowRouter.go('Lists.show', { _id: listId });  
      }
    });
  }
});
```

你还需要展示某种指示以表明按钮已经被点击，method 正在工作，而重定向尚未完成。不要忘了，如果 method 返回了一个错误，你还需要向用户提供某种反馈。

<h2 id="advanced">高级路由</h2>

<h3 id="404s">找不到页面</h3>

通常，如果用户输入了错误的 URL，你希望向他们展示某种缓和的 404 页面。这里有两种类型的 404 页面。第一种是，用户输入的 URL 无法匹配任何路由定义。你可以使用 `FlowRouter.notFound` 来处理这种情况：

```js
// the App_notFound template is used for unknown routes and missing lists
FlowRouter.notFound = {
  action() {
    BlazeLayout.render('App_body', {main: 'App_notFound'});
  }
};
```

第二种情况是，用户输入的 URL 是有效的，但却无法匹配任何数据。在这种情况下，URL 匹配于一个路由，但当这个路由完成数据订阅时，它发现并没有任何数据。这种情况下，最好让页面组件（负责订阅和查询数据）去渲染一个「未找到数据」的模板，而不是使用一个通用的页面模板：

```html
<template name="Lists_show_page">
  {{#each listId in listIdArray}}
    {{> Lists_show (listArgs listId)}}
  {{else}}
    {{> App_notFound}}
  {{/each}}
<template>
```

<h3 id="analytics">数据分析</h3>

通常，我们都想知道应用的哪些页面最常被访问，以及用户都是来自哪里。你可以在[「部署」章节](deployment.html#analytics)查看到如何设置基于 Flow Router 的数据统计。

<h3 id="server-side">服务端路由</h3>

正如我们之前所讨论的那样，Meteor 是一个客户端渲染应用框架，但服务端渲染的需求仍然存在。这里有两种服务端路由的主要场景。

<h4 id="server-side-apis">用于 API 访问的服务端路由</h4>

Meteor 允许你[编写底层的连接处理函数](http://docs.meteor.com/#/full/webapp)来创建任意类型的服务端 API。如果你仅仅是希望为你的 methods 和 publications 创建 RESTful 版本，你可以使用 [`simple:rest`](http://atmospherejs.com/simple/rest) 包来轻松实现。参考[「数据加载」章节](data-loading.html#publications-as-rest)和 [「Methods」章节](methods.html) 以了解更多信息。 

如果你希望进行更多控制，你可以使用更复杂的 [`nimble:restivus`](https://atmospherejs.com/nimble/restivus) 包来以任何你需要的方式创建任意数量的接口。

<h4 id="server-side-rendering">服务端渲染</h4>

Blaze UI 库不支持服务端渲染，因此，如果你使用 Blaze，你不能在服务端渲染页面。然而，React UI 支持服务端渲染。这意味着，如果你使用 React 框架，你可以在服务端渲染 HTML 页面。

虽然，Flow Router可以多多少少像我们关于 Blaze 描述的那样渲染 React 组件，但截止这篇文章编写时，它对 SSR（服务端渲染）的支持[仍处于实验阶段](https://kadira.io/blog/meteor/meteor-ssr-support-using-flow-router-and-react)。然而，如果你想要在 Meteor 使用 SSR，这恐怕是目前最好的方法了。
