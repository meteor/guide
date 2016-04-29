---
title: Atmosphere vs. npm
order: 27
discourseTopicId: 20192
---

Building an application completely from scratch is a tall order. This is one of the main reasons you might consider using Meteor in the first place - you can focus on writing the code that is specific to your app, instead of reinventing wheels like user login and data synchronization. To streamline your workflow even further, it makes sense to use community packages from [npm](https://www.npmjs.com) and [Atmosphere](https://atmospherejs.com). Many of these packages are recommended in the guide, and you can find more in the online directories.

With the release of version 1.3, Meteor has full support for npm. In the future, there will be a time when all packages will be migrated to npm, but currently there are benefits to both systems. You can read more about the tradeoffs between Atmosphere and npm in the [Using Atmosphere Packages](using-atmosphere-packages.html) and [Using npm Packages](using-npm-packages.html) articles.

If you want to distribute and reuse code that you've written for a Meteor application, then you should consider publishing that code on npm if it's general enough to be consumed by a wider JavaScript audience. It's simple to [use npm packages in Meteor applications](using-npm-packages.html#using-npm), and possible to [use npm packages within Atmosphere packages](writing-atmosphere-packages.html#npm-dependencies), so even if your main audience is Meteor developers, npm might be the best choice.

However, if your package depends on an Atmosphere package (which, in Meteor 1.3, includes the Meteor core packages), or needs to take advantage of Meteor's [build system](build-tool.html), then writing an Atmosphere package might be the best option for now.