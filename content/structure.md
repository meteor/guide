---
title: Application structure
description: How to structure your Meteor app with ES2015 modules, ship code to the client and server, and split your code into multiple apps.
---

After reading this article, you'll know:

1. How a Meteor application compares to other types of applications in terms of file structure.
2. How to organize your application both for small and larger applications.
3. How to format your code and name the parts of your application in consistent and maintainable ways.

<h2 id="meteor-structure">Universal JavaScript</h2>

Meteor is a *full-stack* framework for building applications; this means Meteor applications differ from most applications in that they include code that runs on the client, code that runs on the server, and _common_ code that runs in both places. The Meteor build tool enables you to run JavaScript code easily and consistenly in both client and server environments, and includes some application structure conventions to make it easy to specify which code should run where.

<h3 id="es2015-modules">ES2015 modules</h3>

As of version 1.3, Meteor ships with full support for [ES2015 modules](https://developer.mozilla.org/en/docs/web/javascript/reference/statements/import). The ES2015 module standard is the replacement for [CommonJS](http://requirejs.org/docs/commonjs.html) and [AMD](https://github.com/amdjs/amdjs-api), which are commonly used JavaScript module format and loading systems.

In ES2015, you can make variables available outside a file using the `export` keyword. To use the variables somewhere else, you must `import` them using the path of the source. Files that export some variables are called "modules", because they represent a unit of reusable code. Explicitly importing the modules and packages you use helps you write your code in a modular way, avoiding the introduction of global symbols and "action at a distance".

Since this is a new feature introduced in Meteor 1.3, you will find a lot of code online that uses the older, more centralized conventions built around packages and apps declaring global symbols. This old system still works, so to opt-in to the new module system code must be placed in the `imports/` directory in your application. We expect a future release of Meteor will turn on modules by default for all code, because this is more aligned with how developers in the wider JavaScript community write their code.

You can read about the module system in detail in the [`modules` package README](https://github.com/meteor/meteor/tree/release-1.3/packages/modules). This package is automatically included in every new Meteor app as part of the `ecmascript` meta-package, so most apps won't need to do anything to start using modules right away.

<h3 id="importing-from-packages">Importing from packages</h3>

In Meteor, it is also simple and straightforward to use the `import` syntax to load NPM packages on the client or server, and access the package's exported symbols as you would with any other module. You can also import from Atmosphere packages, but the import path must be prefixed with `meteor` to avoid conflict with the NPM package namespace. For example, to import `HTTP` you can do `import { HTTP } from 'meteor/http'`.

<h2 id="javascript-structure">File structure</h2>

To fully use the module system and ensure that our code only runs when we ask it to, we recommend that all of your application code should be placed in the `imports/` directory. This means that the Meteor build system will only bundle and include that file if it is referenced from another file using an `import`.

Meteor will eagerly load any files outside of `imports/` in the application, but it's best to create exactly two eagerly loaded files, `client/main.js` and `server/main.js`, to define explicit entry points. Meteor ensures that any file in a directory named `server/` will only be available on the server, and likewise for `client/`.

These `main.js` files won't do anything themselves, but they should import some _startup_ modules which will run immediately on the client and server when the app loads. These modules should do any configuration necessary for the packages you are using in your app, and import the rest of your app's code.

<h3 id="example-app-structure">Example directory layout</h3>

To start, let's look at the specific example of the Todos example application, which is a great example to follow when structuring your app. Here's an overview of the directory structure:

```sh
imports/
  startup/
    client/
      routes.js                # set up all routes in the app
    server/
      fixtures.js              # fill the DB with example data on startup

  api/
    lists/                     # a unit of domain logic
      server/
        publications.js        # all list-related publications
        publications.tests.js  # tests for the list publications
      lists.js                 # definition of the Lists collection
      lists.tests.js           # tests for the behavior of that collection
      methods.js               # methods related to lists
      methods.tests.js         # tests for those methods

  ui/
    components/                # all reusable components in the application
                               # can be split by domain if there are many
    layouts/                   # wrapper components for behaviour and visuals
    pages/                     # entry points for rendering used by the router

client/
  main.js                      # client entry point, imports all client code

server/
  main.js                      # server entry point, imports all server code
```

<h3 id="structuring-imports">Structuring imports</h3>

Now that we have placed all files in the `imports/` directory, let's think about how best to organize our code using modules. It makes sense to put all code that runs when your app starts in an `imports/startup` directory. Another good idea is splitting data and business logic from UI rendering code. We suggest using directories called `imports/api` and `imports/ui` for this logical split.

Within the `imports/api` directory, it's sensible to split the code into directories based on the domain that the code is providing an API for --- typically this corresponds to the collections you've defined in your app. For instance in the Todos example app, we have the `imports/api/lists` and `imports/api/todos` domains. Inside each directory we define the collections, publications and methods used to manipulate the relevant domain data.

> Note: in a larger application, given that the todos themselves are a part of a list, it might make sense to group both of these domains into a single larger "list" module. The Todos example is small enough that we need to separate these to demonstrate modularity.

Within the `imports/ui` directory it typically makes sense to group files into directories based on the type of UI side code they define---top level `pages`, wrapping `layouts`, or reusable `components`.

For each module defined above, it makes sense to co-locate the various auxiliary files with the the base JavaScript file. For instance, a Blaze UI component should be have its template HTML, JavaScript logic, and CSS rules in the same directory. A JavaScript module with some business logic should be co-located with the unit tests for that module.

<h3 id="startup-files">Startup files</h3>

Some of your code isn't going to be a unit of business logic or UI, it's just some setup or configuration code that needs to run in the context of the app when it starts up. In the Todos example app, the `imports/startup/client/useraccounts-configuration.js` file configures the `useraccounts` login templates and routes (see the [Accounts](accounts.html) article for more information about `useraccounts`). The `imports/startup/client/routes.js` configures all of the routes and then imports *all* other code that is required on the client, forming the main entry point for the rest of the client application:

```js
import { FlowRouter } from 'meteor/kadira:flow-router';
import { BlazeLayout } from 'meteor/kadira:blaze-layout';
import { AccountsTemplates } from 'meteor/useraccounts:core';

// Import to load these templates
import '../../ui/layouts/app-body.js';
import '../../ui/pages/root-redirector.js';
import '../../ui/pages/lists-show-page.js';
import '../../ui/pages/app-not-found.js';

// Import to override accounts templates
import '../../ui/accounts/accounts-templates.js';

// Below here are the route definitions
```

On the server, we import various modules in our `server/main.js` to set up the server environment:

```js
// This defines a starting set of data to be loaded if the app is loaded with an empty db.
import '../imports/startup/server/fixtures.js';

// This file configures the Accounts package to define the UI of the reset password email.
import '../imports/startup/server/reset-password-email.js';

// Set up some rate limiting and other important security settings.
import '../imports/startup/server/security.js';

// This defines all the collections, publications and methods that the application provides
// as an API to the client.
import '../imports/api/api.js';
```

You can see that we don't actually import any variables from these files - we just import them so that they execute in that order.

<h2 id="splitting-your-app">Splitting into multiple apps</h2>

If you are writing a sufficiently complex system, there can come a time where it makes sense to split your code up into multiple applications. For example you may want to create a separate application for the administration UI (rather than checking permissions all through the admin part of your site, you can check once), or separate the code for the mobile and desktop versions of your app.

Another very common use case is splitting a worker process away from your main application so that expensive jobs do not impact on user experience of your visitors by locking up a single web server.

There are some advantages of splitting your application in this way:

 - Your client JavaScript bundle can be significantly smaller if you separate out code that a specific type of user will never use.

 - You can deploy the different applications with different scaling setups and secure them differently (for instance you might restrict access to your admin application to users behind a firewall).

 - You can allow different teams at your organization to work on the different applications independently.

However there are some challenges to splitting your code in this way that should be considered before jumping in.

<h3 id="sharing-code">Sharing code</h3>

The primary challenge is properly sharing code between the different applications you are building. The simplest approach to deal with this issue is to simply deploy the *same* application on different web servers, controlling the behavior via different [settings](deployment.md#environment). This approach allows you to easily deploy different versions with different scaling behavior but doesn't enjoy most of the other advantages stated above.

If you want to create Meteor applications with separate code, you are best sharing the code via a package system---either Meteor's package system or NPM. The easiest way to do this is to factor code that you'd like to share out into a Meteor package and add it directly into the app by placing it in the application's local `packages/` directory.

Then you have two approaches to share this package between applications:

 1. If you put both applications in a single repository (say at `app1/` and `app2/`), you can also include a directory of common packages (`packages/`) and create a symbolic link from `app1/packages/x` and `app2/packages/x` to `packages/x`.

 2. If you'd prefer to put the applications in different repositories, you can use a [git submodule](https://git-scm.com/docs/git-submodule) to include the package (which would live in its own repository) in each app's `packages/` directory.

You can also share a module (like `imports/some-module`) in a similar fashion.

<h3 id="sharing-data">Sharing data</h3>

Another important consideration is how you'll share the data between your different applications.

The simplest approach is to point both applications at the same `MONGO_URL` and allow both applications to read and write from the database directly. This works well thanks to Meteor's support for reactivity through the database. When one app changes some data in MongoDB, users of the any other app connected to the database will see the changes immediately thanks to Meteor's livequery.

However, in some cases it's better to allow one application to be the master, and control access to the data for other applications via an API. This can help if you want to deploy the different applications on different schedules and need to be conservative about how the data changes.

The simplest way to provide a server-server API is to use Meteor's built-in DDP protocol directly. This is the same way your Meteor client gets data from your server, but you can also use it to communicate between different applications. You can use [`DDP.connect()`](http://docs.meteor.com/#/full/ddp_connect) to connect from a "client" server to the master server, and then use the connection object returned to make method calls and read from publications.

XXX: Do we want to show how to mirror a publication (boilerplate)? or passthrough a method call (complexities about authentication).

XXX: Re: sharing user accounts -- what did you have in mind @sashko? The AccountsClient thing lets you authenticate against a 3rd party server from within the app but it's not clear what you should do next -- make method calls / subscribe directly against that server? Pass the resume token through to proxied method calls?