---
title: Vue
description: How to use the Vue frontend rendering library, with Meteor.
---

After reading this guide, you'll know:

1. What Vue is, and why you would consider using it with Meteor.
2. How to install Vue in your Meteor application, and how to use it correctly.
3. [TODO] How to structure your Vue application according to both Meteor's and Vue's style guides
4. [TODO] How to use Vue's SSR (Serverside Rendering) with Meteor. 
5. [TODO] How to integrate Vue with Meteor's realtime data layer.

<h2 id="introduction">Introduction</h2>
[Vue](https://vuejs.org/v2/guide/) (pronounced /vjuː/, like view) is a progressive framework for building user interfaces. 
Unlike other monolithic frameworks, Vue is designed from the ground up to be incrementally adoptable. 
The core library is focused on the view layer only, and is easy to pick up and integrate with other 
libraries or existing projects. On the other hand, Vue is also perfectly capable of powering sophisticated 
Single-Page Applications when used in combination with 
[modern tooling](https://vuejs.org/v2/guide/single-file-components.html) and [supporting libraries](https://github.com/vuejs/awesome-vue#components--libraries).

Vue has an excellent [guide and documentation](https://vuejs.org/v2/guide/). This guide is about integrating it with Meteor.

<h3 id="why-use-vue-with-meteor">Why use Vue with Meteor</h3>
Vue is - like React, Blaze and Angular - a frontend library. Some really nice frameworks are built around Vue. [Nuxt.js](https://nuxtjs.org) for example, aims to create a framework flexible enough that you can use it as a main project base or in addition to your current project based on Node.js.

Though Nuxt.js is full-stack and very pluggable. It lacks the an API to communicate data from and to the server. Also unlike Meteor, Nuxt still relies on a configuration file. 
Meteor's build tool and Pub/Sub API (or alternatively when used with Apollo) provides Vue with this API that you would normally have to integrate yourself greatly reducing the amount
of boilerplate code you would have to write.

<h3 id="integrating-vue-with-meteor">Integrating Vue With Meteor</h3>

To install Vue in Meteor 1.8, you should add it as an npm dependency:

```sh
meteor npm install --save vue
```

To support [Vue's Single File Components](https://vuejs.org/v2/guide/single-file-components.html) with the .vue file extensions, install the following Meteor package created by Vue Core developer [Akryum (Guillaume Chau)](https://github.com/meteor-vue/vue-meteor/tree/master/packages/vue-component).

```sh
meteor add akryum:vue-component
```

At time of writing, there is a known bug in the vue-component package which makes the app refresh endlessly. This has to do with the package's own hot reload system. You can however work around it by setting 
the `NO_HMR=1` env var.

You can now start writing .vue files in your app with the following format:

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

You can render the Vue component hierarchy to the DOM by using the below snippet in you client startup file:

```javascript
import App from './App.vue';

Meteor.startup(() => {
  new Vue({
    el: '#app',
    ...App,
  });
});
```

Of course you will need a base HTML document with a div that has the `app` id:

```html
<body>
  <div id="app"></div>
</body>
```

You will end up with at least 3 files: 

- App.vue (The root component of your app)
- main.js (Initializing the Vue app in Meteor startup)
- main.html (containing the body with the #app div)

