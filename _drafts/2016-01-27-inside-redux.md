---
layout: post
title: Part 1: Inside Redux - Factory functions vs classes 
date: 2016-01-27
categories:
- React
tags:
- assets
status: draft
type: post
published: true
comments: true
author:
  login: amunda
---

There are many articles and blogs about how to use the Redux library and how Redux is an excellent compliment to React. However, In these series of blog posts I would like to talk about things which i learned from looking at Redux codebase. 

I will specifically focus on functional paradigms found in Redux code. I will assume that you are familiar with Redux and understand how to use it. If you don't know how to use the Redux library then checkout this amazing  <a href="https://egghead.io/series/getting-started-with-redux">video series</a> on Redux by Dan Abramov. 

In this first part of the series I will like to look at the function called 'createStore'. As we know that we call 'createStore' to get the Redux store which has all the state of the Application. But lets explore what does this createStore method do under the hood to keep the state.

If you are coming from OO paradigm then you are used to thinking that if you want to store the state of anything, you would create a class and then use instance variables to hold the state. If you are thinking in OO paradigm you might be thinking that createStore would do something like this:

{% highlight js %} 

class ReduxStore {
  constructor(reducer, initialState) {
    this.reducer = reducer;
    this.initialState = initialState;
  }
  
  dispatch(action) {
    this.initialState = this.currentReducer(currentState, action)
    ...
  } 
  ...
}

export default function createStore(reducer, initialState) {
    store = new ReduxStore(reducer, initialState)
    return store;
}

{% endhighlight %}

But this is not the way its implemented in Redux, it's because it follows the functional paradigm. In functional pradiagm, you can completely skip the creation of the class to keep the state. You can use the powerful feature of javascript called <a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures">Closures</a> to keep the state. This is what exactly Redux do as well in createStore. Here is the Redux version:

{% highlight js %} 
export default function createStore(reducer, initialState) {
  var currentState = initialState;
  var currentReducer = reducer; 

  function dispatch(action) {
    ...
    // No need to reference 'this' which makes it easy because you don't have // to worry about wrong binding
    currentState = currentReducer(currentState, action)
    ...
  }

  // Put all public methods, easy to create private methods
  return {
    ...
    dispatch: dispatch
    ...
  }
}
{% endhighlight %}







