---
templateKey: 'blog-post'
title: 'HOW TO USE IMMUTABLE.JS IN A REACT REDUX APPLICATION'
date: 2017-05-18
featuredpost: true
description: >-
  HOW TO USE IMMUTABLE.JS IN A REACT REDUX APPLICATION
tags:
  - immutable.js
author: Prasanna
---
ReactJS is one of the more popular Javascript libraries for developing front-end applications. Along with redux it gives good control in persisting the app state in the browser. State can be of anything among user interactions with the app or API data.

One of the key requirements of redux is that each reducer must be pure. That is given a state and an action it should always give a new state but not mutate the existing one.

This helps in:

__Time Traveling__ – Moving to the Previous states.
__Optimized re rendering__ of react components based on change in redux state. (Since the final state is different from initial based on both reference and value).
The straight forward way of changing redux store based on an action is using spread operators as described. Lets say to update the state’s ```activeSelected``` we can do something like
```
return {...state, { activeSelected: updateValue } }.
```
But this gets complex once the state goes bigger and when you need to do more operations inside each reducer.

There are certain libraries which helps in making this simpler (immutable js, mori, updeep…). One of the popular library among those is Immutable.js which was developed by facebook. The core of immutable JS is

> Data encapsulated in immutable JS is never mutated and always a new copy is returned.

It’s API is very rich and it provides optimised performance by structural sharing.

With immutable JS we can easily do the same task by:
 
```
return state.set(‘activeSelected’, updateValue);
 ```

The above code returns a new copy of state rather than returning a mutated one.

The Immuateble.js library has data structures like Map (for JS objects), List (for JS arrays), Set, ordered Map, Seq and so on.
Although there are really lot of good articles on what is ImmutableJS and why it should be used with redux, there is still room for confusion for a beginner when they start with Redux.

> Hire kickass ReactJS developers

Some of the common questions:
 

__1. Should I maintain my complete Redux state of the application in ImmutableJS ?__
If we are going to use ImmutableJS in our application, the best way to approach it would be to think of the entire store made of Immutable objects. This helps in:

- Using the API efficiently (due to structural sharing).
- Efficiency in memory.
- Lazy Seq to convert into sequencing where can do operations like map, reduce.. and bringing it back to regular Immutable object.
If you encounter any deep nested JS objects, you can use FromJS API to convert them. This will convert every object to Immutable Map and convert every List to Immutable Array.
```
Const deeplyNestedObject = // any deeply nested object
Let state = fromJS(deeplyNestedObject);
```
__2. If so, how will I maintain and modify them?__
The main advantage of converting entire state to ImmutableJS is to use their API which does every operation in an optimized manner. They offer a set of APIs to modify or update the immutable object. For example

Merging a state with another
```
const map1 = Map({ a: 1, b: 2, c: 3, d: 4 })
const map2 = Map({ c: 10, a: 20, t: 30 })
const obj = { d: 100, o: 200, g: 300 }
const map3 = map1.merge(map2, obj); // merges the given objects
// Map { a: 20, b: 2, c: 10, d: 100, t: 30, o: 200, g: 300 }
```
Updating a deep Object
```
const map1 = Map({ a: { b: 2 }})
Map1 = map1.setIn([‘a’,  ‘b’], 3) 
// Map { a: { b: 3 }}
```
There are other useful methods in their API which is useful for solving most of the tasks.

__3. How to do regular lodash type map, reduce and other operations?__
With Immutable JS we have option to convert it into Seq which Represents a sequence of values, but may not be backed by a concrete data structure. This helps us perform lodash type chain operations efficiently.
```
const { Seq } = require('immutable')
const oddSquares = Seq([ 1, 2, 3, 4, 5, 6, 7, 8 ])
  .filter(x => x % 2 !== 0) // Filters all odd numbers
  .map(x => x * x) // squares it. We can keep adding additional operations...
```
The resulting sequence can be converted to any data structure of our choice. This is greatly beneficial since the result is a immutable data structure which is completely new.

__4. Should I use it inside components too ?__
So you are done with redux change. Components which are subscribed to redux will get notifications about the change. If we are using ‘react-redux’ library to connect then the mapStateToProps function will look something like this:
```
mapStateToProps = (state) => {
  activeSelected: state.get(‘activeSelected’) // Returns a new activeSelected even if data is not changed.     
}
```
Since the component re renders based on comparing the props, if we are using the above mentioned way, then it will re render every time when redux store changes. Since immutable objects returns new reference each time (even if it has same data and remains unchanged during transition) this way will slow down the app due to unnecessary re renders. The best way to avoid this is by writing a HOC something like this:
```
import React from 'react';

import { reduce } from 'lodash';

import { Iterable } from 'immutable';

// Higher Order component to wrap the props
const ToJS = (WrappedComponent) => {
  return class ImmutableWrapper extends React.Component {

    constructor(props) {
      super(props);

      this.updateNewProps = this.updateNewProps.bind(this);
      this.newProps = this.updateNewProps(this.props);
    }

    // Function to check if immutable object. If so then convert
    updateNewProps(currentProps) { 
      const objecEntries = Object.entries(currentProps);
      return reduce(objecEntries, (newProps, entry) => {
        newProps[entry[0]] = Iterable.isIterable(entry[1]) ? entry[1].toJS() : entry[1]; // eslint-disable-line
        return newProps;
      }, {});
    }
    
    // Whenever it receives new props, pass in new props
    componentWillReceiveProps(nextProps) {
      this.newProps = this.updateNewProps(nextProps);
    }

    render() {
      return (
        <WrappedComponent
          {...this.newProps}
        />
      );
    }
  };
};

export default ToJS;
```
So that the container can be exported simply like ```ToJS(<Container />). This will ensure that the container will receive props only when there is a data change.```

__5. What is the advantage of using ImmutableJS over lodash or normal JS objects?__
The biggest advantage of using Immutable is efficiency. It achieves its efficiency through structural sharing. More efficient processing means faster apps; without the headache of maintaining a huge codebase.

You can read more about this here.

__6. Should we use immutable JS for our application?__
It really depends on how big or complex the state is. If we have a state which is just going to deal with API data or static data then it is preferable to not use it. If we are going to do lot of state changes with deep nested objects it is good to use it.