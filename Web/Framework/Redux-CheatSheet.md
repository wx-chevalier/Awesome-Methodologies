# Redux CheatSheet

```js
import { createActions, handleActions, combineActions } from 'redux-actions'

const defaultState = { counter: 10 };

const { increment, decrement } = createActions({
  INCREMENT: (amount = 1) => ({ amount }),
  DECREMENT: (amount = 1) => ({ amount: -amount })
});

const reducer = handleActions({
  [combineActions(increment, decrement)] (state, { payload: { amount } }) {
    return { ...state, counter: state.counter + amount };
  }
}, defaultState);

export default reducer;
```

```js
import { createStore, applyMiddleware } from 'redux';
import promiseMiddleware from 'redux-promise-middleware';
import thunkMiddleware from 'redux-thunk';
import logger from 'redux-logger';

const reducer = (state = {}, action) => {
  ...
}

const store = createStore(reducer, {}, applyMiddleware(
  thunkMiddleware,
  promiseMiddleware(),
  logger,
));

export default store;
```

```js
const promiseAction = () => ({
  type: 'PROMISE',
  payload: Promise.resolve(),
})
```
