# Redux CheatSheet | Redux 设计理念与实践技巧

Redux 是受 Flux 启发的，类似于 Event Sourcing 的事件驱动型框架。

# 基础组件

# 实践工具

# ducks

# redux-actions

```js
import { createActions, handleActions, combineActions } from 'redux-actions';

const defaultState = { counter: 10 };

const { increment, decrement } = createActions({
  INCREMENT: (amount = 1) => ({ amount }),
  DECREMENT: (amount = 1) => ({ amount: -amount })
});

const reducer = handleActions(
  {
    INCREMENT: (state, action) => ({
      counter: state.counter + action.payload
    }),

    DECREMENT: (state, action) => ({
      counter: state.counter - action.payload
    })
  },
  defaultState
);

const reducer = handleActions(
  {
    [combineActions(increment, decrement)](
      state,
      {
        payload: { amount }
      }
    ) {
      return { ...state, counter: state.counter + amount };
    }
  },
  defaultState
);

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
  payload: Promise.resolve()
});
```

# Async Actions: 异步 Action 处理

## Promise

[redux-promise](https://github.com/redux-utilities/redux-promise) 会自动处理 payload 为 Promise 对象的 Action，并且会分发该 Action 的副本，包含了 Promise 的处理结果，并且根据处理结果动态设置了 status 属性为 `success` 或者 `error`。

```js
// 创建简单的异步 Action
createAction('FETCH_THING', async id => {
  const result = await somePromise;
  return result.someValue;
});

// 与自定义的 WebAPI 协同使用
import { WebAPI } from '../utils/WebAPI';

export const getThing = createAction('GET_THING', WebAPI.getThing);
export const createThing = createAction('POST_THING', WebAPI.createThing);
export const updateThing = createAction('UPDATE_THING', WebAPI.updateThing);
export const deleteThing = createAction('DELETE_THING', WebAPI.deleteThing);
```

[redux-promise-middleware](https://github.com/pburtchaell/redux-promise-middleware) 为我们提供了类似的异步处理功能，其能够接受某个 Promise，并且依次分发 Pending, Fulfilled, 以及 Rejected 这几个不同状态的 Action:

```js
const promiseAction = () => ({
  type: 'PROMISE',
  payload: Promise.resolve()
});
```

该工具同样可以与 Redux Thunk 协同使用，来进行多个 Action 的分发：

```js
const secondAction = data => ({
  type: 'TWO',
  payload: data
});

const first = () => {
  return dispatch => {
    const response = dispatch({
      type: 'ONE',
      payload: Promise.resolve()
    });

    response.then(data => {
      dispatch(secondAction(data));
    });
  };
};
```

我们也可以自己实现 Promise 中间件，可以参考 [redux/promiseMiddleware](https://github.com/wxyyxc1992/coding-snippets/tree/master/web/redux) 的源代码实现。
