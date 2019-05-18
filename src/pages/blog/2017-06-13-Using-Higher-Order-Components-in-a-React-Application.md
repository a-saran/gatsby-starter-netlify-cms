---
templateKey: 'blog-post'
title: 'Using Higher Order Components in a React Application'
date: 2017-06-13
featuredpost: true
description: >-
  Using Higher Order Components in a React Application
tags:
  - Prasanna
author: Prasanna
---

If you have started using [ReactJS](https://codebrahma.com/react-js-development/) recently often you might have felt that you are performing repeated logic in many components. For example consider an app having:

- Infinite scroll in three different views with all - having different data.
You want to fetch new set of data when you scroll to the current limit.

If you ever thought “Is there a way by which I can simplify this such that I don’t need to rewrite the fetching logic again and again in all three views?”. Well Higher Order Component (HOC) provides solution to such a kind of problems.

HOC is a powerful tool based on which many libraries are getting developed. The objective of this article is to explain about how HOCs can help in simplifying and abstracting repeated logic in a React Applications.

Javascript in general can be treated as functional programming or object orient programming. HOCs are functional implementation of javascript. It has its similarities with the concept of Higher Order functions.

Functions that operate on other functions, either by taking them as arguments or by returning them, are called higher-order functions.

For example
```
function greaterThan(a) { // Higher order function
  return function(b) { 
    return b > a; 
  }
}
 ```
greaterThan(10)(11) will be true.

Also we can do
```
var greaterThan10 = greaterThan(10);
```
And anywhere we can use greaterThan10(11) [ Which will be true ]. This helps in abstraction the inner details thereby giving us ability to think about higher level.

Higher Order components are based on this. Given a function (Functional component) which takes a component and gives back another component.

> Function(component) => (component)

### 1. How it benefits in abstracting repeated logic?

A function (Functional component in React) which takes a component and gives back another component. We do this because we can do something inside the function which can add more power to the resulting component.

We can treat this as inheritance in React. How? We can treat the input functional component as a Parent and the resultant as the inherited components. So the exact benefits are:

1. Add / Update New props.
2. Filter out props.
3. Reuse Logic by maintaining an internal State.
4. Change the Rendering of component based on any logic.

### 2. How To write a Higher order component.

Functional components started from React V15. It can be created like
```
import React from 'react';
 
export const WrapperComponent(WrappedComponent) => {      // Wrapped Component is the parent component
  return class HigherOrderComponent extends React.Component {     // Returns a new component
 
    // Bring any reusable logic here.
    // For example Add a new prop to the existing props
    // or filter props based on the need
    render() {
      return (
        < WrappedComponent {...props} />    // Renders a new prop based on any operation
      );
    }
  }
}
 
// If we want to use this then we can do something like
// This enables reusable functionality
 
const FirstHOC = WrapperComponent(FirstComponent); 
const SecondHOC = WrappedComponent(SecondComponent);
```
### 3. Usage

Lets try to create a Higher order component for a particular use case. Lets consider we have an app which has many tables. Each table can have any number of these three functionality ( can have multiple)

1. Pagination – On clicking each page number, fetches new data correspondingly.
2. Sorting based on each column title in ascending or descending order.
3. Changing Number of rows per page (Limit).
The view would look something similar to this

![pasted image 0](/img/pasted-image-0.png)

The best way to structure is to have three separate logic for pagination, sorting and changing numbers per page. And whenever the parent component needs to conditionally render any of these then it can be done based on props.

```
import React from 'react';
 
export const TableWrapper(WrappedComponent) => {                           
  return class TableComponent extends React.Component {                   
 
    constructor(props) {
      super(props);
 
      this.state = {
        tableData: null; // Make some default fetching by which we can obtain this.
      };
 
      // Bind all the functions here
    }
 
    fetchMoreData(limit, offset, sortBasedOn) {
      // Fetches new Data and stores it in a state
      const tableData = fetch(endpoint, params) // Send params with limit, offset and sortBasedOn
      this.setState({
        tableData,
      })
    }
 
    onChangeLimit(changedLimit) {
      // when the limit changes fetch new data
      fetchMoreData(changedLimit, offset, sortBasedOn)
    }
 
    onChangeOffset(changedOffset) {
      // When the offset changes fetch new data
      fetchMoreData(limit, changedOffset, sortBasedOn);
    }
 
    onChangeSortPriority(currentSortBasedOn) {
      // When the sort priority changes fetch new data
      fetchMoreData(limit, changedOffset, currentSortBasedOn)
    }
    
    render() {
      return (
        

          (
            props.sortingRequired ? 
              < Sorting 
                {...sortingRelatedProps} 
                onChangeSortPriority={this.onChangeSortPriority}    // Callback for change in sort priority
              /> : null
          )
          (
            < WrappedComponent 
              {...props}
              {...this.state.tableData}                             // Render Table Data.
            />
          )                                        
          (
            props.changeNumberOfRowsRequired ? 
              < ChangeNumberOfRowsPerPage 
                {...numberOfRowsPerPageRelatedProps}
                onChangeLimit={this.onChangeLimit}                  // Callback for change in limit
              /> : null
          )
          (
            props.paginationRequired ? 
              < Pagination 
                {...paginationRelatedProps}
                onChangeOffset={this.onChangeOffset}                 // Callback for change in offset
              /> : null
          )
        
      );
    }
  }
}
```
 
For this to make work all you need is a presentational component which renders Table cells based on some data. Lets say``` < Table />``` is such a component then,
```
// A Table without any feature
const WrappedTable = TableWrapper(< Table />);

// Table with pagination
const WrappedTable = TableWrapper(< Table paginationRequired={true}/>);

// Table with all features
const WrappedTable = TableWrapper(
< Table
 paginationRequired={true} 
 changeNumberOfRowsRequired={true} 
 sortingRequired={true} 
/>);
```
Also we can see data fetching and ```this.state.tableData``` is completely abstracted inside of the HOC.

This structure can help us to write HigherOrder component at multiple level one over the other. For example the same can be achieved by writing three higher order components if we carefully interlink the abstraction. If we do that then we can have something like

```
const WrappedTable = PaginationWrapper(SortingWrapper(ChangeLimitWrapper(< Table />)));
```
But there are certain disadvantages in this method. This will create three React classes as supposed to one in the previous method. Also HOCs in common might have performance issues if there are too many props linking each functionality. One must use ShouldComponentUpdate lifecycle in order to avoid that.

 

But apart from that if we are careful enough we can have an architecture which can abstract logic that can be shared across other presentational components using HOC has great benefits.

Proceed to next part of this blog for a more [functional approach to Higher Order Components](https://codebrahma.com/functional-approach-higher-order-components-recompose/)