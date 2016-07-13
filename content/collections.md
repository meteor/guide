---
title: 数据集与数据结构
order: 10
description: 如何在 Meteor 中定义、使用和维护 MongoDB 数据集。
discourseTopicId: 19660
---

通过阅读本章，你可以了解到：
1. 在 Meteor 中不同类型的 MongoDB 数据集及如何使用它们。
2. 如何为某个数据集定义数据结构以控制它的内容。
3. 当定义你的数据结构时，应该考虑什么。
4. 当对一个集合进行写入时，如何确保写入数据符合它的数据结构。
5. 如何小心地改变你的数据结构。
6. 如何应对数据记录的间的关联。

<h2 id="mongo-collections">Meteor 中的 MongoDB 数据集</h2>

本质上讲，一个 web 应用为它的用户提供一个观察和修改一组持久化数据的能力。无论是管理你的待办事项，还是为你所挑选的车下订单，你都是在和一个固定但又持续变化的数据层在打交道。

在 Meteor 中，这个数据层通常会存储在 MongoDB 中。在 MongoDB 中一组相关联的数据被称作「数据集」。在 Meteor 中，你通过[数据集](http://docs.meteor.com/#/full/mongo_collection)来存取 MongoDB 中的数据，这是应用的主要持久化机制。

然而，数据集不仅仅是保存和获取数据的工具。它们也是交互的核心，与用户期待从最好的应用那里获得的那种用户体验相关。Meteor 让这种用户体验很容易就得到实现。

在本文中，我们将深入观察数据集在 Meteor 框架的不同位置上是如何工作的，及如何弄明白其中的大多数。

<h3 id="server-collections">服务器端数据集</h3>

当你在服务器端创建了一个数据集：

```js
Todos = new Mongo.Collection('Todos');
```

实际上是在 MongoDB 中创建了一个数据集，和一个在服务端使用数据集的接口。这是一个相当明了的、构建在 Node MongoDB 驱动之上的层，只不过是同步 API：

```js
// 这行代码直到插入操作结束才会执行完成
Todos.insert({_id: 'my-todo'});
// 这一行会返回些数据
const todo = Todos.findOne({_id: 'my-todo'});
// 看，根本没有回调！
console.log(todo);
```

<h3 id="client-collections">客户端数据集</h3>

在客户端上，用同一行代码创建一个数据集：

```js
Todos = new Mongo.Collection('Todos');
```

做的却是完全不同的一套！

在客户端，并没有到 MongoDB 数据库的直接连接，事实上，做同步 API 也是不可能的（或者说，可能不是你想要的）。其实在客户端，某个数据集是数据库在某个客户端的**缓存**。做到这样的效果要感谢 [Minimongo](https://www.meteor.com/mini-databases) 库——一种在内存中存储数据，全 JS 的 MongoDB API 实现。这意味着，在客户端你写这个：

```js
// 这行改变了内存中 Minimongo 数据结构
Todos.insert({_id: 'my-todo'});
// 这行进行数据查询
const todo = Todos.findOne({_id: 'my-todo'});
// 所以这里就立刻输出了！
console.log(todo);
```

从服务器端数据集（由 MongoDB 所支撑的）装载数据到客户端数据集（客户端内存中）的方式是文章[数据装载](data-loading.html)的主题。通常而言，从某个 **publication** 来**订阅**，将会从服务器端推送数据到客户端。通常你只要假设客户端具有完整的服务器端 MongoDB 数据集的最新片段。

向服务器写回数据，一般你会使用一个 **Meteor 方法**。这也就是[方法](methods.html)这一章的主题。

<h3 id="local-collections">本地数据集</h3>

这里有第三种方式来使用 Meteor 中的数据集。你可以客户端或者服务端使用 `null` 作为参数创建一个数据集：

```js
SelectedTodos = new Mongo.Collection(null);
```

此时实际上是创建了一个**本地数据集**。这是一个没有数据库连接的数据集（通常一个数据集要么是直接连接到数据库服务器上，要么通过客户端订阅）。
本地数据集是一种便利的在内存中使用 Minimongo 库的全部功能的方法。举个例子，如果你需要针对你的数据执行复杂的查询，你可以使用本地数据集来替代一个简单的数组。又或者你会想利用它的反应性在客户端驱动一些 UI ，不过这在 Meteor 中不是事儿。

<h2 id="schemas">定义一个数据结构</h2>

虽然 MongoDB 是一种无结构的数据库，赋予数据结构最大限度的灵活性，但是最好使用一个数据结构来约束集合中的内容，使其符合已知的格式。如果你不这么做的话，那么最终你需要去编写防卫性的代码来检查并确认你的数据结构，并且，你同样需要在数据从数据库*读出来的时候*确定结构以防止使用时出现问题，而不仅仅是在数据*写入*数据库的时候。大多数情况下，你会发现*读取较写入更为频繁*，所以使用数据结构写入数据通常会更简单，也很少会产生 bug。
在 Meteor 中，aldeed:simple-schema 是一款杰出的数据结构包。它提供富有表现力且基于 MongoDB 的数据结构，用于插入和更新文档。
使用 simple-schema 来编写一个数据结构，你可以轻松的创建一个基于 SimpleSchema 类的新实例：

```js
Lists.schema = new SimpleSchema({
  name: {type: String},
  incompleteCount: {type: Number, defaultValue: 0},
  userId: {type: String, regEx: SimpleSchema.RegEx.Id, optional: true}
});
```

这个 Todos 示例应用中的例子用一些简单的规则定义了一个数据结构：

1. 我们指定列表中的 `name` 字段是必选项，必须是一个字符串。
2. 我们指定 `incompleteCount` 是一个数字类型，在没有指定的情况下插入时设置为 `0`。
3. 我们指定 `userId` 是可选项，必须是一个字符串，并且看起来像用户文档中的 ID。

我们直接在 `Lists` 命名空间上附加数据结构，允许我们随时依靠数据结构来检查对象，比如在一个表单或者 Method 中。在下一章节中，我们将了解到，当向集合中写入数据时如何自动化地使用这个数据结构。

你能看到我们用相对少的代码明显地限制了列表的格式。你可以在 [Simple Schema 文档](http://atmospherejs.com/aldeed/simple-schema)中阅读更多详尽内容。

<h3 id="validating-schemas">用数据结构来验证数据</h3>

现在我们有了个数据结构，怎么用呢？

用它验证一个文档很简单，只要这样做：

```js
const list = {
  name: 'My list',
  incompleteCount: 3
};

Lists.schema.validate(list);
```

在这个案例中，这个列表符合数据结构的定义， `validate()` 将能正常运行。但是如果：

```js
const list = {
  name: 'My list',
  incompleteCount: 3,
  madeUpField: 'this should not be here'
};

Lists.schema.validate(list);
```

`validate()` 则会甩出一个 `ValidationError`，它包含了 `list` 对象的错误原因。

<h3 id="validation-error">`ValidationError`</h3>

什么是一个 [`ValidationError`](https://github.com/meteor/validation-error/)？它是一种在 Meteor 中，在修改数据集时指出用户输入的错误。通常，一个 `ValidationError` 的细节被用来标识那些不符合数据结构定义的输入。我们将在 [methods 章节](methods.html#validation-error)中了解更多关于 `ValidationError` 如何工作的细节。

<h2 id="schema-design">设计你的数据结构</h2>

现在你已经熟悉了 Simple Schema 的基本 API，应当去考虑一些影响你设计数据结构的、关于 Meteor 系统设计方面的约束。尽管通常讲你可以建立一个与任何 MongoDB 数据结构类似的 Meteor 数据图式，但仍有一些重要的细节需要记得。
首当其冲的是如何使用 DDP，它是 Meteor 的数据装载协议，通过网络传输文档。要记住的关键是，当在文档发生变更时，DDP 发送顶层的字段变动。这意味着如果你的文档中有大量复杂的子字段频繁的变更，DDP 会发送很多不必要的变更数据。
例如，在「纯」MongoDB 中你需要设计视图以便每个待办列表的文档都拥有一个叫 `todos` 的字段，包含待办项目的数组：

```js
Lists.schema = new SimpleSchema({
  name: {type: String},
  todos: {type: [Object]}
});
```

这个架构的问题在于，由于刚提到过的 DDP 行为，每次对于某个列表中*任意*的待办项的变化，DDP 就会通过网络发送该列表中*整组*待办项目。这是因为 DDP 对于在 `todos` 数组第三项中的 text 字段的「变化」没有任何的概念，因此它简单地认为 `todos` 字段变成了全新的数组。

<h3 id="denormalization">正常化和多个数据集</h3>

上面这样的问题是，我们为了子文档需要创建更多的数据集。在 Todos 应用里，我们需要 `Todos` 和 `Lists` 两个数据集来存储每个待办列表里的每一条待办事项。所以你需要做一点你通常对一个 SQL 数据库要做的事情，比如使用另一个表（数据集）中的键（`todo.listId`）将两个文档联系在一起。

在 Meteor 中，这样做通常意味着更少的问题，尤其是在涉及有交集的文档时， (we might need one set of users to render one screen of our app, and an intersecting set for another), which may stay on the client as we move around the application. So in that scenario there is an advantage to separating the subdocuments from the parent.

However, given that MongoDB prior to version 3.2 doesn't support queries over multiple collections ("joins"), we typically end up having to denormalize some data back onto the parent collection. Denormalization is the practice of storing the same piece of information in the database multiple times (as opposed to a non-redundant "normal" form). MongoDB is a database where denormalizing is encouraged, and thus optimized for this practice.

In the case of the Todos application, as we want to display the number of unfinished todos next to each list, we need to denormalize `list.incompleteTodoCount`. This is an inconvenience but typically reasonably easy to do as we'll see in the section on [abstracting denormalizers](#abstracting-denormalizers) below.

Another denormalization that this architecture sometimes requires can be from the parent document onto sub-documents. For instance, in Todos, as we enforce privacy of the todo lists via the `list.userId` attribute, but we publish the todos separately, it might make sense to denormalize `todo.userId` also. To do this, we'd need to be careful to take the `userId` from the list when creating the todo, and updating all relevant todos whenever a list's `userId` changed.

<h3 id="designing-for-future">Designing for the future</h3>

An application, especially a web application, is rarely finished, and it's useful to consider potential future changes when designing your data schema. As in most things, it's rarely a good idea to add fields before you actually need them (often what you anticipate doesn't actually end up happening, after all).

However, it's a good idea to think ahead to how the schema may change over time. For instance, you may have a list of strings on a document (perhaps a set of tags). Although it's tempting to leave them as a subfield on the document (assuming they don't change much), if there's a good chance that they'll end up becoming more complicated in the future (perhaps tags will have a creator, or subtags later on?), then it might be easier in the long run to make a separate collection from the beginning.

The amount of foresight you bake into your schema design will depend on your app's individual constraints, and will need to be a judgement call on your part.

<h3 id="schemas-on-write">Using schemas on write</h3>

Although there are a variety of ways that you can run data through a Simple Schema before sending it to your collection (for instance you could check a schema in every method call), the simplest and most reliable is to use the [`aldeed:collection2`](https://atmospherejs.com/aldeed/collection2) package to run every mutator (`insert/update/upsert` call) through the schema.

To do so, we use `attachSchema()`:

```js
Lists.attachSchema(Lists.schema);
```

What this means is that now every time we call `Lists.insert()`, `Lists.update()`, `Lists.upsert()`, first our document or modifier will be automatically checked against the schema (in subtly different ways depending on the exact mutator).

<h3 id="default-value">`defaultValue` and data cleaning</h3>

One thing that Collection2 does is ["clean" the data](https://github.com/aldeed/meteor-simple-schema#cleaning-data) before sending it to the database. This includes but is not limited to:

1. Coercing types - converting strings to numbers
2. Removing attributes not in the schema
3. Assigning default values based on the `defaultValue` in the schema definition

However, sometimes it's useful to do more complex initialization to documents before inserting them into collections. For instance, in the Todos app, we want to set the name of new lists to be `List X` where `X` is the next available unique letter.

To do so, we can subclass `Mongo.Collection` and write our own `insert()` method:

```js
class ListsCollection extends Mongo.Collection {
  insert(list, callback) {
    if (!list.name) {
      let nextLetter = 'A';
      list.name = `List ${nextLetter}`;

      while (!!this.findOne({name: list.name})) {
        // not going to be too smart here, can go past Z
        nextLetter = String.fromCharCode(nextLetter.charCodeAt(0) + 1);
        list.name = `List ${nextLetter}`;
      }
    }

    // Call the original `insert` method, which will validate
    // against the schema
    return super.insert(list, callback);
  }
}

Lists = new ListsCollection('Lists');
```

<h3 id="hooks">Hooks on insert/update/remove</h3>

The technique above can also be used to provide a location to "hook" extra functionality into the collection. For instance, when removing a list, we *always* want to remove all of its todos at the same time.

We can use a subclass for this case as well, overriding the `remove()` method:

```js
class ListsCollection extends Mongo.Collection {
  // ...
  remove(selector, callback) {
    Package.todos.Todos.remove({listId: selector});
    return super.remove(selector, callback);
  }
}
```

This technique has a few disadvantages:

1. Mutators can get very long when you want to hook in multiple times.
2. Sometimes a single piece of functionality can be spread over multiple mutators.
3. It can be a challenge to write a hook in a completely general way (that covers every possible selector and modifier), and it may not be necessary for your application (because perhaps you only ever call that mutator in one way).

A way to deal with points 1. and 2. is to separate out the set of hooks into their own module, and simply use the mutator as a point to call out to that module in a sensible way. We'll see an example of that [below](#abstracting-denormalizers).

Point 3. can usually be resolved by placing the hook in the *Method* that calls the mutator, rather than the hook itself. Although this is an imperfect compromise (as we need to be careful if we ever add another Method that calls that mutator in the future), it is better than writing a bunch of code that is never actually called (which is guaranteed to not work!), or giving the impression that your hook is more general that it actually is.

<h3 id="abstracting-denormalizers">Abstracting denormalizers</h3>

Denormalization may need to happen on various mutators of several collections. Therefore, it's sensible to define the denormalization logic in one place, and hook it into each mutator with one line of code. The advantage of this approach is that the denormalization logic is one place rather than spread over many files, but you can still examine the code for each collection and fully understand what happens on each update.

In the Todos example app, we build a `incompleteCountDenormalizer` to abstract the counting of incomplete todos on the lists. This code needs to run whenever a todo item is inserted, updated (checked or unchecked), or removed. The code looks like:

```js
const incompleteCountDenormalizer = {
  _updateList(listId) {
    // Recalculate the correct incomplete count direct from MongoDB
    const incompleteCount = Todos.find({
      listId,
      checked: false
    }).count();

    Lists.update(listId, {$set: {incompleteCount}});
  },
  afterInsertTodo(todo) {
    this._updateList(todo.listId);
  },
  afterUpdateTodo(selector, modifier) {
    // We only support very limited operations on todos
    check(modifier, {$set: Object});

    // We can only deal with $set modifiers, but that's all we do in this app
    if (_.has(modifier.$set, 'checked')) {
      Todos.find(selector, {fields: {listId: 1}}).forEach(todo => {
        this._updateList(todo.listId);
      });
    }
  },
  // Here we need to take the list of todos being removed, selected *before* the update
  // because otherwise we can't figure out the relevant list id(s) (if the todo has been deleted)
  afterRemoveTodos(todos) {
    todos.forEach(todo => this._updateList(todo.listId));
  }
};
```

We are then able to wire in the denormalizer into the mutations of the `Todos` collection like so:

```js
class TodosCollection extends Mongo.Collection {
  insert(doc, callback) {
    doc.createdAt = doc.createdAt || new Date();
    const result = super.insert(doc, callback);
    incompleteCountDenormalizer.afterInsertTodo(doc);
    return result;
  }
}
```

Note that we only handled the mutators we actually use in the application---we don't deal with all possible ways the todo count on a list could change. For example, if you changed the `listId` on a todo item, it would need to change the `incompleteCount` of *two* lists. However, since our application doesn't do this, we don't handle it in the denormalizer.

Dealing with every possible MongoDB operator is difficult to get right, as MongoDB has a rich modifier language. Instead we focus on just dealing with the modifiers we know we'll see in our app. If this gets too tricky, then moving the hooks for the logic into the Methods that actually make the relevant modifications could be sensible (although you need to be diligent to ensure you do it in *all* the relevant places, both now and as the app changes in the future).

It could make sense for packages to exist to completely abstract some common denormalization techniques and actually attempt to deal with all possible modifications. If you write such a package, please let us know!

<h2 id="migrations">Migrating to a new schema</h2>

As we discussed above, trying to predict all future requirements of your data schema ahead of time is impossible. Inevitably, as a project matures, there will come a time when you need to change the schema of the database. You need to be careful about how you make the migration to the new schema to make sure your app works smoothly during and after the migration.

<h3 id="writing-migrations">Writing migrations</h3>

A useful package for writing migrations is [`percolate:migrations`](https://atmospherejs.com/percolate/migrations), which provides a nice framework for switching between different versions of your schema.

Suppose, as an example, that we wanted to add a `list.todoCount` field, and ensure that it was set for all existing lists. Then we might write the following in server-only code (e.g. `/server/migrations.js`):

```js
Migrations.add({
  version: 1,
  up() {
    Lists.find({todoCount: {$exists: false}}).forEach(list => {
      const todoCount = Todos.find({listId: list._id})).count();
      Lists.update(list._id, {$set: {todoCount}});
    });
  },
  down() {
    Lists.update({}, {$unset: {todoCount: true}});
  }
});
```

This migration, which is sequenced to be the first migration to run over the database, will, when called, bring each list up to date with the current todo count.

To find out more about the API of the Migrations package, refer to [its documentation](https://atmospherejs.com/percolate/migrations).

<h3 id="bulk-data-changes">Bulk changes</h3>

If your migration needs to change a lot of data, and especially if you need to stop your app server while it's running, it may be a good idea to use a [MongoDB Bulk Operation](https://docs.mongodb.org/v3.0/core/bulk-write-operations/).

The advantage of a bulk operation is that it only requires a single round trip to MongoDB for the write, which usually means it is a *lot* faster. The downside is that if your migration is complex (which it usually is if you can't just do an `.update(.., .., {multi: true})`), it can take a significant amount of time to prepare the bulk update.

What this means is if users are accessing the site whilst the update is being prepared, it will likely go out of date! Also, a bulk update will lock the entire collection while it is being applied, which can cause a significant blip in your user experience if it takes a while. For these reason, you often need to stop your server and let your users know you are performing maintenance while the update is happening.

We could write our above migration like so (note that you must be on MongoDB 2.6 or later for the bulk update operations to exist). We can access the native MongoDB API via [`Collection#rawCollection()`](http://docs.meteor.com/#/full/Mongo-Collection-rawCollection):

```js
Migrations.add({
  version: 1,
  up() {
    // This is how to get access to the raw MongoDB node collection that the Meteor server collection wraps
    const batch = Lists.rawCollection().initializeUnorderedBulkOp();
    Lists.find({todoCount: {$exists: false}}).forEach(list => {
      const todoCount = Todos.find({listId: list._id}).count();
      // We have to use pure MongoDB syntax here, thus the `{_id: X}`
      batch.find({_id: list._id}).updateOne({$set: {todoCount}});
    });

    // We need to wrap the async function to get a synchronous API that migrations expects
    const execute = Meteor.wrapAsync(batch.execute, batch);
    return execute();
  },
  down() {
    Lists.update({}, {$unset: {todoCount: true}});
  }
});
```

Note that we could make this migration faster by using an [Aggregation](https://docs.mongodb.org/v2.6/aggregation/) to gather the initial set of todo counts.

<h3 id="running-migrations">Running migrations</h3>

To run a migration against your development database, it's easiest to use the Meteor shell:

```js
// After running `meteor shell` on the command line:
Migrations.migrateTo('latest');
```

If the migration logs anything to the console, you'll see it in the terminal window that is running the Meteor server.

To run a migration against your production database, run your app locally in production mode (with production settings and environment variables, including database settings), and use the Meteor shell in the same way. What this does is run the `up()` function of all outstanding migrations, against your production database. In our case, it should ensure all lists have a `todoCount` field set.

A good way to do the above is to spin up a virtual machine close to your database that has Meteor installed and SSH access (a special EC2 instance that you start and stop for the purpose is a reasonable option), and running the command after shelling into it. That way any latencies between your machine and the database will be eliminated, but you still can be very careful about how the migration is run.

**Note that you should always take a database backup before running any migration!**

<h3 id="breaking-changes">Breaking schema changes</h3>

Sometimes when we change the schema of an application, we do so in a breaking way -- so that the old schema doesn't work properly with the new code base. For instance, if we had some UI code that heavily relied on all lists having a `todoCount` set, there would be a period, before the migration runs, in which the UI of our app would be broken after we deployed.

The simple way to work around the problem is to take the application down for the period in between deployment and completing the migration. This is far from ideal, especially considering some migrations can take hours to run (although using [Bulk Updates](#bulk-data-changes) probably helps a lot here).

A better approach is a multi-stage deployment. The basic idea is that:

1. Deploy a version of your application that can handle both the old and the new schema. In our case, it'd be code that doesn't expect the `todoCount` to be there, but which correctly updates it when new todos are created.
2. Run the migration. At this point you should be confident that all lists have a `todoCount`.
3. Deploy the new code that relies on the new schema and no longer knows how to deal with the old schema. Now we are safe to rely on `list.todoCount` in our UI.

Another thing to be aware of, especially with such multi-stage deploys, is that being prepared to rollback is important! For this reason, the migrations package allows you to specify a `down()` function and call `Migrations.migrateTo(x)` to migrate _back_ to version `x`.

So if we wanted to reverse our migration above, we'd run
```js
// The "0" migration is the unmigrated (before the first migration) state
Migrations.migrateTo(0);
```

If you find you need to roll your code version back, you'll need to be careful about the data, and step carefully through your deployment steps in reverse.

<h3 id="migration-caveats">Caveats</h3>

Some aspects of the migration strategy outlined above are possibly not the most ideal way to do things (although perhaps appropriate in many situations). Here are some other things to be aware of:

1. Usually it is better to not rely on your application code in migrations (because the application will change over time, and the migrations should not). For instance, having your migrations pass through your Collection2 collections (and thus check schemas, set autovalues etc) is likely to break them over time as your schemas change over time.

  One way to avoid this problem is simply to not run old migrations on your database. This is a little bit limiting but can be made to work.

2. Running the migration on your local machine will probably make it take a lot longer as your machine isn't as close to the production database as it could be.

Deploying a special "migration application" to the same hardware as your real application is probably the best way to solve the above issues. It'd be amazing if such an application kept track of which migrations ran when, with logs and provided a UI to examine and run them. Perhaps a boilerplate application to do so could be built (if you do so, please let us know and we'll link to it here!).

<h2 id="associations">Associations between collections</h2>

As we discussed earlier, it's very common in Meteor applications to have associations between documents in different collections. Consequently, it's also very common to need to write queries fetching related documents once you have a document you are interested in (for instance all the todos that are in a single list).

To make this easier, we can attach functions to the prototype of the documents that belong to a given collection, to give us "methods" on the documents (in the object oriented sense). We can then use these methods to create new queries to find related documents.

<h3 id="collection-helpers">Collection helpers</h3>

We can use the [`dburles:collection-helpers`](https://atmospherejs.com/dburles/collection-helpers) package to easily attach such methods (or "helpers") to documents. For instance:

```js
Lists.helpers({
  // A list is considered to be private if it has a userId set
  isPrivate() {
    return !!this.userId;
  }
});
```

Once we've attached this helper to the `Lists` collection, every time we fetch a list from the database (on the client or server), it will have a `.isPrivate()` function available:

```js
const list = Lists.findOne();
if (list.isPrivate()) {
  console.log('The first list is private!');
}
```

<h3 id="association-helpers">Association helpers</h3>

Now we can attach helpers to documents, it's simple to define a helper that fetches related documents

```js
Lists.helpers({
  todos() {
    return Todos.find({listId: this._id}, {sort: {createdAt: -1}});
  }
});
```

Now we can easily find all the todos for a list:

```js
const list = Lists.findOne();
console.log(`The first list has ${list.todos().count()} todos`);
```
