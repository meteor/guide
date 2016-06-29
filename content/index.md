---
title: Introducción
order: 0
description: Ésta es la guía para usar Meteor, una plataforma full-stack de JavaScript para desarrollar aplicaciones web y móviles modernas. Meteor incluye un conjunto clave de tecnologías para construir aplicaciones reactivas, una herramienta de construcción, y un conjunto seleccionado de paquetes de Node.js y la comunidad general de JavaScript.
---

<h2 id="what-is-meteor">¿Qué es Meteor?</h2>

Meteor es una plataforma full-stack de JavaScript para desarrollar aplicaciones web y móviles modernas. Meteor incluye un conjunto clave de tecnologías para construir aplicaciones reactivas, una herramienta de construcción, y un conjunto seleccionado de paquetes de Node.js y la comunidad JavaScript general.

- Meteor te permite desarrollar en **un solo lenguaje**, JavaScript, en todos los entornos: aplicación de servidor, navegador web, y dispositivos móviles.

- Meteor utiliza **data on the wire**, que quiere decir que el servidor envía datos, no HTML, y el cliente lo renderiza.

- Meteor **acepta al ecosistema**, entregándote las mejores partes de la comunidad extremadamente activa de JavaScript de una forma cuidadosa y considerada.

- Meteor provee **reactividad full stack**, permitiendo a la interfáz gráfica reflejar sin problemas el verdadero estado del mundo con mínimo esfuerzo de desarrollo.

<h2 id="what-is-it">¿Qué es la Guía de Meteor?</h2>

Éste es un conjunto de artículos explicando de forma general opiniones de mejores prácticas para desarrollo de aplicaciones utilizando la plataforma [Meteor](https://meteor.com). Nuestro objetivo es cubrir patrones que son comunes para el desarrollo de todas las aplicaciones web y móviles modernas, así que muchos conceptos documentados aquí no son necesariamente específicos de Meteor y podrían aplicarse a cualquier aplicación construida con enfoque en interfaces de usuario modernas e interactivas.

Nada en la guía de Meteor es *necesario* para construir una aplicación con Meteor---podrías utilizar la plataforma en formas que contradigan los principios y patrones de la guía. Sin embargo, esta guía es un intento por documentar las mejores prácticas y convenciones de la comunidad, así que esperamos que la mayoría de la comunidad se beneficiará de adoptar las prácticas documentadas aquí.

Las APIs de la plataforma Meteor están disponibles en el [sitio de la documentación](https://docs.meteor.com), y puedes explorar los paquetes de la comunidad en [atmosphere](https://atmospherejs.com).

<h3 id="audience">Audiencia objetivo</h3>

La guía está dirigida a desarrolladores intermedios que tienen alguna familiaridad con JavaScript, la plataforma Meteor, y desarrollo web en general. Si usted recien comenzó a usar Meteor, le recomendamos que empiece con el [tutorial oficial](https://www.meteor.com/tutorials/blaze/creating-an-app).

<h3 id="example-app">Aplicación de ejemplo</h3>

Muchos artículos se refieren a la aplicación de ejemplo de Todos (pendientes). Éste código está siendo desarrollado activamente a la par de la guía. Usted puede ver el código fuente más reciente de la aplicación, y reportar un problema o hacer sugerencias vía pull requests en su [repositorio de GitHub](https://github.com/meteor/todos).

<h2 id="guide-concepts">Desarrollo de la guía</h2>

<h3 id="contributing">Contribuyendo</h3>

El desarrollo continuo de la Guía de Meteor se encuentra **abierto** [en Github](https://github.com/meteor/guide). Recomendamos pull requests e issues para discutir problemas que afecten y cambios que pudieran hacerse al contenido. Esperamos que manteniendo nuestro proceso abierto y honesto deje claro lo que planeamos incluir en la guía y qué cambios vendrán en futuras versiones de Meteor.

<h3 id="goals">Metas del proyecto</h3>

Las decisiones tomadas y prácticas explicadas en la guía deben ser necesariamente **sesgadas**. Ciertas prácticas serán resaltadas y otros enfoques válidos ignorados. Nuestro objetivo es alcanzar un consenso en la comunidad alrededor de decisiones importantes pero siempre habrá otras formas de resolver problemas mientras usted desarrolla su aplicación. Creemos que es importante saber cuál es la forma "estándar" de resolver un problema antes de explorar otras opciones. Si un enfoque alternativo prueba ser superior, entonces debería encontrar su lugar en una futura versión de la guía.

Una función importante de la guía es **dar forma al desarrollo futuro** de la plataforma Meteor. Al documentar las mejores prácticas, la guía arroja a la luz áreas de la plataforma que podrían ser mejores, más fáciles o con mejor desempeño, y por lo tanto serán usadas para guiar gran parte de las decisiones futuras de la plataforma.

De forma similar, omisiones en la plataforma destacadas por la guía con frecuencia pueden ser complementadas por **paquetes de la comunidad**; esperamos que si usted ve una oportunidad para mejorar el flujo de trabajo de Meteor al escribir un paquete, ¡que usted la tome! Si no está seguro de cuál es la mejor forma de diseñar su paquete, busque ayuda en los foros y comience una discusión.

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
