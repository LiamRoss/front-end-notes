# Redux Notes

Notes outlining the best practices for the various Redux components, and some semantics when using it in React environments, or with TypeScript.

#### Skip to:
1. [Actions](#actions)
1. [Reducers](#reducers)
	- [Basics](#basics)
	- [Immutable Examples](#immutable)
1. [Store](#store)
1. [Middleware](#middleware)

---

## Actions

Actions specify what has or should happen.

* A simple-as-possible JavaScript Object that encompasses all the required information
* Must have a **type** field, everything else is optional

```javascript
// example action
{
  type:    'CREATE_TODO',
  payload: 'Build my first Redux app'
}
```

---

## Reducers

### Basics

Reducers are functions that receive the state of the app and an action, and return a new state.

#### 1) Example Reducer

```javascript
const todos = (state = [], action) => {
  switch(action.type) {
    case 'ADD_TODO':    // First action type
      return [
        ...state,
        {                             // concat new todo onto state array
          id: action.id,
          text: action.text,
          completed: false
        }
      ];
    case 'TOGGLE_TODO':  // Second action type
      return state.map(todo => {      // map each todo in state
        if (todo.id !== action.id) {  // if id doesn't match, do nothing
          return todo;
        }
        return {                      // if id does match, toggle completed
          ...todo,
          completed: !todo.completed  // overwrites completed
        };
      })
    default:
      return state;
  }
};
```

#### 2) Example using Reducer Composition (abstract away some of reducer functionality)

```javascript
// "sub" reducer, deals with a smaller part of the state
const todo = (state, action) => {
  switch(action.type) {
    case 'ADD_TODO':
      return {
        id: action.id,
        text: action.text,
        completed: false
      };
    case 'TOGGLE_TODO':
      if (state.id !== action.id) {           // state == individual todo
        return state;
      }

      return {
        ...state,
        completed: !state.completed
      };
};

// "main" reducer, distributes tasks to sub reducer
const todos = (state = [], action) => {
  switch(action.type) {
    case 'ADD_TODO':
      return [
        ...state,
        todo(undefined, action)               // call sub reducer todo
      ];
    case 'TOGGLE_TODO':
      return state.map(t => todo(t, action)); // call sub reducer todo
    default:
      return state;
  }
};
```

### Immutable

Reducers must be kept pure, and must return a new state without altering the previous state.

#### 1) Change **Object** element

* Use Object.assign to change only a single part of the state

```javascript
// Old
return Object.assign({}, todo, {
  completed: !todo.completed                  // overwrites completed in todo
});

// ES6 (use this)
return {
  ...todo,
  completed: !todo.completed                  // overwrites completed in todo
};
```

#### 2) Change **List** element

* Use slice to alter single item in a list

```javascript
// Old
.slice(0, index)
  .concat([list[index] + 1])                  // incrementing this item
  .concat(list.slice(index + 1));

// ES6 (use this)
return [
  ...list.slice(0, index),
  list[index] + 1,                            // incrementing this item
  ...list.slice(index + 1)
];
```

#### 3) Add element

* Use concat instead of push to add an element into the state

```javascript
// Old
return list.concat([0]);

// ES6 (use this)
return [...list, 0];
```

#### 4) Remove element

* Use slice instead of splice to remove item from array

```javascript
// Old
return list
  .slice(0, index)
  .concat(list.slice(index + 1));

// ES6 (use this)
return [
  ...list.slice(0, index),
  ...list.slice(index + 1)
];
```

---

## Store

The store is the one source of truth for all of the Redux components.

* Brings together the reducers, state, and middleware (see [Middleware](#middleware))

```javascript
import { createStore, applyMiddleware } from 'redux';
import { composeWithDevTools } from 'redux-devtools-extension';

import { reducers } from './reducers/index';

// Create a middlewares array in order to push all middlewares independently.
const middlewares = [];

// Push each imported middleware to the middlewares array.
// ex: middlewares.push(someMiddleware);

// Apply middlewares to middleware constant.
const middleware = applyMiddleware(...middlewares);

// Create store with devtools and middleware.
const store = createStore(reducers, composeWithDevTools( middleware ));

export { store };
```

---

## Middleware

Middleware gives you the opportunity to intercept dispatched actions, and complete asynchronous actions before forwarding new actions back to the reducers.

#### Examples:

[Thunk Middlware](https://github.com/gaearon/redux-thunk)

[Saga Middleware](https://github.com/redux-saga/redux-saga/blob/master/README.md)



