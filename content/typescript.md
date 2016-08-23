---
title: TypeScript
order: 42
description: Adding optional static typing to your Meteor app.
discourseTopicId: 28023
---


After reading this guide, you'll know:

1. What TypeScript is
2. How static typing can improve the quality of your code
3. How Typings can help you use 3rd-party libraries

<h2 id="introduction">Introduction</h2>

[TypeScript](https://www.typescriptlang.org/) is ES2015 with optional types.
 
Make sure you are familiar with the [new features introduced by ES2015](https://www.sitepoint.com/es2015-useful-parts-rundown-best-new-features/) before reading this.
 
Adding types to your code make it easier for your development environment, yourself, and your colleagues, to understand your code. This becomes increasingly important as your codebase grow, which is why TypeScript is very powerful for large apps. 

The file extensions are exactly what you would expect them to be. Just like JavaScript has .js (and .jsx for React), are the analogue TypeScript file extension .ts (and .tsx for React). 

If you already have a large codebase, and don't want to rename all the files it contain, don't despair - it will be explained in the configuration section how you in a simple manner can convert a single file at a time.

<h3 id="installing-typescript">Installing TypeScript</h3>

In order for Meteor to compiles the .ts and .tsx files  is it necessary to intall a TypeScript compiler. If you have the `ecmascript` package installed can this in Meteor be done as simple as:

```sh
meteor add barbatus:typescript
```
It is important to understand that TypeScript only is a tool you use while building your apps. When you are ready to run your code will Meteor compile everything down to JavaScript, where all the types are gone.

<h3 id="configuring-typescript">Configuring TypeScript</h3>

You have to configure the TypeScript compiler to make it work. You do this by creating a tsconfig.json in your projects root and setting a few of the many [compiler options](http://www.typescriptlang.org/docs/handbook/compiler-options.html).

The most important compiler options will now be explained while examining a tsconfig.json used for a typical Meteor and React app.
```
{
  "compilerOptions": {
    "target": "es5",
    "module": "commonjs",
    "moduleResolution": "node",
    "sourceMap": true,
    "allowJs": true,
    "jsx": "react"
  },
  "filesGlob": [
    "client/**/*.ts",
    "client/**/*.tsx",
    "server/**/*.ts",
    "server/**/*.tsx",
    "typings/**/*.d.ts"
  ],
  "exclude": [
    "node_modules",
    ".meteor"
  ]
}
```
The **target** option tells us that this project compile TypeScript down to ES5, thereby removing all let/const, lambda functions, spread operators and other features shiped with ES2015.

The **module** option let you choose which module loeader you want to use. This project convert the modern ES2015 syntax (import / export) to the older CommonJS syntax (require / module.export).

The **moduleResolution** option tell the compiler which algorithm it should use when searching for modules. This app mimic the strategy used by Node and io.js, which is the default setting as of writing.

The **sourceMap** option can help you debug your app. If you run your app, and an error occur, will the line number displayed correspond to the line in the .js file where the error happened. The issue is that this line number doesn't correspond to the original TypeScript file where you can fix the error. The solution is to use a source map (.js.map) that in combination with a .js will give you the line number of the original .ts file.

The **allowJs** option allow incremental adoption of TypeScript. If set to true can you import JavaScript files from TypeScript files. This feature is extremely practical if you  have an existing codebase that you want to migrate, or just want to try out TypeScript.

The **jsx** option allow you to use JSX inside .tsx files. If set to  "react" will the .tsx files be compiled directly to .js. If you prefer converting the files from .jsx to .js another way can you set the option to "preserve".

Now that all the compiler options are covered, let's look at **filesGlob**. This is where you specify which files you would like to be compiled. By setting filesGlob as shown above will the compiler search for all .ts and .tsx files in either the /server or /client directory. The filesGlob can be configured using regular expressions. You will get back to the path "typings/**/*.d.ts" in the Typings section below.

Any Node project contain several node modules. They are for obvious reasons placed in a /node_modules directory, which you don't want the compiler to waste time looking through. The same is the case for the .meteor folder which contain all the atmosphere packages and a local database that you also don't wanna waste time on looking through. The solution to this is **exclude** both "node_modules" and ".meteor" as shown.

<h2 id="typings-type-definition-manager">Typings - type definition manager</h2>

There are a wealth of JavaScript libraries out there and you can use all of them with TypeScript. You can even add autocompletion to most of them for improved tooling. The easiest way to explain how this is done is through an example.

If you have installed React through npm will you in JavaScript be able to import and log it to the console with a syntax similar to:

```js
import * as React from 'react'
console.log(React);
```

This is still true in TypeScript, but it will complain and say that it can't find 'react'. The reason for this is not that react is missing, and in fact will React still be logged just fine if you run the app. 

The error instead appear because TypeScript is looking for type definitions for React, and it is not able to find it. Think of type definition as a file that describes a JavaScript library. Type deinitions for a library contain names and return types for all functions including which parameters the functions take and which types they are. 

<h3 id="installing-type-definitions">Installing type definitions</h3>

If TypeScript has a type definition available can it provide you with a first class intellisence, so let's install one for React.

The tool use to install type definitions is called [Typings](https://github.com/typings/typings). You can install it with:

```sh
npm install typings -g
```

When it is installed can you in the terminal search for type definitions for a library with:

```sh
typings search react
```

A lot of definitions should appear, showing either "npm" or "dt" as their source. "dt" stands for [Definitely Typed](https://github.com/DefinitelyTyped/DefinitelyTyped) which is the old source for type definitions before they started moving to "npm". A piece of advice when choosing between multiple possible type definitions - check which of them that was updated most recently. 

The Typings cli works similar to npm, so you can install the react type defitition as:

```sh
typings install react --save
```

This will by default choose the react type definition with "npm" as its source. Should you prefer the type definition with "dt" as its source would you have to write:

```sh
typings install dt~react --save --global
```

Nice people have also created type definitions for Meteor that you can install with:
```sh
typings install env~meteor --save --global
```

<h3 id="load-the-type-definitions-into-typescript">Load the type definitions into TypeScript</h3>

One thing is to download the type defnintions to a /typings directory, something else is to make TypeScript aware that they exist.

This is done by adding the path "typings/\*\*/\*.d.ts" to the **filesGlob** in the tsconfig.json as shown in the previous *Configuring TypeScript* section.

When you have done this, and probably restarted your IDE, should TypeScript be able to pick up the typings and stop complaining about that it can't find 'react'. 

Try pressing CMD (macOS) or Control (windows) and click 'react' in your import statement. This should open the respective type definition.

<h2 id="using-typescript">Using TypeScript</h2>

Now that you have TypeScript and Typings installed, lets look at to at how to use it. For a general introduction to TypeScript check the official [video on typescriptlang.org](http://www.typescriptlang.org/). What this video doesn't explain is how to use TypeScript with React, so let's at an example of how we can use TypeScript when creating components.

<h3 id="react-component-with-internal-state">React component with internal state</h3>

The core below show the creation of a React Component that represent a square. The square has a color property, and it's internal state keep track of whether it is hovered. The code is a combination of ES2015 and TypeScript, so it is easy to see which part TypeScript add.

```js
// TypeScript allow the creation of interfaces, which we'll use for the properties and state variables. Notice the '?' for the color property - it means that this property is optional, and you should remember to set a default value for that one.
  
/*TS*/  	interface IProps {
/*TS*/    		color?: string;
/*TS*/  	}

/*TS*/		interface IState {
/*TS*/    		hovered: boolean;
/*TS*/ 		}


// The interfaces can be added to the component as shown here:

//ES2015 	class ComponentWithState extends React.Component {
/*TS*/  	class ComponentWithState extends React.Component<IProps, IState> {
  
              static defaultProps = {
                color: '#aff',
              };

              constructor(props: IProps) {
                super(props);

                this.state = {
                  hovered: false
                };

                this.onMouseOver = this.onMouseOver.bind(this);
                this.onMouseOut = this.onMouseOut.bind(this);
              }

              onMouseOver() {
                this.setState({ hovered: true });
              }

              onMouseOut() {
                this.setState({ hovered: false });
              }

              render() {
                return (
                  <div onMouseOver={this.onMouseOver} onMouseOut={this.onMouseOut}>
                    <div style={{ backgroundColor: this.props.color }}>
                      State: { JSON.stringify(this.state)}
                    </div>
                  </div>
                )
              }
            }
            
            
// This is the common way to set property types in ES2015, using the React typesystem, before TypeScript interfaces became an option.

//ES2015	ComponentWithState.propTypes = {
//ES2015    color: React.PropTypes.string
//ES2015	};

```

To see a running example of these components can you visit the codepen below:

[Codepen for ES2015](https://codepen.io/birkskyum-1471370946/pen/qNvAdP)

[Codepen for TypeScript](https://codepen.io/birkskyum-1471370946/pen/qNvAdP)

<h3 id="react-component-as-a-pure-function">React component as a pure function</h3>

If your component doesn't have any internal state can it be written in simpler way as a pure function. An example of this is shown below, and it is the same for both ES2015 and TypeScript.

```js
const ComponentWithoutState = (props) => {
  return (
    <div>
      <div style={{ backgroundColor: props.color }}>
        No state
      </div>
    </div>
  );
};
```

This component is also shown in the codepens refered above.

<h2 id="other-advantages-of-typescript">Other advantages of TypeScript</h2>

This guide only mention a small subset of all the features of TypeScript. Other features include 

- Type guards
- Union types, intersection types, shape-based interfaces
- Control flow based type analysis
- The never type
- Abstract properties
- .. and many more.

The tooling of TypeScript is also very good at this point, featuring [integration with most major editors](https://github.com/Microsoft/TypeScript/wiki/TypeScript-Editor-Support). Especially the debugging tools in Visual Studio Code are worth checking out.

<h3 id="linting-with-tslint">Linting with TSLint</h3>

TSLint is the TypeScript version of ESLint, and it can help you remember to keep your code clean. 

It can easily be installed as a Visual Studio Code plugin, and it is configured through a tslint.json file in the project root. More information can be found [here](https://github.com/palantir/tslint). 
