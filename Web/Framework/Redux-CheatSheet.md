# Redux CheatSheet | Redux 设计理念与实践技巧清单

Redux 是受 Flux 启发的，类似于 Event Sourcing 的事件驱动型框架，能够为我们优雅地处理了 React 等前端框架中的跨层级的组件状态管理问题。从 OOP 的角度来看，As in OOP, Redux inverts the responsibility of control from caller to receiver — the UI doesn’t directly manipulate the state but rather sends it a message for the state to interpret.

Through this lens, a Redux store is an object, reducers are method handlers, and actions are messages. store.dispatch({ type: "foo", payload: "bar" }) is equivalent to Ruby's store.send(:foo, "bar"). Middleware are used in much the same way Aspect-Oriented Programming (e.g. Rails' before_action) and React-Redux's connect is dependency injection.

凡事皆有两面性，Redux 在为我们带来完全地可预测性与可测试性的同时，也带来了一定的代价，这方面可以参考[现代 Web 导论/数据流驱动的界面]()一节中的讨论。

# Principles | 设计理念与简要实现

![Redux 设计理念](https://user-images.githubusercontent.com/5803001/39965487-55f864c2-56cc-11e8-82c8-c9d00a6ae2fa.png)

# Basic Usage | 基本使用

## Store

## Action

## Reducer

```js
import { combineReducers } from 'redux';
import currentUserReducer from './currentUserReducer';
import postsListReducer from './postsListReducer';

export default combineReducers({
  currentUser: currentUserReducer,
  postsList: postsListReducer
});
```

## React

```js
import { connect } from 'react-redux';

const mapStateToProps = state => {
  // state is the state of our store
  // return the props that we want to use for our component
  return {
    count: state.count
  };
};

const mapDispatchToProps = dispatch => {
  // dispatch is our store dispatch function
  // return the props that we want to use for our component
  return {
    onButtonClicked: () => {
      dispatch({ type: 'BUTTON_CLICKED' });
    }
  };
};

// create our enhancer function
const enhancer = connect(mapStateToProps, mapDispatchToProps);

// wrap our "dumb" component with the enhancer
const HelloWorldContainer = enhancer(HelloWorld);

// and finally we export it
export default HelloWorldContainer;
```

# Async Actions | 异步 Action 处理

## Thunk

```js
const setLoading = createAction('SET_LOADING')
const setUser = createAction('SET_USER')
const unsetLoading = createAction('UNSET_LOADING')

function login(credentials) {
	return (dispatch) => {
		dispatch(setLoading());

		authenticate(credentials)
			.then(user => {
				dispatch(batchActions([
					setUser(user),
					unsetLoading()
				], 'LOGIN_SUCCESS'))
			})
		})
	}
}
```

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

# 样式风格

## ducks

## redux-actions

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

# Middleware | 中间件
