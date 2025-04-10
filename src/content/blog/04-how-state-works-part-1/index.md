---
title: "How State Works Part 1"
description: "Explore the basics of how state works. Part 1 of ?"
date: "Apr 10 2025"
---

# Back to the basics

When using vanilla JS, if you have ever tried to make a complex web application you will have 100’s or 1000’s of manually appending elements and removing them from the DOM, while also reflecting changes made to the state of the application. Let's say you have an object with locally scoped variables and get and set methods:

```javascript
function createCounter() {
  let count = 0;

  return {
    getCount: () => count,
    setCount: (value) => { count = value; }
  };
}

const counter = createCounter();
counter.setCount(1);
console.log(counter.getCount()); // 1
```

You’ll have to keep track of your entire DOM, and remove elements or add them in specified places. Imagine if you were using the count in multiple areas of the DOM. You would have to track these and ensure that all places are updated according to the most recent value.

This is traditionally done with an observer pattern approach, where elements are registered to be observable, and so they will each inherit these methods of get and set. Taking our code example above you could modify it to meet this requirement by adding subscribers to a Set, and iterating over it to hook up get and set methods.

```javascript
function createObservable(value) {
  let internalValue = value;
  const subscribers = new Set();

  return {
    get: () => internalValue,
    set: (newValue) => {
      internalValue = newValue;
      subscribers.forEach(fn => fn(newValue));
    },
    subscribe: (fn) => {
      subscribers.add(fn);
      fn(internalValue); // Optional immediate call
      return () => subscribers.delete(fn);
    }
  };
}
```

Now we could register elements like so:

```javascript
// Usage:
const count = createObservable(0);

// Subscribe two DOM elements
const el1 = document.getElementById("count-display-1");
const el2 = document.getElementById("count-display-2");

count.subscribe(value => {
  el1.textContent = value;
  el2.textContent = value;
});

// Later, change the value:
count.set(5); // Both elements will update
```

While this is a great first step, those with a keen eye will have noticed this won’t actually track state over time, since our `internalValue` is locally scoped to the function. So you can’t do an operation like `count += 1`, if we wanted to increment the value over time.

# Meet Proxy

This is where we could use what's known as a Proxy object, which is, to put it simply, a middleman between the user and an object. When you try to change the value of the object, the proxy intercepts it and performs some function. In our example below, `const state` will be the original object, and `const stateProxy` is our proxy object.

```javascript
const state = { counter: 0 };

const stateProxy = new Proxy(state, {
  set(target, prop, value) {
    target[prop] = value;
    if (prop === 'counter') {
      document.getElementById('counter-display').textContent = value; // Automatically update DOM
    }
    return true;
  }
});
```

Proxy takes in a target object (`const state`) and a handler object. The middleman object will intercept when you try to mess with the target object. Proxy has a number of “trap” methods which are unique to Proxy. `set` is just one of these methods, which takes in a target, property (`prop`), and a value (which is what we’ll be incrementing).

Now if you call:

```javascript
stateProxy.counter += 1;  // stateProxy triggers set method since we are assigning a new value, incrementing counter in the state object, and reflecting the change in the DOM.
```

This is just a glimpse of the complexity that goes on when attempting to manage state across a web application. Proxies are used under the hood by frameworks like Vue, Svelte, SolidJS, and more.

Keeping track of state is already complicated enough with tracking our simple counter. Now imagine how complicated it would be to track changes to 100’s of variables across an application. Not to mention, our implementation doesn’t account for error handling, or optimize performance whatsoever.

Now that we understand a bit about state management at a basic level, in part 2 we’ll talk about what React does, and what it does differently.
