题注：本文详细地阐述了 Redux 的设计理念与实践技巧，包含了其三大原则与简单的仿制、基础组件以及 React 集成使用、基于 Thunk, Promise, Sagas 三种不同的异步处理方式、Selector, Ducks 等其他常见的样式规范、中间件的实现原理与代码分析等。

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

## Sagas

Sagas 是源于[计算机科学与技术](http://www.cs.cornell.edu/andru/cs711/2002fa/reading/sagas.pdf)的概念，用于描述事务及其关联处理操作的约束。[redux-sagas](https://github.com/redux-saga/) 的官方描述是用于处理副作用(Side Effects)的 Redux 中间件，允许我们以同步地方式编写异步代码，并使用 try-catch 进行异常处理。

![redux-saga-dataflow](https://user-images.githubusercontent.com/5803001/40041848-05f758a2-5852-11e8-9ff5-12a9ff8046aa.png)

与 redux-thunk 相比，[redux-sagas](https://github.com/redux-saga/) 并不是直接从 UI 调用逻辑代码，而是进行纯粹地 dispatch action；所有的异步流程控制都被移入到了 sagas，从而增强组件与逻辑代码的可复用性与可测试性。redux-sagas 基于 ES6 Generator，能为我们提供高级的异步控制流以及并发管理，可以使用简单的同步方式描述异步流，并通过 fork 实现并发任务。

![8cc1a873-c675-9009-570d-9684da4a704f](https://user-images.githubusercontent.com/5803001/40041849-063158b8-5852-11e8-9e24-39bcbdef83a4.png)

参考 [fe-boilerplate/redux](https://parg.co/YA2) 的示例，我们首先需要引入并且创建 Sagas 中间件实例：

```js
import createSagaMiddleware from 'redux-saga';
import rootSaga from '../sagas/sagas';

const sagaMiddleware = createSagaMiddleware();

// 引入中间件，并构建 Store 对象
...

sagaMiddleware.run(rootSaga);
```

这里的 sagas 文件，即包含了具体的逻辑处理代码：

```js
// helloSaga 会在 sagaMiddleware.run 时即刻执行
export function* helloSaga() {
  console.log('Hello Saga!');
}

// worker saga
export function* incrementAsync() {
  yield call(delay, 1000);
  // 继续分发事件
  yield put({ type: 'SAGA_INCREMENT' });
}

// watcher saga
export function* watchIncrementAsync() {
  // 监听 Action，并执行关联操作
  yield takeEvery('SAGA_INCREMENT_ASYNC', incrementAsync);
}

// root saga
export default function* rootSaga() {
  yield [helloSaga(), watchIncrementAsync()];
}
```

Sagas 为我们定义了三种不同的 Saga，其中 Worker Saga 负责 API 调用、异步请求、结果处理等具体的任务；Watcher Saga 则监听被 dispatch 的 actions，当接收到 action 或者知道其被触发时，调用 worker saga 执行任务。而 Root Saga 则是立即启动 Sagas 的唯一入口。Saga 使用 Generator 函数来 yield Effect，其利用生成器可以暂停执行，再次执行的时候从上次暂停的地方继续执行的特性。Effect 是一个简单的对象，该对象包含了一些给 middleware 解释执行的信息。可以通过使用 effects API， 如 fork，call，take，put，cancel 等来创建 Effect。

如 `yield call(fetch, '/user')` 即 `yield` 了下面的对象，`call` 创建了一条描述结果的信息，然后，`redux-saga` middleware 将确保执行这些指令并将指令的结果返回给生成器：

```
{
  type: CALL,
  function: fetch,
  args: ['/user']
}
```

我们也可以并发执行多个任务：

```js
const [users, repos] = yield[(call(fetch, '/users'), call(fetch, '/repos'))];
```

同样以常见的接口请求，与结果处理为例：

```js
import { take, fork, call, put } from 'redux-saga/effects';

// The worker: perform the requested task
function* fetchUrl(url) {
  // 指示中间件调用 fetch 异步任务
  const data = yield call(fetch, url);

  // 指示中间件发起一个 action 到 Store
  yield put({ type: 'FETCH_SUCCESS', data });
}

// The watcher: watch actions and coordinate worker tasks
function* watchFetchRequests() {
  while (true) {
    // 指示中间件等待 Store 上指定的 action，即监听 action
    const action = yield take('FETCH_REQUEST');

    // 指示中间件以无阻塞调用方式执行 fetchUrl
    yield fork(fetchUrl, action.url);
  }
}
```

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
