# React

1. Introduction to React
  1. Background: developed by Facebook, used widely in production, vibrant ecosystem
  2. Installing and using React in Meteor 1.3
  3. A simple example

2. Meteor and React
  1. JSX and the Meteor build tool (link to build tool article)
  2. Using 3rd party react libs in apps and packages

3. Getting Meteor Data into React Components
  1. The `ReactMeteorData` mixin
  2. The `ReactMeteorDataContainer` higher order component
  3. Containers vs. presentational components [call out to UX guide]
  4. Optimizing re-renders with `shouldComponentUpdate` and [pre-bindings](https://medium.com/@esamatti/react-js-pure-render-performance-anti-pattern-fb88c101332f)

4. Routing with React
  1. FlowRouter
    1. Using mount
  2. `react-router`
    1. Basic demo, "spirit" of our routing guide

5. Using Redux to Supplement Meteor
  1. Advantages of Redux (rich dev/debug tools, time travel, dead simple undo/redo)
  2. ???
  3. Profit

6. Performance Testing
  1. React Perf Tools
    1. Installation and Setup
    2. Measuring Wasted Re-renders
