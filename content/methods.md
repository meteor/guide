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

          // 查看从 Method 返回的验证错误
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
正如你所看到的，表单中有大量的 boilerplate 可以用来处理错误，大部分 boilerplate 可以通过现成的表单结构或者你自己封装的简单应用程序提取出来。

<h2 id="loading-data">通过 Methods 加载数据</h2>

因为 Method 可以作为通用 RPC，所以也可以被用于获取数据而不是发布数据。对比通过发布加载数据，使用 Methods 加载数据有有点也有不足，但到目前为止，我们建议总是通过发布加载数据。

当服务器端数据变化时，服务器端一个复杂计算的结果不变，在这种情况下用 Method 获取数据非常有用，通过 Method 获取数据的最大不足就是获取的数据不会自动加载到 Minimongo，Meteor 的客户端数据缓存，所以需要你手动管理数据周期。

<h4 id="local-collection">使用本地存储储存和展示通过 Method 获取的数据</h4>

Collections are a very convenient way of storing data on the client side. If you're fetching data using something other than subscriptions, you can put it in a collection manually. Let's look at an example where we have a complex algorithm for calculating average scores from a series of games for a number of players. We don't want to use a publication to load this data because we want to control exactly when it runs, and don't want the data to be cached automatically.

First, you need to create a _local collection_ - this is a collection that exists only on the client side and is not tied to a database collection on the server. Read more in the [Collections article](http://guide.meteor.com/collections.html#local-collections).

```js
// In client-side code, declare a local collection
// by passing `null` as the argument
ScoreAverages = new Mongo.Collection(null);
```

Now, if you fetch data using a Method, you can put into this collection:

```js
import { calculateAverages } from '../api/games/methods.js';

function updateAverages() {
  // Clean out result cache
  ScoreAverages.remove({});

  // Call a Method that does an expensive computation
  calculateAverages.call((err, res) => {
    res.forEach((item) => {
      ScoreAverages.insert(item);
    });
  });
}
```

We can now use the data from the local collection `ScoreAverages` inside a UI component exactly the same way we would use a regular MongoDB collection. Instead of it updating automatically, we'll need to call `updateAverages` every time we need new results.

<h2 id="advanced">Advanced concepts</h2>

While you can easily use Methods in a simple app by following the Meteor introductory tutorial, it's important to understand exactly how they work to use them effectively in a production app. One of the downsides of using a framework like Meteor that does a lot for you under the hood is that you don't always understand what is going on, so it's good to learn some of the core concepts.

<h3 id="call-lifecycle">Method call lifecycle</h3>

Here's exactly what happens, in order, when a Method is called:

<h4 id="lifecycle-simulation">1. Method simulation runs on the client</h4>

If we defined this Method in client and server code, as all Methods should be, a Method simulation is executed in the client that called it.

The client enters a special mode where it tracks all changes made to client-side collections, so that they can be rolled back later. When this step is complete, the user of your app sees their UI update instantly with the new content of the client-side database, but the server hasn't received any data yet.

If an exception is thrown from the Method simulation, then by default Meteor ignores it and continues to step (2). If you are using `ValidatedMethod` or pass a special `throwStubExceptions` option to `Meteor.apply`, then an exception thrown from the simulation will stop the server-side Method from running at all.

The return value of the Method simulation is discarded, unless the `returnStubValue` option is passed when calling the Method, in which case it is returned to the Method caller. ValidatedMethod passes this option by default.

<h4 id="lifecycle-ddp-message">2. A `method` DDP message is sent to the server</h4>

The Meteor client constructs a DDP message to send to the server. This includes the Method name, arguments, and an automatically generated Method ID that represents this particular Method invocation.

<h4 id="lifecycle-server">3. Method runs on the server</h4>

When the server receives the message, it executes the Method code again on the server. The client side version was a simulation that will be rolled back later, but this time it's the real version that is writing to the actual database. Running the actual Method logic on the server is crucial because the server is a trusted environment where we know that security-critical code will run the way we expect.

<h4 id="lifecycle-result">4. Return value is sent to the client</h4>

Once the Method has finished running on the server, it sends a `result` message to the client with the Method ID generated in step 2, and the return value itself. The client stores this for later use, but _doesn't call the Method callback yet_. If you pass the [`onResultReceived` option to `Meteor.apply`](http://docs.meteor.com/#/full/meteor_apply), that callback is fired.

<h4 id="lifecycle-publications">5. Any DDP publications affected by the Method are updated</h4>

If we have any publications on the page that have been affected by the database writes from this Method, the server sends the appropriate updates to the client. Note that the client data system doesn't reveal these updates to the app UI until the next step.

<h4 id="lifecycle-updated">6. `updated` message sent to the client, data replaced with server result, Method callback fires</h4>

After the relevant data updates have been sent to the correct client, the server sends back the last message in the Method life cycle - the DDP `updated` message with the relevant Method ID. The client rolls back any changes to client side data made in the Method simulation in step 1, and replaces them with the actual changes sent from the server in step 5.

Lastly, the callback passed to `Meteor.call` actually fires with the return value from step 4. It's important that the callback waits until the client is up to date, so that your Method callback can assume that the client state reflects any changes done inside the Method.

<h4 id="lifecycle-error">Error case</h4>

In the list above, we didn't cover the case when the Method execution on the server throws an error. In that case, there is no return value, and the client gets an error instead. The Method callback is fired instantly with the returned error as the first argument. Read more about error handling in the section about errors below.

<h3 id="methods-vs-rest">Benefits of Methods over REST</h3>

We believe Methods provide a much better primitive for building modern applications than REST endpoints built on HTTP. Let's go over some of the things you get for free with Methods that you would have to worry about if using HTTP. The purpose of this section is not to convince you that REST is bad - it's just to remind you that you don't need to handle these things yourself in a Meteor app.

<h4 id="non-blocking">Methods use synchronous-style APIs, but are non-blocking</h4>

You may notice in the example Method above, we didn't need to write any callbacks when interacting with MongoDB, but the Method still has the non-blocking properties that people associate with Node.js and callback-style code. Meteor uses a coroutine library called [Fibers](https://github.com/laverdet/node-fibers) to enable you to write code that uses return values and throws errors, and avoid dealing with lots of nested callbacks.

<h4 id="ordered">Methods always run and return in order</h4>

When accessing a REST API, you will sometimes run into a situation where you make two requests one after the other, but the results arrive out of order. Meteor's underlying machinery makes sure this never happens with Methods. When multiple Method calls are received _from the same client_, Meteor runs each Method to completion before starting the next one. If you need to disable this functionality for one particularly long-running Method, you can use [`this.unblock()`](http://docs.meteor.com/#/full/method_unblock) to allow the next Method to run while the current one is still in progress. Also, since Meteor is based on Websockets instead of HTTP, all Method calls and results are guaranteed to arrive in the order they are sent. You can also pass a special option `wait: true` to `Meteor.apply` to wait to send a particular Method until all others have returned, and not send any other Methods until this one returns.

<h4 id="change-tracking">Change tracking for Optimistic UI</h4>

When Method simulations and server-side executions run, Meteor tracks any resulting changes to the database. This is what lets the Meteor data system roll back the changes from the Method simulation and replace them with the actual writes from the server. Without this automatic database tracking, it would be very difficult to implement a correct Optimistic UI system.

<h3 id="calling-method-from-method">Calling a Method from another Method</h3>

Sometimes, you'll want to call a Method from another Method. Perhaps you already have some functionality implemented and you want to add a wrapper that fills in some of the arguments automatically. This is a totally fine pattern, and Meteor does some nice things for you:

1. Inside a client-side Method simulation, calling another Method doesn't fire off an extra request to the server - the assumption is that the server-side implementation of the Method will do it. However, it does run the _simulation_ of the called Method, so that the simulation on the client closely matches what will happen on the server.
2. Inside a Method execution on the server, calling another Method runs that Method as if it were called by the same client. That means the Method runs as usual, and the context - `userId`, `connection`, etc - are taken from the original Method call.

<h3 id="consistent-id-generation">Consistent ID generation and optimistic UI</h3>

When you insert documents into Minimongo from the client-side simulation of a Method, the `_id` field of each document is a random string. When the Method call is executed on the server, the IDs are generated again before being inserted into the database. If it were implemented naively, it could mean that the IDs generated on the server are different, which would cause undesirable flickering and re-renders in the UI when the Method simulation was rolled back and replaced with the server data. But this is not the case in Meteor!

Each Meteor Method invocation shares a random generator seed with the client that called the Method, so any IDs generated by the client and server Methods are guaranteed to be the same. This means you can safely use the IDs generated on the client to do things while the Method is being sent to the server, and be confident that the IDs will be the same when the Method finishes. One case where this is particularly useful is if you want to create a new document in the database, then immediately redirect to a URL that contains that new document's ID.

<h3 id="retries">Method retries</h3>

If you call a Method from the client, and the user's Internet connection disconnects before the result is received, Meteor assumes that the Method didn't actually run. When the connection is re-established, the Method call will be sent again. This means that, in certain situations, Methods can be sent more than once. This should only happen very rarely, but in the case where an extra Method call could have negative consequences it is worth putting in extra effort to ensure that Methods are idempotent - that is, calling them multiple times doesn't result in additional changes to the database.

Many Method operations are idempotent by default. Inserts will throw an error if they happen twice because the generated ID will conflict. Removes on collections won't do anything the second time, and most update operators like `$set` will have the same result if run again. The only places you need to worry about code running twice are MongoDB update operators that stack, like `$inc` and `$push`, and calls to external APIs.

<h3 id="comparison-with-allow-deny">Historical comparison with allow/deny</h3>

The Meteor core API includes an alternative to Methods for manipulating data from the client. Instead of explicitly defining Methods with specific arguments, you can instead call `insert`, `update`, and `remove` directly from the client and specify security rules with [`allow`](http://docs.meteor.com/#/full/allow) and [`deny`](http://docs.meteor.com/#/full/deny). In the Meteor Guide, we are taking a strong position that this feature should be avoided and Methods used instead. Read more about the problems with allow/deny in the [Security article](security.html#allow-deny).

Historically, there have been some misconceptions about the features of Meteor Methods as compared with the allow/deny feature, including that it was more difficult to achieve Optimistic UI when using Methods. However, the client-side `insert`, `update`, and `remove` feature is actually implemented _on top of_ Methods, so Methods are strictly more powerful. You get great default Optimistic UI just by defining your Method code on the client and the server, as described in the Method lifecycle section above.
