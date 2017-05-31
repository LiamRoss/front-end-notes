# Saga Middleware

Notes on Saga middleware syntax and usage with redux.

---

## Basics

#### Dependencies

`npm install --save redux-saga` or `yarn add redux-saga`

#### Notes

1. Intercept actions
    * actions are given a type that is unique to the saga middleware files
        * actions can be intercepted as takeEvery or takeLatest
            * takeEvery: allows concurrent fetches
            * takeLatest: no concurrent fetches, will cancel pending fetch if new one is dispatched
      * see [Examples of Saga Files](#saga-files)
2. Do async action
    * sagas use ES6 Generator functions to complete their async actions
    * generators syntax is `export function* saga() {...}`
3. Send new action back to reducers with async payload
    * syntax is `yield put({ type: 'ASYNC_SUCCEEDED', payload: "" })`

---

## Examples

### Store

```javascript
import { createStore, applyMiddleware } from 'redux';
import { composeWithDevTools } from 'redux-devtools-extension';
import { reducers } from './reducers/index';
// Importing middleware creator and own saga files.
import createSagaMiddleware from 'redux-saga';
import { sagas } from './sagas/index';

// Create a middlewares array in order to push all middlewares independently.
const middlewares = [];

// Push each imported middleware to the middlewares array.
// 1. Saga Middleware
const sagaMiddleware = createSagaMiddleware();
middlewares.push(sagaMiddleware);

// Apply middlewares to middleware constant.
const middleware = applyMiddleware(...middlewares);

// Create store with devtools and middleware.
const store = createStore(reducers, composeWithDevTools( middleware ));

// Run the saga middleware after the store is created.
sagaMiddleware.run(sagas);

export { store };
```

### Saga Files

#### index.jsx

```javascript
import { takeLatest, fork, all } from 'redux-saga/effects'

import { fetchUsers } from './usersSaga';

// Our watcher Saga: spawn a new incrementAsync task on each USERS_FETCH.
export function* sagas() {
  yield all([
    fork(takeLatest, 'USERS_FETCH', fetch),
    // Add forks here for more saga files.
    // ex. fork(takeLatest, 'USERS_DELETE', delete)
  ])
}
```

#### usersSaga.jsx

```javascript
import { call, put } from 'redux-saga/effects';
import { fetchFromAPI } from '../api/api';

// Our worker Saga: will perform the async increment task.
export function* fetchUsers() {
  try {
    const users = yield call(fetchUsers);
    yield put({ type: 'USERS_FETCH_SUCCEEDED', users })
  }
  catch(error) {
    yield put({ type: 'USERS_FETCH_FAILED', error })
  }
}
```






