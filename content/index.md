---
title: Einleitung
order: 0
description: Das ist ein Leitfaden für Meteor, ein Full-stack JavaScript-Framework mit dem man moderne Anwendungen für das Internet und mobile Endgeräte entwickeln kann. Meteor beinhaltet einen Schlüsselsatz an Technologien, um mit dem Server verbundene, sog. reaktive Anwendungen zu entwickeln, ein Build-Tool, sowie ausgewählte Pakete der Node.js und JavaScript-Community.
---

<h2 id="what-is-meteor">Was ist Meteor?</h2>

Meteor ist eine Full-stack JavaScript-Platform zum Entwickeln moderner Anwendungen für das Internet und für mobile Endgeräte. Meteor beinhaltet einen Schlüsselsatz an Technologien, um mit dem Server verbundene, sog. reaktive Anwendungen zu entwickeln, ein Build-Tool, sowie ausgewählte Pakete der Node.js und JavaScript-Community.

- Meteor ermöglicht dir in **einer Sprache**, JavaScript, zu entwickeln, und das für jede Umgebung: den Server, den Browser, und mobile Entgeräte.

- Meteor verwendet einen **laufenden Datenstrom**. Der Server sendet kein HTML, sondern Daten, die direkt vom Client gerendert werden.

- Meteor **umarmt das Ökosystem**. Es stellt dir auf gezielte und überlegte Art und Weise die besten Bausteine der enorm aktiven JavaScript-Community bereit.

- Meteor liefert dir **Full-stack Reaktivität**. Das erlaubt deinem UI, nahtlos den wahren Stand deiner Daten mit minimalem Entwicklungsaufwand abzubilden.

<h2 id="what-is-it">Was ist der Meteor Guide?</h2>

Der Leitfaden ist eine Reihe von Artikeln, der Meinungen und Best-practices zur Entwicklung von Anwendungen mit [Meteor](https://meteor.com) umreißen soll. Wir streben es an, übliche Vorgänge und Muster für die Entwicklung moderner Web- und Mobilanwendungen zu behandeln. Viele der hier aufgezeigten Konzepte sind also nicht zwingend spezifisch für Meteor und können auch auf andere Anwendungen mit modernen, interaktiven User Interfaces bezogen werden.

Nichts in diesem Guide ist *erforderlich* um Meteor-Anwendungen zu bauen, du kannst die Platform auch auf eine Art und Weise nutzen, die den Prinzipien und Mustern dieses Leitfaden widersprechen. Allerdings ist dieser Guide ein Versuch, Best-practices und Gepflogenheiten der Meteor Community zu dokumentieren. Deshalb hoffen wir, das der Großteil der Nutzer durch Anwendung der hier beschriebenen Vorgenhensweisen profitieren wird.

Die APIs der Meteor-Platform findest du in der [Dokumentation](https://docs.meteor.com), die Packages der Community kannst du auf [Atmosphere](https://atmospherejs.com) durchstöbern.

<h3 id="audience">Target audience</h3>

The guide is targeted towards intermediate developers that have some familiarity with JavaScript, the Meteor platform, and web development in general. If you are just getting started with Meteor, we recommend starting with the [official tutorial](https://www.meteor.com/tutorials/blaze/creating-an-app).

<h3 id="example-app">Example app</h3>

Many articles reference the Todos example application. This code is being actively developed alongside the guide. You can see the latest source code for the app, and file issues or make suggestions via pull request at its [GitHub repository](https://github.com/meteor/todos).

<h2 id="guide-concepts">Guide development</h2>

<h3 id="contributing">Contributing</h3>

Ongoing Meteor Guide development takes place **in the open** [on GitHub](https://github.com/meteor/guide). We encourage pull requests and issues to discuss problems with and changes that could be made to the content. We hope that keeping our process open and honest will make it clear what we plan to include in the guide and what changes will be coming in future Meteor versions.

<h3 id="goals">Goals of the project</h3>

The decisions made and practices outlined in the guide must necessarily be **opinionated**. Certain best practices will be highlighted and other valid approaches ignored. We aim to reach community consensus around major decisions but there will always be other ways to solve problems when developing your application. We believe it's important to know what the "standard" way to solve a problem is before branching out to other options. If an alternate approach proves itself superior, then it should make its way into a future version of the guide.

An important function of the guide is to **shape future development** in the Meteor platform. By documenting best practices, the guide shines a spotlight on areas of the platform that could be better, easier, or more performant, and thus will be used to focus a lot of future platform choices.

Similarly, gaps in the platform highlighted by the guide can often be plugged by **community packages**; we hope that if you see an opportunity to improve the Meteor workflow by writing a package, that you take it! If you're not sure how best to design or architect your package, reach out on the forums and start a discussion.

<h3 id="future">Future additions</h3>

The Meteor Guide will never be done. In particular, it will be updated with new features released in every Meteor version. The current release targets Meteor 1.2.x. There are a few more articles we will add in the coming weeks to align with the Meteor 1.3 release, which will bring ECMAScript module support, seamless NPM integration, improved integration and unit testing, and better mobile features:

1. Application structure
2. Testing
3. Code style and linting
4. Forms and user input
5. Mobile apps

<h2 id="learning-more">Other Meteor resources</h2>

1. The place to get started with Meteor is the [official tutorial](https://www.meteor.com/tutorials/blaze/creating-an-app).

2. [Stack Overflow](http://stackoverflow.com/questions/tagged/meteor) is the best place to ask (and answer!) technical questions. Be sure to add the meteor tag to your question.

3. Visit the [Meteor discussion forums](https://forums.meteor.com) to announce projects, get help, talk about the community, or discuss changes to core.

4. The [Meteor docs](https://docs.meteor.com) is the best place to find the core API documentation of the platform.

5. [Atmosphere](https://atmospherejs.com) is the repository of community packages designed especially for Meteor.

6. The [projects](https://www.meteor.com/projects) section of the Meteor website describes the projects that make up the Meteor platform.
