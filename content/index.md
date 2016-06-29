---
title: Introducción
order: 0
description: Ésta es la guía para usar Meteor, una plataforma full-stack de JavaScript para desarrollar aplicaciones web y móviles modernas. Meteor incluye un conjunto clave de tecnologías para construir aplicaciones reactivas, una herramienta de construcción, y un conjunto seleccionado de paquetes de Node.js y la comunidad general de JavaScript.
---

<h2 id="what-is-meteor">¿Qué es Meteor?</h2>

Meteor es una plataforma full-stack de JavaScript para desarrollar aplicaciones web y móviles modernas. Meteor incluye un conjunto clave de tecnologías para construir aplicaciones reactivas, una herramienta de construcción, y un conjunto selecto de paquetes de Node.js y la comunidad general de JavaScript.

- Meteor te permite desarrollar en **un solo lenguaje**, JavaScript, en todos los entornos: aplicación de servidor, navegador web, y dispositivos móviles.

- Meteor utiliza **data on the wire**, que quiere decir que el servidor envía datos, no HTML, y el cliente lo renderiza.

- Meteor **acepta al ecosistema**, entregándote las mejores partes de la comunidad extremadamente activa de JavaScript de una forma cuidadosa y considerada.

- Meteor provee **reactividad full stack**, permitiendo a la interfáz gráfica reflejar sin problemas el verdadero estado del mundo con mínimo esfuerzo de desarrollo.

<h2 id="what-is-it">¿Qué es la Guía de Meteor?</h2>

Éste es un conjunto de artículos explicando de forma general opiniones de mejores prácticas para desarrollo de aplicaciones utilizando la plataforma [Meteor](https://meteor.com). Nuestro objetivo es cubrir patrones que son comunes para el desarrollo de todas las aplicaciones web y móviles modernas, así que muchos conceptos documentados aquí no son necesariamente específicos de Meteor y podrían aplicarse a cualquier aplicación construida con enfoque en interfaces de usuario modernas e interactivas.

Nada en la guía de Meteor es *necesario* para construir una aplicación con Meteor---usted podría utilizar la plataforma en formas que contradigan los principios y patrones de la guía. Sin embargo, esta guía es un intento por documentar las mejores prácticas y convenciones de la comunidad, así que esperamos que la mayoría de la comunidad se beneficiará de adoptar las prácticas documentadas aquí.

Las APIs de la plataforma Meteor están disponibles en el [sitio de docs](https://docs.meteor.com), y puedes explorar los paquetes de la comunidad en la [atmosfera](https://atmospherejs.com).

<h3 id="audience">Audiencia objetivo</h3>

La guía está dirigida a desarrolladores intermedios que tienen alguna familiaridad con JavaScript, la plataforma Meteor, y desarrollo web en general. Si usted recien comenzó a usar Meteor, le recomendamos que empiece con el [tutorial oficial](https://www.meteor.com/tutorials/blaze/creating-an-app).

<h3 id="example-app">Aplicación de ejemplo</h3>

Muchos artículos se refieren a la aplicación de ejemplo de Todos (pendientes). Éste código está siendo desarrollado activamente a la par de la guía. Usted puede ver el código fuente más reciente de la aplicación, y reportar un problema o hacer sugerencias vía pull requests en su [repositorio de GitHub](https://github.com/meteor/todos).

<h2 id="guide-concepts">Desarrollo de la guía</h2>

<h3 id="contributing">Contribuyendo</h3>

El desarrollo continuo de la Guía de Meteor se encuentra **abierto** [en Github](https://github.com/meteor/guide). Recomendamos el uso pull requests e issues para discutir problemas que afecten y cambios que pudieran hacerse al contenido. Esperamos que manteniendo nuestro proceso abierto y honesto deje claro lo que planeamos incluir en la guía y qué cambios vendrán en futuras versiones de Meteor.

<h3 id="goals">Metas del proyecto</h3>

Las decisiones tomadas y prácticas explicadas en la guía deben ser necesariamente **sesgadas**. Ciertas prácticas serán resaltadas y otros enfoques válidos ignorados. Nuestro objetivo es alcanzar un consenso en la comunidad alrededor de decisiones importantes pero siempre habrá otras formas de resolver problemas mientras usted desarrolla su aplicación. Creemos que es importante saber cuál es la forma "estándar" de resolver un problema antes de explorar otras opciones. Si un enfoque alternativo prueba ser superior, entonces debería encontrar su lugar en una futura versión de la guía.

Una función importante de la guía es **dar forma al desarrollo futuro** de la plataforma Meteor. Al documentar las mejores prácticas, la guía arroja a la luz áreas de la plataforma que podrían ser mejores, más fáciles o con mejor desempeño, y por lo tanto serán usadas para guiar gran parte de las decisiones futuras de la plataforma.

De forma similar, omisiones en la plataforma destacadas por la guía con frecuencia pueden ser complementadas por **paquetes de la comunidad**; esperamos que si usted ve una oportunidad para mejorar el flujo de trabajo de Meteor al escribir un paquete, ¡que usted la tome! Si no está seguro de cuál es la mejor forma de diseñar su paquete, busque ayuda en los foros y comience una discusión.

<h3 id="future">Adiciones futuras</h3>

La guía de Meteor nunca estará terminada. Particularmente, será actualizada con nuevas características publicadas en cada versión de Meteor. La versión más reciente se enfoca en Meteor 1.2.x. Hay algunos artículos más que añadiremos en las semanas siguientes para coincidir con el lanzamiento de Meteor 1.3, que incluirá soporte para módulos de ECMAScript, integración directa con NPM, integración y pruebas unitarias mejoradas, y mejores características para móvil:

1. Estructura de la aplicación
2. Pruebas
3. Estilo de código y linting
4. Formularios y entradas de usuario
5. Aplicaciones móviles

<h2 id="learning-more">Otros recursos de Meteor</h2>

1. El lugar para empezar con Meteor es el [tutorial official](https://www.meteor.com/tutorials/blaze/creating-an-app).

2. [Stack Overflow](http://stackoverflow.com/questions/tagged/meteor) es el mejor lugar para hacer (¡y responder!) preguntas técnicas. Asegúrese de agregar la etiqueta de meteor a su pregunta.

3. Visite los [foros de discusión de Meteor](https://forums.meteor.com) para anunciar proyectos, obtener ayuda, hablar sobre la comunidad, o discutir cambios al núcleo del proyecto.

4. Los [docs de Meteor](https://docs.meteor.com) es el mejor lugar para encontrar la documentación de la API del núcleo de la plataforma.

5. La [Atmosfera](https://atmospherejs.com) es el repositorio de los paquetes de la comunidad diseñado especialmente para Meteor.

6. La sección de [proyectos](https://www.meteor.com/projects) del sitio de Meteor describe los proyectos que conforman la plataforma Meteor.
