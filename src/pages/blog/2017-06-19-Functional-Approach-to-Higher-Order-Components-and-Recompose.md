---
templateKey: 'blog-post'
title: 'Functional Approach to Higher Order Components and Recompose'
date: 2017-08-19
featuredpost: true
description: >-
  Functional Approach to Higher Order Components and Recompose
tags:
  - Prasanna
---

This is a continuation of our previous post where we talked about how Higher Order Components (HOC) help abstraction of logic. This in turn helps simplifying a React application.

Now let’s say we have a React Component on which we need to do certain tasks:

1. Omit certain props which parent component passes. In our case we will omit ```isDone``` prop.
2. Conditionally check if we need to still show spinner based on certain props. In our case, if we have ```isLoading``` prop set to ```true``` then show a Component which renders a Preloader. That is show a preloader if ```isLoading``` prop is ```true``` and render the component if ```isLoading``` prop is ```false```.
3. Attach an event handler which does some preprocessing of data and then logs (on console) the data on clicking the button. (Since we cannot have event handlers in functional components, I particularly want to discuss how we can do this)

For this the usual way would be writing a HOC for each task separately and combine by using one on top of each other.

Now let’s look at an alternative way for achieving the same using functional programming. We described Higher Order Functions in our previous post as:

__Functions that operate on other functions, either by taking them as arguments or by returning them, are called higher-order functions.__

If we adopt functional programming, we need to think about composing existing functions as much as possible in order to create new functions. Currying is one major technique used to achieve this.

Currying is a method of constructing functions which facilitates the function to have all arguments passed to it returning a result or have partial arguments to be passed resulting a new function which expects remaining arguments. For example:
```
const fullName = firstName => lastName => {
  return firstName+lastName;
};
```
Now if we use ```fullName(‘Code’)``` it will return another function which expects a ```lastName```. Its first name will be replaced as Code.
```
const partialName = fullName(‘Code’);
```
When we write
```
const finalName = partialName(‘brahma’); // Returns ‘Codebrahma’
```
```finalName``` will be assigned with ‘Codebrahma’. This is a useful technique since it uses to create and compose multiple functions.

So considering the previously discussed task, first we will create a ```conversionFunction``` which can omit the __‘isDone’__ prop.
```
// Import React
import React from 'react';

// Import omit from 'lodash'
import omit from 'lodash/omit';

// A simple dummy component for which we are omitting
const DummyComponent = props => (
  <div>
    A simple dummy component
  </div>
);

// React.createFactory is considered legacy
// We are writing a custom function which takes a component, its props and returns the new component

// Why a function here ? So that we can modify either the props or component

const createFactory = Component => {
  return props => React.createElement(
    Component,
    props,
  );
}

// Read this as 'conversionFunction' takes 'anyFunction' and a React component and returns a new function
// This function returns a new React component based on currying

const conversionFunction = anyFunction => Component => {
  const factory = createFactory(Component);
  const mapProps = props => factory(anyFunction(props));
  return mapProps;
};

// This line does omits based on currying
export default conversionFunction(props => omit(props, ['isDone']))(DummyComponent);
```
Now we can see that without adding an additional React Class we are able to write a HOC which omits the mentioned prop.

Now the second task is to show a Preloader if we have ```isLoading``` prop set to true. To achieve this the we need to change the conversion function like this.
```
const conversionFunction = anyFunction => Component => {
  const factory = createFactory(Component);
  // Render a preloader if you have such a props
  const mapProps = props => (props.isLoading ? <Preloader /> : factory(anyFunction(props)));
  return mapProps;
};
```
Now if we pass ```isLoading``` prop as true, it will always render a preloader. This gives control from the parent component to decide when to load the preloader and when to load the actual component

The third task is to attach an event handler to the functional component and console logging some data on clicking a button using that even handler. This is an extension to the above concept except that we will return one more React Component squashed inside.
```
// Import React
import React from 'react';

// Import omit from 'lodash'
import omit from 'lodash/omit';

import merge from 'lodash/merge';

const Preloader = props => (
  <div>
    Loading...
  </div>
);

const createFactory = Component => {
  return props => {
    return React.createElement(
      Component,
      props,
    );
  }
}

// Just include a button
const DummyComponent = props => (
  <div>
  A simple dummy component
  <button onClick={() => { props.handleButtonClick(1) }}> Click to Log </button>
  </div>
);

// Expects { handlerName, handlerFunction } and returns a factory with event handler in props
// see the render function.
const conversionFunction = ({ handlerName, handlerFunction }) => anyFunction => Component => {
  const factory = createFactory(Component);
  return class ComponentWithHandler extends React.Component {
    constructor(props) {
      super(props);
    }
    render() {
      return this.props.isLoading ? <Preloader /> : factory({
        ...this.props,
        [handlerName]: handlerFunction,
      });
    }
  }
}

// We can change this handler as we like
const sampleHandlerFunction = {
  handlerName: 'handleButtonClick',
  handlerFunction: (data) => {
    console.log(' logging data ', data);
  }
};

export default conversionFunction(sampleHandlerFunction)(props => omit(props, ['isDone']))(DummyComponent);
```
We can see that we have used functional programming to achieve all the tasks mentioned above. ```conversionFunction``` is now a curried function which will work for any component that can do all the target tasks. Also we can ignore certain tasks as per the requirement.

The whole idea of doing this is functionality abstraction which results in components which depends only on props. The advantage of having more and more stateless components are

- Clean code while avoiding bloated components.
- Easy to understand and easy to test.
- Better performance since there are no lifecycle methods.
 

Recompose [(GitHub link)](https://github.com/acdlite/recompose) is a library which was developed based on the idea. It has list of utility functions which they claim as “Recompose is a React utility belt for function components and higher-order components. Think of it like lodash for React.” They have various cool functionalities abstracted through HOCs. From providing state through props for functional components to adding lifecycle events to functional components they have added functionalities which we feel is required most of the times while developing a React app.

Using these methods we can write really cool abstracting logic which we are regularly repeating among components.