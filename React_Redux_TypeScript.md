# React and Redux with TypeScript

Notes outlining and comparing the use of TypeScript in a React + Redux application.

#### Skip to:
1. [Components](#components)
2. [Reducers](#reducers)
3. [Store](#store)
4. [Dependencies](#dependencies)

---

## Components

#### JavaScript

```javascript
import React, { Component } from 'react';
import { connect } from 'react-redux';
import { changeWorld } from '../actions/WorldActions';

// Map any listener props to their elements in the state.
const mapStateToProps = (state) => ({ world: state.world});
// Map any dispatcher props to a dispatch function.
const mapDispatchToProps = (dispatch) => ({
  change: (event) => {
    dispatch(changeWorld(event.target.value));
  }
})

class World extends Component {
  render() {
    return(
      <div className="World">
        <div className="world-title">
          Hello {this.props.world.name}
        </div>
        <input 
          className="world-input" 
          type="text" 
          onChange={this.props.change}
        />
      </div>
    )
  }
}

export default connect(mapStateToProps, mapDispatchToProps)(World);
```

#### TypeScript

```typescript
import * as React from 'react';
import { Component } from 'react';
import { connect } from 'react-redux';
import { changeWorld } from '../actions/WorldActions';
// Import types for typechecking.
import { Store } from '../store';
import * as Redux from 'redux';

// Set interfaces for own state and props, as well as connected (for mapping).
interface OwnState {}                       // own state
interface OwnProps {}                       // own props
interface ConnectedState {                  // any props from mapStateToProps
  name: string;
}
interface ConnectedDispatch {               // any props from mapDispatchToProps
  change: (event: any) => void;
}

// Map any listener props to their elements in the state.
const mapStateToProps = (state: Store.All) => ({name: state.world.name});
// Map any dispatcher props to a dispatch function.
const mapDispatchToProps = (dispatch: Redux.Dispatch<Store.All>) => ({
  change: (event: any) => {
    dispatch(changeWorld(event.target.value));
  }
});

class World extends Component<OwnProps & ConnectedState & ConnectedDispatch, OwnState> {
  render() {
    return(
      <div className="World">
        <div className="world-title">
          Hello {this.props.name}
        </div>
        <input 
          className="world-input" 
          type="text" 
          onChange={this.props.change}
        />
      </div>
    );
  }
}

export default connect(mapStateToProps, mapDispatchToProps)(World);
```

---

## Reducers

#### JavaScript
index.jsx
```javascript
import { combineReducers } from "redux";
import { worldReducer as world } from './worldReducer';

const reducers = combineReducers({
  world
});

export { reducers };
```
WorldReducer.jsx
```javascript
// Declare the base state.
const defaultState = {
  name: ''
};

export const worldReducer = (state = defaultState, action) => {
  switch (action.type) {
    case "CHANGE_WORLD":
      return {
        ...state, 
        name: action.name
      };
    default:
      return state
  }
}
```

#### TypeScript
index.tsx
```typescript
import { combineReducers } from 'redux';
import { worldReducer as world } from './WorldReducer';
import { Store } from '../store';

const reducers = combineReducers<Store.All>({
  world
});

export { reducers };
```
WorldReducer.tsx
```typescript
import { WorldAction } from '../actions/WorldActions';
import { Store } from '../store';

// Declare the base state.
const defaultState: Store.World = {
  name: ''
};

export const worldReducer = (state: Store.World = defaultState,
                             action: WorldAction) => {
  switch (action.type) {
    case 'CHANGE_WORLD':
      return {
        ...state,
        name: action.name
      };
    default:
      return state;
  }
};
```

---

## Store

#### JavaScript

```javascript
import { createStore, applyMiddleware } from 'redux';
import { composeWithDevTools } from 'redux-devtools-extension';

import { reducers } from './reducers/index';

// Create a middlewares array in order to push all middlewares independently.
const middlewares = [];

// Push each imported middleware to the middlewares array.
// TODO: push middleware to middlewares array

// Apply middlewares to middleware constant.
const middleware = applyMiddleware(...middlewares);

// Create store with devtools and middleware.
const store = createStore(reducers, composeWithDevTools( middleware ));

export { store };
```

#### TypeScript

```typescript
import { createStore, applyMiddleware } from 'redux';
import { composeWithDevTools } from 'redux-devtools-extension';

import { reducers } from './reducers';

// Export Store namespace. This describes the entire state of the application.
export namespace Store {
    export type World = {                // world element, contains name
      name: string
    };
    export type All = {                  // complete state, contains world
        world: World
    };
}

// Create a middlewares array in order to push all middlewares independently.
const middlewares: any[] = [];

// Push each imported middleware to the middlewares array.
// TODO: push middleware to middlewares array

// Apply middlewares to middleware constant.
const middleware = applyMiddleware(...middlewares);

// Create store with devtools and middleware.
const store = createStore(reducers, composeWithDevTools( middleware ));

export { store };
```

## Dependencies

In some cases, a dependency import will need to be prefaced with `@types/`.

For example:
```json
"dependencies": {
  "@types/node": "^7.0.22",
  "@types/react": "^15.0.25",
  "@types/react-dom": "^15.5.0",
  "@types/react-redux": "^4.4.40",
  "react": "^15.5.4",
  "react-dom": "^15.5.4",
  "react-redux": "^5.0.5"
}
```

In this case, these would have to be imported with `yarn add @types/react` or `npm install --save @types/react`.
