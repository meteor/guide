---
title: Using Atmosphere Packages
order: 28
discourseTopicId: 20193
---

<h2 id="atmosphere-searching">Searching for packages</h2>

There are a few ways to search for Meteor packages published to Atmosphere:

1. Search on the [Atmosphere website](https://atmospherejs.com/).
2. Use `meteor search` from the command line.
3. Use a community package search website like [Fastosphere](http://fastosphere.meteor.com/).

The main Atmosphere website provides additional curation features like trending packages, package stars, and flags, but some of the other options can be faster if you're trying to find a specific package. For example, you can use `meteor show kadira:flow-router` from the command line to see the description of that package and different available versions.

<h3 id="atmosphere-naming">Package naming</h3>

You may notice that, with the exception of Meteor platform packages, all packages on Atmosphere have a name of the form `prefix:name`. The prefix is the name of the organization or user that published the package. Meteor uses such a convention of package naming to make sure that it's clear who has published a certain package, and to avoid an ad-hoc namespacing convention. Meteor platform packages do not have any `prefix:`.

<h2 id="installing-atmosphere">Installing Atmosphere Packages</h2>

To install an Atmosphere package, you simply run `meteor add`:

```bash
meteor add kadira:flow-router
```

This will add the newest version of the desired package that is compatible with the other packages in your app. If you want to specify a particular version, you can specify it by adding a suffix to the package name, like so: `meteor add kadira:flow-router@2.10.0`.

Regardless of how you add the package to your app, its actual version will be tracked in the file at `.meteor/versions`. This means that anybody collaborating with you on the same app is guaranteed to have the same package versions as you. If you want to update to a newer version of a package after installing it, use `meteor update`. You can run `meteor update` without any arguments to update all packages and Meteor itself to their latest versions, or pass a specific package to update just that one, for example `meteor update kadira:flow-router`.

If your app is running when you add a new package, Meteor will automatically download it and restart your app for you.

> The actual files for a given version of an Atmosphere package are stored in your local `~/.meteor/packages` directory.

<h2 id="using-atmosphere">Using Atmosphere Packages</h2>

To use an Atmosphere Package in your app you can import it with the `meteor/` prefix:

```js
import { SimpleSchema } from 'meteor/aldeed:simple-schema';
```

Typically a package will export one or more symbols which you'll need to grab with the destructuring syntax. Sometimes a package will have no exports and simply have side effects when included in your app. In such cases you don't need to import the package at all.

<h2 id="peer-npm-dependencies">Peer npm Dependencies</h2>

Atmosphere packages can ship with contained [npm dependencies](writing-atmosphere-packages.html#npm-dependencies), in which case you don't need to do anything to make them work. However, some Atmosphere packages will expect that you have installed certain "peer" npm dependencies in your application.

Typically the package will warn you if you have not done so. For example, if you install the [`react-meteor-data`](https://atmospherejs.com/meteor/react-meteor-data) atmosphere package into your app, you'll also need to [install](#installing-npm) the [`react`](https://www.npmjs.com/package/react) and [`react-addons-pure-render-mixin`](https://www.npmjs.com/package/react-addons-pure-render-mixin) npm packages:

```bash
meteor npm install --save react react-addons-pure-render-mixin
meteor add react-meteor-data
```
