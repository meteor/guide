---
title: Apollo: using GraphQL with Meteor
order: 14
description: How and where to load data in your Meteor app using publications and subscriptions.
---

After reading this article, you'll know:

1. What Apollo and GraphQL are and why you'd use them
1. How to integrate a GraphQL API server into your app
1. How to access GraphQL data from your components
1. How to make GraphQL queries and mutations (writes) against your Mongo database.

# Outline

## What is Apollo?

### Introduction

 - In the [data loading] article, we saw how data traditionally has been moved around in Meteor
 - Although this is great for Mongo backed, realtime document based data, it has some limitations, which we saw in the advanced section.
 - When extending Meteor's data model to other databases, it becomes useful to *abstract* the data the client sees from the actual database (Mongo is good as it's document-based, SQL less so. Joining across data stores, impossible).
 - So the next generation solution involves a client-side data query language: GraphQL

### GraphQL basics

 - GraphQL is a query language first specified by Facebook for writing graph-based, relational client-initiated queries.
 - Read the [Apollo GraphQL guide] for all the details
 - In GraphQL, rather than writing a single publication (server side) and accessing it with the client, you write a "root query" with a resolver on the server and they you configure the data you want to return from the query (including arbitrary nesting of related documents) on the client.
 - The data format is completely client driven and doesn't need to map to the database in any way.
 - This allows you to access arbitrary databases easily and even run queries that map over various data sources.

 - A GraphQL query looks like this: and the result looks like that:

### When should I use GraphQL/Apollo in Meteor?

 - The chief reason to use GraphQL to access data in a Meteor app is to access data that isn't well suited for a publication:
   - Relational data
   - Data from non-MongoDB databases
   - Data from RESTful endpoints
   - Data that should not be reactive.

## Getting Started with Apollo

### Installation

 - Follow these steps to install apollo into your Meteor app:
 - The packages do the follow things + link to docs

### Writing a Schema

 - Switch publications from Todos over, referencing publish-composite solution.
 - Talk about various types in the schema

### Initialization

 - To get the schema resolving in the Todos app, we wire it into Apollo Server like so:
 - Note that this server uses Meteor's MongoDB integration, however there is nothing Mongo specific in the setup (beyond Meteor providing a MongoDB API). You could easily use a different database after installing a driver from npm.

### Using GraphiQL

 - Now our GraphQL server is running, we can access it with GraphiQL, a GraphQL explorer

## Using Apollo on the client side

 - The easiest way to connect to a GraphQL server on the client side is using [Apollo Client], a full-featured GraphQL client.
 - You can read about features of and patterns to use Apollo Client in it's [documentation]
 - We'll interact with Apollo Client principally via the view layer intergrations below, but the way we initialize it is common:

### Basic Blaze example

 - To use Apollo client in Blaze, we need to install the apollo-client-blaze meteor package:

 - This package adds a [Tracker-based] API to our Apollo queries

 - It also adds some new Blaze APIs for wiring in Apollo queries, similar to `template.subscribe`: `template.query`
  - Talk about polling / force fetch
  - Link to docs
  - Note that like `template.subscribe`, it automatically unsubscribes to queries for us

 - We call mutations using the `template.mutate` API.

### Basic React example

 - The pattern to connect to Apollo Client from React is conceptually identical to the Blaze implementation above.
 - Install the npm package `react-apollo`
 - Write a container component like so

### Basic Angular example

 - Investigate..

## Apollo and MongoDB

 - Another note about choices of DBs and RESTful APIs.
 - Note that at this point Meteor's user system still uses Mongo behind the scenes, so if you want to use the `accounts-` packages, you will need a MongoDB server

### Writing Mongo-based resolvers

 - We can use Meteor's standard `Mongo.find/One()` calls
 - We can also use `context.userId` for authentication, transparently

### Writing Mongo-based mutations

 - Here's an example:
 - A note about bindEnvironment?
 - What to return from mutations?

# Migration log

Going to add notes here of how the migration of Todos to Apollo went in case it ends up being interesting, in the style of Sashko's dev log.

1. Blindly followed instructions from http://docs.apollostack.com/apollo-client/meteor.html to install apollo + meteor package
  1. Added `graphql` as a dependency too
  2. Also decided to update to `1.4-beta.14`
  3. Ran into some version issues, cloned the latest `apollostack/meteor-integration` repo as a submodule

2. Added `imports/api/apollo-server.js` + accompanying `resolvers.js` and `schema.js`
  1. What should my schema look like? Surely the existing simple-schema schema should give me some hints.
    1. Copied the schemas from `lists/lists.js` and `todos/todo.js` into `schema.js` -- would it make more sense to import the sub-schemas?
    2. Also copied the publications in from `lists|todos/server/publications` to inspire the root queries
  2. Ended up exporting the GraphQL schema from the same file as the SimpleSchema. It's clear they should share code somehow.
    1. Not sure if the Todo should have a `list` or `listId` field. Hopefully it'll become clear.
    2. Switched the `userId` field on Lists to be `private`, which better reflects the client-side nature of the field (the client doesn't care who it belongs to, just if it's the current user).
    3. Added a `todos` field to the List.
  3. Wrote some simple resolvers in `resolvers.js` to see how it would go
    1. Why do the docs use `async/await` when Meteor's API doesn't return promises?
    2. Apollo Server docs are pretty hard to use here. You can get what you need if you follow the tutorial but this isn't a good way for reference.

https://github.com/meteor/todos/commit/f7f3713e267932b4e9133704da7289152aab0d3f
