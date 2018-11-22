---
title: Vue
description: How to use the Vue frontend rendering library, with Meteor.
---
<h2 id="introduction">Vue + Meteor Integration Guide</h2>

[Vue](https://vuejs.org/v2/guide/) (pronounced /vjuː/, like view) is a progressive front-end framework for building user interfaces.
Unlike other [monolithic frameworks](https://vuejs.org/v2/guide/comparison.html), Vue is designed from the ground up to be incrementally adoptable.

Vue focuses on the view layer in a way that's easy to pick learn and integrate with other libraries or existing projects.

Vue powers sophisticated Single-Page Applications when used in combination with [modern tooling](https://vuejs.org/v2/guide/single-file-components.html) and [supporting libraries](https://github.com/vuejs/awesome-vue#components--libraries).

Vue has an excellent [guide and documentation](https://vuejs.org).

<h3 id="why-use-vue-with-meteor">Why use Meteor with Vue?</h3>

[Meteor](https://guide.meteor.com/#what-is-meteor) is a full-stack JavaScript platform for developing modern web and mobile applications. 

Meteor offers a sophisticated real-time backend for a variety of view frameworks like Vue, React, Angular, Blaze, and others.  Meteor brings:

- Built-in socket connection w/ reactive updates
- Client-side Minimongo database for reusable code on client/server
- Automatically push changes to subscribed clients on database writes
- Express middle-ware integration
- A curated set of packages for Meteor and Node.js
- Sophisticated build tool 
- Cordova/Phonegap integration for HTML mobile apps

Vue & Meteor are two best of breed frameworks.

<h3 id="vue-resources">Vue Documentation</h3>

Vue already has an [excellent guide](https://vuejs.org/v2/guide/) with many advanced topics: [Routing](https://router.vuejs.org/), [Code Structure and Style Guide](https://vuejs.org/v2/style-guide/), [SSR (Server-side Rendering)](https://ssr.vuejs.org/), and [State Management with Vuex](https://vuex.vuejs.org/).  

[Nuxt](https://nuxtjs.org/) is an excellent static site generator that also uses Vue.

<h2 id="integrating-vue-with-meteor">Integrating Vue With Meteor</h2>

To start a new project:  

```sh
meteor create .
```

To install Vue in Meteor 1.7, you should add it as an npm dependency:

```sh
meteor npm install --save vue
```

To support [Vue's Single File Components](https://vuejs.org/v2/guide/single-file-components.html) with the .vue file extensions, install the following Meteor package created by Vue Core developer [Akryum (Guillaume Chau)](https://github.com/meteor-vue/vue-meteor/tree/master/packages/vue-component).

```sh
meteor add akryum:vue-component
```

You will end up with at least 3 files: 

1. `/client/App.vue` (The root component of your app)
2. `/client/main.js` (Initializing the Vue app in Meteor startup)
3. `/client/main.html` (containing the body with the #app div)

We need a base HTML document that has the `app` id.  If you created a new project from `meteor create .`, put this in your `/client/main.html`.

```html
<body>
  <div id="app"></div>
</body>
```

You can now start writing .vue files in your app with the following format.  If you created a new project from `meteor create .`, put this in your `/client/App.vue`.

```vuejs
<template>
  <div>
    <p>This is a Vue component and below is the current date:<br />{{date}}</p>
  </div>
</template>

<script>
export default {
  data() {
    return {
      date: new Date(),
    };
  }
}
</script>

<style scoped>
  p {
    font-size: 2em;
    text-align: center;
  }
</style>
```

You can render the Vue component hierarchy to the DOM by using the below snippet in you client startup file.  If you created a new project from `meteor create .`, put this in your `/client/main.js`.

```javascript
import Vue from 'vue';
import App from '/client/App.vue';
import '/client/main.html';

Meteor.startup(() => {
  new Vue({
    el: '#app',
    ...App,
  });
});
```


<h3 id="vue-and-meteor-realtime-data-layer">Adding Vue to Meteor</h3>

Meteor brings the advantages of a realtime data layer to your app: reactivity, methods, publications, and subscriptions.

To integrate Vue, first install the `vue-meteor-tracker` package from NPM:

```
meteor npm install --save vue-meteor-tracker
```

Next, initialize Vue in `/client/main.js`:

```javascript
import Vue from 'vue';
import VueMeteorTracker from 'vue-meteor-tracker';   // here!
Vue.use(VueMeteorTracker);                           // here!

import App from '/client/App.vue';
import '/client/main.html';

Meteor.startup(() => {
  new Vue({
    el: '#app',
    ...App,
  });
});
```

To run your [Vue+Meteor app in Meteor 1.8](https://github.com/meteor-vue/vue-meteor/pull/335), we need to turn off HMR: 

``` bash
NO_HMR=1 meteor
```

Open the app and you should see a message with the time.  Success!


<h3 id="vue-and-meteor-realtime-data-layer-subscriptions">Using Meteor in Vue components</h3>

Currently our Vue application only shows the time it was loaded, without interacting with Meteor.  Let's:

- add an "Update Time" button
- show a list of times the server has started

To flex Meteor's realtime plumbing, we'll create:

1.  A [Meteor Collection](https://docs.meteor.com/api/collections.html) called `Time` with a `currentTime` doc.
2.  A [Meteor Publication](https://guide.meteor.com/data-loading.html#publications-and-subscriptions) called `Time` that sends all documents
3.  A [Meteor Method](https://guide.meteor.com/methods.html#what-is-a-method) called `UpdateTime` to update the `currentTime` doc.
4.  A [Meteor Subscription](https://docs.meteor.com/api/pubsub.html) to `Time`
5.  [Vue/Meteor Reactivity](https://github.com/meteor-vue/vue-meteor-tracker) to update the Vue component

The first 3 steps are basic Meteor:

1)  Create a [Meteor Collection](https://docs.meteor.com/api/collections.html) in `/imports/collections/Time.js`

``` javascript
console.log(`/imports/collections/Time.js: Collection for client and server`);
Time = new Mongo.Collection("time");
```

2)  Create a [Meteor Publication](https://guide.meteor.com/data-loading.html#publications-and-subscriptions) in `/imports/publications/Time.js`

``` javascript
Meteor.publish('Time', function () {
  console.log(`/imports/publications/Time.js: Time Publication subscribed`);
  return Time.find({});
});
```

3)  Create a [Meteor Method](https://guide.meteor.com/methods.html#what-is-a-method) in `/imports/methods/UpdateTime.js`

``` javascript
Meteor.methods({
  UpdateTime() {
    console.log(`/imports/methods/UpdateTime.js: UpdateTime Method`);
    Time.upsert('currentTime', { $set: { time: new Date() } });
  },
});
```

Then, let's import these files into our server.  Update `/server/main.js` to use our new stuff:

``` javascript
import { Meteor } from 'meteor/meteor';

import '/imports/collections/Time';
import '/imports/publications/Time';
import '/imports/methods/UpdateTime';

Meteor.startup(() => {
  // Update the current time
  Meteor.call('UpdateTime');
  // Add a new doc on each start.
  Time.insert({ time: new Date() });  // Saves time server was started!
  // Print the current time from the database
  console.log(`/server/main.js: The time is now ${Time.findOne().time}`);
});
```

Next, let's make a [Meteor `/settings.json` file](https://galaxy-guide.meteor.com/environment-variables.html#settings-example):

``` json
{ "public": { "hello": "world" } }
```

Then, let's remove [`autopublish`](https://guide.meteor.com/security.html#checklist) - a prototyping package that automatically publishes all your data to the client:

``` bash
meteor remove autopublish
```

Restart your Vue Meteor app to use the `settings.json` and refresh your browser.  You should see the same web page, with new console messages on both the browser and server - sometimes the from the same code!

``` bash
NO_HMR=1 meteor --settings=settings.json 
```

4) and 5) Great, let's integrate this with Vue using [Vue Meteor Tracker](https://github.com/meteor-vue/vue-meteor-tracker).

```javascript
<template>
  <div>
    <div v-if="!$subReady.Time">Loading...</div>
    <div v-else>
      <p>Hello {{hello}},
        <br>The time is now: {{currentTime}}
      </p>
      <button @click="updateTime">Update Time</button>
      <p>Startup times:</p>
      <ul>
        <li v-for="t in TimeCursor">
          {{t.time}}  -  {{t._id}}
        </li>
      </ul>
      <p>Meteor settings</p>
      <pre><code>
        {{settings}}
      </code></pre>
    </div>
  </div>
</template>

<script>
import '/imports/collections/Time';

export default {
  data() {
    console.log('Sending non-Meteor data to Vue component');
    return {
      hello: 'World',
      settings: Meteor.settings.public,   // not Meteor reactive
    }
  },
  // Vue Methods
  methods: {  
    updateTime() {
      console.log('Calling Meteor Method UpdateTime');
      Meteor.call('UpdateTime');          // not Meteor reactive
    }
  },
  // Meteor reactivity
  meteor: {
    // Subscriptions - Errors not reported spelling and capitalization.
    $subscribe: {
      'Time': []
    },
    // A helper function to get the current time
    currentTime () {
      console.log('Calculating currentTime');
      var t = Time.findOne('currentTime') || {};
      return t.time;
    },
    // A Minimongo cursor on the Time collection is added to the Vue instance
    TimeCursor () {
      // Here you can use Meteor reactive sources like cursors or reactive vars
      // as you would in a Blaze template helper
      return Time.find({}, {
        sort: {time: -1}
      })
    },
  }
}
</script>

<style scoped>
  p { font-size: 2em; }
</style>
```

Meteor should refresh the page automatically with: 

  - the current time
  - a button to Update the current time
  - startup times for the server (added to the Time collection on startup)
  - The Meteor settings from your settings file

Noice!  You've successfully integrated Meteor and Vue.  Go build great things together!  Please [PR](https://github.com/meteor/guide/blob/master/CONTRIBUTING.md) if you have improvements for this guide.


<h2 id="advanced-topics">Advanced Topics</h2>

<h3 id="ssr-code-splitting">SSR and Code Splitting</h3>

> Todo: This section is still under development.  Extending the above example into the code below would be a [great contribution](https://github.com/meteor/guide/blob/master/CONTRIBUTING.md).  Thank you!

Single Page Applications (SPAs) create a streamlined user experience once loaded.  But, they can take awhile to load, and are difficult for web crawlers to read their contents.

Server Side Rendering (SSR) is important for content that is served for search engines (Google, etc), content aggregators (FaceBook, etc), and for page load times.  Not all websites need to be SPA's, or even applications.  [Nuxt](https://nuxtjs.org/) is an excellent static site generator that also uses Vue.

Vue has [an excellent guide on how to render your Vue application on the server](https://vuejs.org/v2/guide/ssr.html). It includes code splitting, async data fetching and many other practices that are used in most apps that require this.  Better integration of these features is a huge opportunity for Vue and Meteor.

Integrating Vue SSR with Meteor is similar to integrating Vue with [Express](https://expressjs.com/).  However instead of defining a wildcard route, we can use Meteor's [server-render package](https://docs.meteor.com/packages/server-render.html) that exposes an `onPageLoad` function. 

Every time a page is requested from the server side, the `onPageLoad` function is triggered, and we can connect our code described on the [VueJS SSR Guide](https://ssr.vuejs.org/guide/#integrating-with-a-server).

To add the packages, run:

```sh
meteor add server-render
meteor npm install --save vue-server-renderer
```

Now, let's connect it all together in `main.js`:

```javascript
import { Meteor } from 'meteor/meteor';
import Vue from 'vue';
import { onPageLoad } from 'meteor/server-render';
import { createRenderer } from 'vue-server-renderer';

const renderer = createRenderer();

onPageLoad(sink => {
  console.log('onPageLoad');
  
  const url = sink.request.url.path;
  
  const app = new Vue({
    data: {
      url
    },
    template: `<div>The visited URL is: {{ url }}</div>`
  });

  renderer.renderToString(app, (err, html) => {
    if (err) {
      sink.request.status(500).end('Internal Server Error');
      return
    }
    console.log('html', html);
    
    sink.renderIntoElementById('app', html);
  })
})
```

Luckily [Akryum](https://github.com/akryum) has us covered and provided us with a Meteor package for this: 
[akryum:vue-ssr](https://github.com/meteor-vue/vue-meteor/tree/master/packages/vue-ssr) allows us to write our server-side code like below:

```javascript
import { VueSSR } from 'meteor/akryum:vue-ssr';
import createApp from './app';

VueSSR.createApp = function () {
  // Initialize the Vue app instance and return the app instance
  const { app } = createApp(); 
  return { app };
}
```

<h4 id="serverside-routes">Server-side Routing</h4>

Sweet, but most apps have some sort of routing functionality. We can use the VueSSR context parameter 
for this. It simply passes the Meteor server-render request url which we need to push into our router instance:

```javascript
import { VueSSR } from 'meteor/akryum:vue-ssr';
import createApp from './app';

VueSSR.createApp = function (context) {
  // Initialize the Vue app instance and return the app + router instance
  const { app, router } = createApp(); 
  
  // Set router's location from the context
  router.push(context.url);
  
  return { app };
}
```

<h3 id="style-guide">Style Guides and Structure</h3>

Like code linting and style guides are tools for making code easier and more fun to work with.  This section show how to leverage existing tools and configurations.
  
[Meteor's style guide](https://guide.meteor.com/code-style.html) and [Vue's style guide](https://vuejs.org/v2/style-guide/) can be overlapped like this:

1. [Configure your Editor](https://guide.meteor.com/code-style.html#eslint-editor)
2. [Configure eslint for Meteor](https://guide.meteor.com/code-style.html#eslint-installing)
3. [Review the Vue Style Guide](https://vuejs.org/v2/style-guide/#Rule-Categories)
4. Open up the [ESLint rules](https://eslint.org/docs/rules/) as needed.
  
Application Structure is documented here:

1. [Meteor's Application Structure](https://guide.meteor.com/structure.html#example-app-structure) is the default start.
2. [Vuex's Application Structure](https://vuex.vuejs.org/guide/structure.html) may be interesting.
