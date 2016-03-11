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

<h3 id="audience">Zielgruppe</h3>

Dieser Guide soll fortgeschrittene Entwickler ansprechen, die bereits Erfahrungen mit Javascript, der Meteor-Platform und Webentwicklung im Allgemeinen haben. Wenn du gerade deine ersten Schritte mit Meteor unternimmst, empfehlen wir das [offizielle Tutorial](https://www.meteor.com/tutorials/blaze/creating-an-app).

<h3 id="example-app">Beispielapp</h3>

Viele Artikel verweisen auf die Todos Beispielapp. Der Code für diese Anwendung wird neben dem Guide aktiv entwickelt. Du kannst dir den aktuellsten Quelltext für die App ansehen, Issues einreichen oder Vorschläge durch Pull Requests im [GitHub Repository](https://github.com/meteor/todos) machen.

<h2 id="guide-concepts">Leitfadenentwicklung</h2>

<h3 id="contributing">Beitragen</h3>

Die fortlaufende Entwicklung des Guides findet **in der Öffentlichkeit** [auf GitHub](https://github.com/meteor/guide) statt. Wir ermutigen dich, mithilfe von Pull Requests und Issues über Probleme und Änderungen des Inhalts zu diskutieren. Wir hoffen, dass durch diesen offenen und ehrlichen Prozess erkennbar wird, was wir planen, in diesen Guide zu integrieren und welche Veränderungen in den kommenden Versionen von Meteor zu erwarten sind.

<h3 id="goals">Ziele des Projekts</h3>

Die Entscheidungen des Guides müssen zwingenderweise einer **Meinung** entsprechen. Wir werden bestimmte Best-practices  hervorheben, während wir andere, berechtigte Herangehensweisen ignorieren werden. Wir streben nach Übereinstimmung der Community-Meinungen rund um bedeutende Entscheidungen. Es wird aber stets möglich sein, Probleme während der Entwicklung einer App durch Alternativwege zu lösen. Wir halten es für wichtig, erst den "Standardweg" zu kennen, bevor wir andere Optionen in Betracht ziehen können. Sollte sich eine Alternative als überlegen herausstellen, sollte sie Einzug in den Meteor Guide finden.

Eine wichtige Funktion des Leitfadens ist es, die **zukünftige Entwicklung** der Plattform **zu formen**. Durch die Dokumentation von Best-practices rückt der Guide den Fokus auf Bereiche der Plattform, die besser, einfacher und leistungsfähiger sein könnten. Dadurch bündelt er in Zukunft die Wahlmöglichkeiten dieser.

Gleichermaßen können die hervorgehobenen Differenzen durch **Community Packages** behoben werden. Wir hoffen, dass wenn du die Gelegenheit siehst, den Meteor-Workflow durch das Schreiben eines Packages zu verbessern, diese auch nutzt! Wenn du dir nicht sicher bist, wie du dein Package am besten designen oder aufbauen sollst, frage im Forum nach und starte eine Diskussion.

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
