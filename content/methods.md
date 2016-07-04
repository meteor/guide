---
title: "Method"
order: 12
description: 如何使用 Method, Meteor 通过使用 Method 将数据变化写入数据库
discourseTopicId: 19662
---

读完全文，你将能够：

1. Meteor 中 Method 指什么，以及他们是怎样工作的。
2. Method 的定义及调用实践。
3. 如何使用 Method 抛出和处理错误。
4. 如何从表单中调用 Method。

<h2 id="what-is-a-method">什么是 Method？</h2>

Method 是 Meteor 的远程程序调用系统, 用于保存用户在客户端输入的数据和事件。如果你熟悉 REST APIs 或者 HTTP，你可以把它想象为 POST 请求到服务器，但是它是在一种针对现代 web 应用设计的形式中。 在文章的最后，我们会谈到使用 Method 在哪些方面优于使用 HTTP 端点。

Method 的核心是作为服务器的 API 端点，你可以在服务器定义一个 Method，然后在客户端定义它的副本，然后调用 Method，将数据写入数据库，并获得 返回值。Meteor Methods 紧密集成 Meteor 的发布/订阅和数据装载系统，实现 UI 优化[Optimistic UI](http://info.meteor.com/blog/optimistic-ui-with-meteor-latency-compensation) —— 在客户端模拟服务器端操作的能力，让你的应用运行得更快。

我们在谈到 Meteor Method 时会用大些字母 M， 是为了区别 JavaScript 中的 methods。

<h2 id="defining-and-calling">定义和调用 Methods</h2>

<h3 id="basic">基础 Method</h3>

在一个简单的应用，定义一个 Method 是跟定义一个函数一样简单的事。在一个复杂的应用，你会需要一些额外的功能，使得 Method 更强大，更加容易测试。首先，我们讲一下如何使用 Meteor 核心 API 定义 Method，在后面的章节我们会讲到如何使用自己创建的封装包建立更好的 Method 工作流。

<h4 id="basic-defining">定义</h4>

这里提供如何使用内建的[`Meteor.methods`API](http://docs.meteor.com/#/full/meteor_methods) 来定义 Method。注意，Method 的定义应该是装载在客户端和服务器端来优化 UI 的通用代码。如果您的 Method 有秘密的程序代码，请参考[安全相关章节](security.html#secret-code)来了解如何从客户端隐藏秘密的程序代码。

下面这个案例添加了包： `aldeed:simple-schema`，这个包在很多文章都有提到，是用来验证 Method 参数的。

```js
Meteor.methods({
  'todos.updateText'({ todoId, newText }) {
    new SimpleSchema({
      todoId: { type: String },
      newText: { type: String }
    }).validate({ todoId, newText });

    const todo = Todos.findOne(todoId);

    if (!todo.editableBy(this.userId)) {
      throw new Meteor.Error('todos.updateText.unauthorized',
        'Cannot edit todos in a private list that is not yours');
    }

    Todos.update(todoId, {
      $set: { text: newText }
    });
  }
});
```

<h4 id="basic-calling">调用</h4>

从客户端和服务器中使用[`Meteor.call`](http://docs.meteor.com/#/full/meteor_call) 可以调用 Method。请注意，您应该只在一些代码需要从客户端调用的情况下使用 Method; 如果你只是想在服务器端实现代码模块化，应该使用常规的 JavaScript 函数，而不是 Method。

我们如何从客户端调用 Method:

```js
Meteor.call('todos.updateText', {
  todoId: '12345',
  newText: 'This is a todo item.'
}, (err, res) => {
  if (err) {
    alert(err);
  } else {
    // success!
  }
});
```

如果 Method 抛出一个错误，你会得到回调的第一个参数。如果没有抛出错误，第二个参数会返回结果，第一个参数 `err` 会被设置为 `undefined`。有关错误的详细信息，请参阅以下有关错误处理的部分。

<h3 id="advanced-boilerplate">高级 Method boilerplate</h3>

Meteor Method 有几个特点并非立竿见影，但是每一个复杂的应用程序都会或多或少使用到 Method。这些功能是以向后兼容的方式逐年递增的，所以解锁 Method 的全部功能，需要大量的 boilerplate。在这篇文章中，我们首先会告诉你所有的代码，你需要写每一个功能，那么下一节我们将谈论 Method 封装包，使得 Method 的使用更加方便。

一个好的 Method 应该具备以下功能：

1. 在运行 Method 本体前能自身验证数据
2. 较容易覆盖 Method 用于测试。
3. 较容易通过自定义的用户 ID 调用方法，特别是在测试的时候（推荐文章 [Discover Meteor two-tiered methods pattern](https://www.discovermeteor.com/blog/meteor-pattern-two-tiered-methods/)）。
4. 通过 JS 函数模块调用方法，而非通过字符串。
5. 通过 Method 模拟返回值来获得所插入文档的 ID.
6. 如果客户端验证失败也避免调用服务器端的 Method, 不至于占用服务器资源。

<h4 id="advanced-boilerplate-defining">定义</h4>

```js
export default updateText = {
  name: 'todos.updateText',

  // 将有效性验证分离，这样可以独立运行（1）
  validate(args) {
    new SimpleSchema({
      todoId: { type: String },
      newText: { type: String }
    }).validate(args)
  },

  // 将方法本体分离，这样可以单独调用（3）
  run({ todoId, newText }) {
    const todo = Todos.findOne(todoId);

    if (!todo.editableBy(this.userId)) {
      throw new Meteor.Error('todos.updateText.unauthorized',
        'Cannot edit todos in a private list that is not yours');
    }

    Todos.update(todoId, {
      $set: { text: newText }
    });
  },

  // 通过引用 JS 对象来调用方法（4）
  // 我们可以在实现 Method 的时候指定 Meteor.apply 选项，而不是要求调用者指定。
  call(args, callback) {
    const options = {
      returnStubValue: true,     // (5)
      throwStubExceptions: true  // (6)
    }

    Meteor.apply(this.name, [args], options, callback);
  }
};

// 实际上通过 Meteor 的 DDP 系统注册 Method
Meteor.methods({
  [updateText.name]: function (args) {
    updateText.validate.call(this, args);
    updateText.run.call(this, args);
  }
})
```

<h4 id="advanced-boilerplate-calling">调用</h4>

现在调用 Method 就跟调用一个 JavaScript 函数一样简单了：

```js
import { updateText } from './path/to/methods.js';

// 首先调用 Method
updateText.call({
  todoId: '12345',
  newText: 'This is a todo item.'
}, (err, res) => {
  if (err) {
    alert(err);
  } else {
    // 成功！
  }
});

// 调用有效性
updateText.validate({ wrong: 'args'});

// 在测试中通过自定义的客户 ID 调用方法
updateText.run.call({ userId: 'abcd' }, {
  todoId: '12345',
  newText: 'This is a todo item.'
});
```
正如你所看到的，这种 Method 调用方式可以有一个更好的开发流程, 你可以更容易地分开处理的 Method 的不同部位，更容易测试你的代码，而不必深挖 Meteor 内部工作原理。但这种 Method 调用方式需要你在定义 Method 的时候写了很多 boilerplate。

<h3 id="validated-method">使用包 mdg:validated-method 来实现 Method 的高级功能</h3>

为了帮助你在在定义 Method 的时候写正确的 boilerplate，我们已经发布了一个包：`mdg:validated-method`，可以帮你省下很多工作量。下面是与上述相同的 Method，在包中定义：

```js
export const updateText = new ValidatedMethod({
  name: 'todos.updateText',
  validate: new SimpleSchema({
    todoId: { type: String },
    newText: { type: String }
  }).validator(),
  run({ todoId, newText }) {
    const todo = Todos.findOne(todoId);

    if (!todo.editableBy(this.userId)) {
      throw new Meteor.Error('todos.updateText.unauthorized',
        'Cannot edit todos in a private list that is not yours');
    }

    Todos.update(todoId, {
      $set: { text: newText }
    });
  }
});
```

在这里 Method 的调用跟前面的一样，但在这里 Method 的定义是非常简单的。我们相信，这种定义方法可以让你清楚地看到定义 Method 的重点，通过 Method 的名称，预期参数的格式，以及 JavaScript 命名空间可以调用该方法。

<h2 id="errors">错误处理</h2>

常规的 JavaScript 函数，我们以抛出`Error`对象的形式来发现错误。Meteor Method 也是以这种形式抛出错误，但也会出现复杂的情形，比如 `Error` 对象会通过 WebSocket 被发往客户端。引发错误工作几乎相同的方式，微复杂之处在于，某些情况下这个错误对象会通过 WebSocket 被发往客户端。
译注：事实上，作为 DDP 消息的一种，错误对象并不总是通过 WebSocket 传送。当 WebSocket 因为某些原因不可用的时候，Meteor 会尝试使用轮询的方式建立 DDP 连接来传送 DDP 消息。

<h3 id="throwing-errors">从 Method 抛出错误</h3>

Meteor 介绍了两种 JavaScript 错误：[`Meteor.Error`](http://docs.meteor.com/#/full/meteor_error) 以及 [`ValidationError`](https://atmospherejs.com/mdg/validation-error). 这两种错误应该和常见的 JavaScript 错误区分开来。

<h4 id="internal-server-errors">内部服务器错误引发的常见错误</h4>

当服务器内部有错误，不需要报告给客户端时，抛出一个常规的 JavaScript 错误对象。客户端将会接收到一个没有细节的完全不透明的内部服务器错误报告。

<h4 id="meteor-error">一般运行时错误下的 Meteor.Error</h4>

当服务器在已知条件下不能够完成用户所需的操作，应该抛出一个描述性的 `Meteor.Error` 对象给客户端。在 Todos 应用程序中，我们使用这种方法报告当前用户无权完成某个操作，或该操作不被应用允许——例如，删除最后一个公开的 Todo 列表。

`Meteor.Error` 有三个参数： `error`, `reason`, 和 `details`.

1. `error` 应该是简短，唯一，机器可读的错误代码串，客户端可以翻译它并了解发生了什么。最好是在 `error` 加上 Method 的名称作为前缀，例如：`'todos.updateText.unauthorized'`。
2. `reason` 是为开发者简短描述发生的错误。它应该能够提供足够的信息用于调试错误。`reason` 参数不应该直接对终端用户可见，这意味着需要在发送错误信息前在服务器端实现国际化，UI 开发者在考虑 UI 展示时也应该思考 Method 的实现
3. `details` 不是必需的，可以用于提供额外的信息帮助客户端发现哪里出错。另外，它还可以结合 `error` 参数给终端用户提供更有用的错误信息。
译注：
1. 在一些应用中你会看到 error 参数是一个 HTTP 响应码，如 404。我们不建议在这里使用响应码，因为它不容易被理解，尤其是使用不常用响应码的时候。
2. 在不需要国际化的情况下，常用 reason 参数直接作为显示的信息。尽管如此，按照这份指南来做总不会吃亏的。

<h4 id="validation-error">参数验证错误下的 ValidationError</h4>

因为参数类型错误而导致调用方法失败，最好抛出一个 `ValidationError`，原理跟 `Meteor.Error` 一样，但它有一个产生标准错误格式的定制构造器，可以被不同的表单和验证库读取。如果你从表单中调用 Method，抛出一个 `ValidationError`会使得错误信息在表单中某一位置展示更加简单。

像上面那样，当你使用 `mdg:validated-method` 和 `aldeed:simple-schema`, 就会抛出这种错误。

查看此文了解更多错误格式 [`mdg:validation-error` 文档](https://atmospherejs.com/mdg/validation-error).

<h3 id="handling-errors">处理错误</h3>

当调用一个 Method 的时候，任何抛出的错误都会在回调中返回。这个时候要识别错误类型并将信息传递给用户。在这个例子中，Method 几乎不会抛出 `ValidationError` 或者内部服务器错误，所以我们只会处理未经授权错误：

```js
// 调用 Method
updateText.call({
  todoId: '12345',
  newText: 'This is a todo item.'
}, (err, res) => {
  if (err) {
    if (err.error === 'todos.updateText.unauthorized') {
      // 在一个真实的应用中你可能不需要调用 alert 展示警报；你会有一个好看的 UI 界面去展示这些错误，还可以用 i18n 库从错误代码中自动产生信息。
      alert('You aren\'t allowed to edit this todo item');
    } else {
      // 未知错误，在 UI 端处理
    }
  } else {
    // 成功！
  }
});
```

在下面章节我们将讨论如何处理 `ValidationError`。

<h3 id="throw-stub-exceptions">Method 在客户端模拟运行时的错误</h3>

当一个 Method 被调用时，它通常运行两次—— 一次在客户端上，以模拟优化 UI 的输出结果，第二次在服务器上进行数据库的实际变化操作。这意味着，如果 Method 抛出一个错误，它可能会在客户端和服务器上都出现错误。出于这个原因，`ValidatedMethod` 打开 Meteor 中的无证选项，以避免在客户端模拟抛出错误时继续调用服务器端执行。

这种运行方式在 Method 发生错误时对节省服务器资源有帮助，但要确保的是，当 Method 在服务器端可以成功时客户端的模拟不会发生错误（例如你不用在客户端加载数据用于 Method 模拟）。在这个例子中，你可以把服务器端的代码封装成一个模块，用于检验方法模拟：

```js
if (!this.isSimulation) {
  // 此处逻辑依赖于服务器端环境
}
```

<h2 id="method-form">从表单调用 Method</h2>

`ValidationError` 最主要的用途是在 Method 和调用它们的表单之间进行简单的集成。在一般情况下，你的应用很可能有在 UI 上的表单跟 Method 是一对一映射的。首先，为我们的业务逻辑定义一个 Method：

```js
// 该 Method 对表单验证需求进行编码。
// 通过在 Method 中定义表单，我们在同一个地方进行客户端和服务器端的验证。
export const insert = new ValidatedMethod({
  name: 'Invoices.methods.insert',
  validate: new SimpleSchema({
    email: { type: String, regEx: SimpleSchema.RegEx.Email },
    description: { type: String, min: 5 },
    amount: { type: String, regEx: /^\d*\.(\d\d)?$/ }
  }).validator(),
  run(newInvoice) {
    // 在这里我们可以确认参数 newInvoice 已验证。

    if (!this.userId) {
      throw new Meteor.Error('Invoices.methods.insert.not-logged-in',
        'Must be logged in to create an invoice.');
    }

    Invoices.insert(newInvoice)
  }
});
```

让我们定义一个简单的 HTML 表单：

```html
<template name="Invoices_newInvoice">
  <form class="Invoices_newInvoice">
    <label for="email">Recipient email</label>
    <input type="email" name="email" />
    {{#each error in errors "email"}}
      <div class="form-error">{{error}}</div>
    {{/each}}

    <label for="description">Item description</label>
    <input type="text" name="description" />
    {{#each error in errors "description"}}
      <div class="form-error">{{error}}</div>
    {{/each}}

    <label for="amount">Amount owed</label>
    <input type="text" name="amount" />
    {{#each error in errors "amount"}}
      <div class="form-error">{{error}}</div>
    {{/each}}
  </form>
</template>
```

现在，我们将通过 JavaScript 语言来更好地处理该表单：

```js
import { insert } from '../api/invoices/methods.js';

Template.Invoices_newInvoice.onCreated(function() {
  this.errors = new ReactiveDict();
});

Template.Invoices_newInvoice.helpers({
  errors(fieldName) {
    return this.errors.get(fieldName);
  }
});

Template.Invoices_newInvoice.events({
  'submit .Invoices_newInvoice'(event, instance) {
    const data = {
      email: event.target.email.value,
      description: event.target.description.value,
      amount: event.target.amount.value
    };

    insert.call(data, (err, res) => {
      if (err) {
        if (err.error === 'validation-error') {
          // 初始化错误对象
          const errors = {
            email: [],
            description: [],
            amount: []
          };

          // 遍历从 Method 返回的验证错误
          err.details.forEach((fieldError) => {
            // XXX i18n
            errors[fieldError.name].push(fieldError.type);
          });

          // 更新 ReactiveDict，错误会在 UI 显示。
          instance.errors.set(errors);
        }
      }
    });
  }
});
```

As you can see, there is a fair amount of boilerplate to handle errors nicely in a form, but most of it can be easily abstracted by an off-the-shelf form framework or a simple application-specific wrapper of your own design.
正如你所看到的，表单中有大量的 boilerplate 可以用来处理错误，大部分 boilerplate 可以通过现成的表单框架或你自己封装的、基于特定应用的设计来实现。

<h2 id="loading-data">通过 Methods 加载数据</h2>

因为 Method 可以作为通用 RPC，所以也可以被用于获取数据而不用通过 publications。对比通过 publications 加载数据，使用 Methods 加载数据有有点也有不足，但到目前为止，我们建议总是通过 publications 加载数据。

当服务器端数据变化时，服务器端一个复杂计算的结果不变，在这种情况下用 Method 获取数据非常有用，通过 Method 获取数据的最大不足就是获取的数据不会自动加载到 Minimongo，即 Meteor 的客户端数据缓存，所以需要你手动管理数据生命周期。

<h4 id="local-collection">使用一个本地数据集来存储并显示来自 Method 的数据</h4>

数据集可以方便地在客户端存储数据。如果你不是通过 subscriptions 的方式获取数据，可以手动将数据添加到数据集。 下面我们来看一个例子，将复杂的算法用于计算拥有众多玩家的一系列游戏的平均得分。我们不想使用 publications 加载数据因为我们想要控制何时让它运行，而不想让数据自动缓存。

首先，你需要创建一个本地数据集， 该数据集只存在于客户端而不跟服务器端的数据集联系。在[数据集一章](http://guide.meteor.com/collections.html#local-collections)中了解更多。

```js
// 在客户端声明一个本地数据集
// 传送一个 `null` 作为参数
ScoreAverages = new Mongo.Collection(null);
```

现在，如果你通过 Method 获取数据，可以放在该数据集里面：

```js
import { calculateAverages } from '../api/games/methods.js';

function updateAverages() {
  // 清除结果缓存
  ScoreAverages.remove({});

  // 调用 Method 用于复杂计算
  calculateAverages.call((err, res) => {
    res.forEach((item) => {
      ScoreAverages.insert(item);
    });
  });
}
```

现在我们可以在一个 UI 组件中使用名为 `ScoreAverages` 的本地数据集中的数据，跟我们使用 MongoDB 数据集 是完全一样的。它不会自动更新，每次计算结果的时候都需要调用 `updateAverages`.

<h2 id="advanced">进阶概念</h2>

虽然根据 Meteor 的入门手册你可以很轻松地在简单应用中使用 Meteor, 但是了解 Method 是如何运作的可以帮你更好地使用它。类似 Meteor 这样的框架为你做了很多工作，缺点之一就是你不总是理解究竟是怎样运作的，所以学习一些核心概念是很重要的。

<h3 id="call-lifecycle">Method 调用的生命周期</h3>

当一个 Method 被调用时，会发生以下事件：

<h4 id="lifecycle-simulation">1. Method 模拟在客户端上运行</h4>

跟所有 Method 一样，如果我们在客户端和服务器端定义该 Method, Method 模拟在调用它的客户端上运行。

客户端进入到一种特殊模式，在该模式下可以跟踪到客户端数据集的所有变化，将来可以回滚。该步骤结束后，应用的用户可以看到 UI 界面立即根据客户端数据库的内容进行更新，即使服务器还没有收到任何数据。

如果 Method 模拟抛出一个异常，默认情况下 Meteor 会忽略它并继续步骤 (2)。如果你使用 `ValidatedMethod` 或者传送一个特殊的 `throwStubExceptions` 选项到 `Meteor.apply`,那么从模拟中抛出的异常将导致服务器端的 Method 彻底停止运行。

除非在调用 Method 时传送 `returnStubValue` 选项，否则 Method 返回值会被抛弃，而不会返回到 Method 调用者。ValidatedMethod 在默认情况下会传送 `returnStubValue` 选项。

<h4 id="lifecycle-ddp-message">2. 一个 `method` DDP 消息发送到服务器</h4>

Meteor 客户端构建一个 DDP 消息并发送到服务器。消息包括 Method 的名称，参数，自动生成的代表该调用的 Method 的 ID.

<h4 id="lifecycle-server">3. Method 在服务器上运行</h4>

当服务器收到消息，会在服务器上再次运行 Method 代码。客户端的版本只是一个模拟，将会被回滚，而服务器上是真实的版本，将会被写入数据库。在服务器上运行实际的 Method 代码是很重要的，因为我们知道服务器是一个值得信赖的环境，关键的代码可以在上面按照我们所期望的方式安全地运行。

<h4 id="lifecycle-result">4. 返回值被发送到客户端</h4>

Method 在服务器上结束运行后，就会把在第二步生成的 ID , `result` 消息和返回值发送到客户端。这些数据会被储存在客户端，但不会调用 Method 回调。如果你传递一个[`onResultReceived` 选项 到 `Meteor.apply`](http://docs.meteor.com/#/full/meteor_apply)，回调就会被启动。

<h4 id="lifecycle-publications">5. DDP 发布通过 Method 实现更新</h4>

如果通过 Method 写入数据库影响到某些发布，服务器会把更新的部分发送到客户端。请注意，客户端的数据系统并不会立即根据这些变化更新应用的 UI，具体的更新我们将在下一步讨论。

<h4 id="lifecycle-updated">6. `updated` 发送到客户端，数据替换服务器结果，Method 回调启动</h4>

当相关的数据更新发送到正确的客户端后，服务器返回 Method 生命周期中的最后一条信息 —— DDP `updated` 信息和关联的 Method ID. 客户端回滚在第一步 Method 模拟中客户端数据的任何变化，并替换为第五步中服务器端发送的更新数据。

最后，传递给 `Meteor.call` 的回调实际上跟第四步中的返回值一起启动。客户端更新后再实现回调是很重要的，所以 Method 回调可以假设客户端状态反映了 Method 所做的任何改变。

<h4 id="lifecycle-error">错误情况</h4>

在上面的步骤中，我们没有考虑 Method 在服务器端执行抛出错误的情况。在这种情况下是没有返回值的，客户端也会得到一个错误值。Method 回调立即启动并返回一个错误值作为第一个参数。了解更多信息请阅读上文关于错误处理的段落。

<h3 id="methods-vs-rest">与 REST 相比之下 Method 的优点</h3>

相比在 HTTP REST 端点上建立现代应用程序，我们相信 Method 提供了一个更好的起点。我们回顾一下有哪些方面是你使用 HTTP 需要担心，而 Meteor 可以帮你控制的。这部分的目的不是跟你说 REST 有多不好 —— 而是提醒你在 Meteor 应用中你不必处理这些情况。

<h4 id="non-blocking">Methods 使用同步，非阻塞的 APIs</h4>

你可能注意到上面的 Method 例子中，当和 MongoDB 交互时我们并不需要写任何回调，但 Method 仍然具有 Node.js 和 回调风格代码的无阻塞特性。Method 使用 Fibers 协同程序库 [Fibers](https://github.com/laverdet/node-fibers)，这样你可以在代码中使用返回值和抛出的错误，并且避免处理很多嵌套的回调。

<h4 id="ordered">Methods 总是按顺序运行和返回</h4>

访问 REST API 有时会出现会出现你按顺序提出两个请求，但结果却没有按顺序返回的情况。Meteor 的运行机制可以保证这种情况不会在 Method 中产生。当从客户端收到多个调用 Method 的请求时， Meteor 会按顺序执行完一个 Method 再执行下一个 Method. 如果在一个需要长时间运行的 Method 中你想要禁用这种方法，可以使用 `this.unblock()` [`this.unblock()`](http://docs.meteor.com/#/full/method_unblock)，这样即使 Method 在运行，下一个 Method 也可以同时进行。因为 Meteor 是基于 WebSockets 而非 HTTP 的，所有的 Method 和调用和返回结果都可以保证以正确的顺序到达。你也可以把 `wait: true` 传递给 `Meteor.apply`，这样就可以实现不运行其他 Method 除非该特殊的 Method 返回结果。

<h4 id="change-tracking">跟踪变动，以便优化 UI</h4>

当 Method 开始模拟，服务器开始运行， Meteor 跟踪所有数据库的变化。在这种情况下， Meteor 数据系统回滚 Meteor 模拟中出现的变化并替换服务器端的数据。如果不是这种自动的数据库跟踪，要实现正确地优化 UI 是很难的。

<h3 id="calling-method-from-method">从一个 Method 中调用另一个 Method</h3>

有时候你需要从一个 Method 中调用另外一个 Method, 或许你已经实现了一些功能，但你需要可以添加一个封装包自动填充参数。这是一个完全正常模式，Method 也为之做了很多工作：

1. 在一个客户端的 Method 模拟中，调用另一个 Method 并不会因此像服务器发送额外的请求 —— 我们假设服务器端 Method 的执行会这样做，它确实运行了调用 Method 的模拟，所以客户端的模拟跟服务器上发生的模拟非常贴近。
2. 在一个服务器端的 Method 模拟中，调用并运行另外一个 Method,  跟在客户端运行一样。这说明 Method 像往常一样运行，`userId`, `connection` 等环境变量可以从原来调用的 Method 获取。
<h3 id="consistent-id-generation">一致的 ID 生成和 UI 的优化</h3>

当从客户端 Method 模拟中往 Minimongo 插入文件时，每个文件的 `_id` 属性都是随机产生的字符串。当 Method 在服务器端执行时，在插入数据库前会重新生成 ID。如果执行不当的话，服务器端产生的 ID 可能就是不同的，在这种情况下，当 Method 模拟回滚并替换服务器的数据时，可能会出现页面闪烁和 UI 布局问题。但 Meteor 觉不会让这种情况发生！

Meteor Method 和调用该 Method 的客户端共享一个随机产生的种子，所以客户端和服务器端产生的 ID 都可以保证是相同的。这意味着当 Method 被发送到服务器时你也可以放心使用客户端产生的 ID，相信当 Method 执行结束时所产生的 ID 跟我们所使用的 ID 是一样的。一个应用场景就是用于在数据库创建文件，然后立即重定向到包含该文件 ID 的 url.

<h3 id="retries">Method 重试</h3>

如果从客户端调用 Method, 用户在收到结果前互联网连接断开的情况，Meteor 默认 Method 并没有实际执行。当重新建立连接后，会重新调用 Method。这意味着，在特定情况下，Method 可以被多次发送。这种情况应该很少发生，如果调用其它方法会产生不利后果，应该投入额外的努力确保 Method 是幂等的 —— 多次调用并不会影响数据库的更新。

大部分 Method 默认是幂等的。如果插入命令发生两次的话会报错因为生成的 ID 会互相冲突。执行第二次删除不会有任何结果，大部分的更新操作如 `$set` 在执行第二次时结果不变。 所需要关心的代码运行两次的唯一地方是 MongoDB 的叠加更新，如 `$inc` 和 `$push`，以及调用外部的 API.

<h3 id="comparison-with-allow-deny">与 allow/deny 的比较</h3>

Meteor 核心 API 有一个代替 Method 从客户端操作数据的选择。相对于明确定义 Method 和参数，你可以直接在客户端使用 `insert`, `update`, 和 `remove`，只需要安全规则 allow/deny 中说明，[`allow`](http://docs.meteor.com/#/full/allow) 和 [`deny`](http://docs.meteor.com/#/full/deny)。在 Meteor 手册中，我们强烈建议使用 Method，而不是 allow/deny. 了解更多 allow/deny 可能出现的问题请阅读文章 [安全性章节](security.html#allow-deny).

跟 allow/deny 的特点相比，对 Meteor Method 的了解一向存在误解，包括 Method 更难实现 UI 优化。但是，客户端的 `insert`, `update`, 和 `remove` 是在 Method 的基础上执行的，所以 Method 更加强大。只需要在客户端和服务器上定义 Method 代码，就默认得到优化的 UI, 如以上方法生命周期部分所述。
