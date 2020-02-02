

# MobX CheatSheet | MobX 语法基础，实践技巧与设计理念

MobX 遵循透明函数响应式编程 TFRP 的设计理念，允许我们按照惯有的面向对象的思想来编写代码，而不需要去学习很多新的范式或者抽象概念。

严格来说，MobX 应该看做数据流(Dataflow)库，而不是一个完整的状态管理框架；MobX 也不限定于双向数据流，在实践中，我们同样推崇单向数据流的机制，即避免在组件中以属性访问的方式进行值修改。Redux 则是有着强约束的 Opinionated 状态管理框架。Redux 为我们提供了完全地可预测性(Predicability)与可测试性(Testability)的同时，也带了流程的碎片化与大量的模板代码，其异步处理的流程也相对复杂。而 Cycle.js 同样学习曲线也较为陡峭，其遵循不同的设计范式，需要使用者充分理解 RxJS 的理念。与基于 [React Context]() 的原生状态管理方案，MobX 又能提供独立于框架的状态管理方案，便于我们分割与重用逻辑代码，并且进行测试用例。

# MobX 的基础使用

## 测试

```js
class MessageStore {
  // bad
  markMessageAsRead = message => {
    if (message.status === 'new') {
      fetch({
        method: 'GET',
        path: `/notification/read/${message.id}`
      }).then(() => (message.status = 'read'));
    }
  };
  // good
  markMessageAsRead = message => {
    if (message.status !== 'new') {
      return Promise.reject('Message is not new');
    }
    // it's now easily mockable
    return api.markMessageAsRead(message).then(() => {
      // this is a pure function
      // you can test it easily
      return this.updateMessage(message, { status: ' read' });
    });
  };
}
```

# MobX 与 React 集成范式

# MST

# MobX 的设计理念

## MobX 与 Redux 的对比

本文的最后，我们也讨论下

First of all, state within Redux is simply represented as plain objects. Nothing more. Redux makes state tangible. You can pick up objects and drop them in a different place. Like in local storage or in the body of a network request.

Secondly, Redux offers many constraints that make it easy to write generic algorithms that reason about the state without specific domain knowledge (unlike reducers). Such algorithms are typically expressed as middleware. Creating such mechanisms is feasible, partly because the data is immutable, but mostly because the state is tree shaped and preferably serializable. This makes it possible to traverse any Redux state tree in a predictable manner.

[mobx-state-tree](https://github.com/mobxjs/mobx-state-tree)

```js
await this.props.actions.choose({
    workShiftIdList: _.map(items, d => d.workShiftId)
});
this.trace("batch-chose", { size: items.length });
await this.fetchTableData();

actions.chooseWorks = (works) => (dispatch) => {
  choose(works).then(
    () => {
      // 更新结果信息
      dispatch(createAction(CHOOSE_WORKS));
      // 执行重新抓取操作
      dispatch(actions.fetchChoices());
      // 清空已选
      dispatch({
        type: `${prefix}/CLEAR_SELECTED_WORKs`
      });
    },
    () => {}
  );
};
```

# WebSocket

```ts
import { onBecomeObserved, onBecomeUnobserved } from 'mobx';
import { observable, decorate } from 'mobx';
class AutoObservable<T> {
  data: T;

  constructor(onObserved: () => void, onUnobserved: () => void) {
    onBecomeObserved(this, 'data', onObserved);
    onBecomeUnobserved(this, 'data', onUnobserved);
  }
}
decorate(AutoObservable, {
  data: observable
});
```

```ts
let autoObservable: AutoObservable<number>;
let socket: Websocket;
const openStream = () => {
  socket = new Websocket('ws://localhost:8080');
  socket.on('message', message => {
    autoObservable.data = message;
  });
};
const closeStream = () => {
  socket.close();
};
autoObservable = new AutoObservable(openStream, closeStream);
```
