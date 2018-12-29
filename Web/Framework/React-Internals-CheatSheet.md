# React 内部原理与实现浅析

# setState

## 多端识别

The answer is that every renderer sets a special field on the created class. This field is called updater. It’s not something you would set — rather, it’s something React DOM, React DOM Server or React Native set right after creating an instance of your class:

```js
// Inside React DOM
const inst = new YourComponent();
inst.props = props;
inst.updater = ReactDOMUpdater;

// Inside React DOM Server
const inst = new YourComponent();
inst.props = props;
inst.updater = ReactDOMServerUpdater;

// Inside React Native
const inst = new YourComponent();
inst.props = props;
inst.updater = ReactNativeUpdater;
```

Looking at the setState implementation in React.Component, all it does is delegate work to the renderer that created this component instance:

```js
// A bit simplified
setState(partialState, callback) {
  // Use the `updater` field to talk back to the renderer!
  this.updater.enqueueSetState(this, partialState, callback);
}
```

Instead of an updater field, Hooks use a “dispatcher” object. When you call React.useState(), React.useEffect(), or another built-in Hook, these calls are forwarded to the current dispatcher.

```js
// In React (simplified a bit)
const React = {
  // Real property is hidden a bit deeper, see if you can find it!
  __currentDispatcher: null,

  useState(initialState) {
    return React.__currentDispatcher.useState(initialState);
  },

  useEffect(initialState) {
    return React.__currentDispatcher.useEffect(initialState);
  },
  // ...
};
```

And individual renderers set the dispatcher before rendering your component:

```js
// In React DOM
const prevDispatcher = React.__currentDispatcher;
React.__currentDispatcher = ReactDOMDispatcher;
let result;
try {
  result = YourComponent(props);
} finally {
  // Restore it back
  React.__currentDispatcher = prevDispatcher;
}
```
