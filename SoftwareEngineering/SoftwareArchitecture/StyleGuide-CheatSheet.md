```js
/**
 * 按钮点击事件
 * @param {number} status 活动状态：1 开团进行中 2 开团失败 3 商品售罄 4 开团成功 5 系统取消
 */
const onButtonClick = (status) => {
  if (status == 1) {
    sendLog('processing') jumpTo('IndexPage')
  } else if (status == 2) {
    sendLog('fail') jumpTo('FailPage')
  } else if (status == 3) {
    sendLog('fail') jumpTo('FailPage')
  } else if (status == 4) {
    sendLog('success') jumpTo('SuccessPage')
  } else if (status == 5) {
    sendLog('cancel') jumpTo('CancelPage')
  } else {
    sendLog('other') jumpTo('Index')
  }
}
```

```js
const actions = {
  '1': ['processing', 'IndexPage'],
  '2': ['fail', 'FailPage'],
  '3': ['fail', 'FailPage'],
  '4': ['success', 'SuccessPage'],
  '5': ['cancel', 'CancelPage'],
  default: ['other', 'Index']
};
/**
 * 按钮点击事件
 * @param {number} status 活动状态：1开团进行中 2开团失败 3 商品售罄 4 开团成功 5 系统取消
 */
const onButtonClick = status => {
  let action = actions[status] || actions['default'],
    logName = action[0],
    pageName = action[1];
  sendLog(logName);
  jumpTo(pageName);
};
```

```js
const actions = () => {
  const functionA = () => {
    /*do sth*/
  };
  const functionB = () => {
    /*do sth*/
  };
  const functionC = () => {
    /*send log*/
  };
  return new Map([
    [/^guest_[1-4]$/, functionA],
    [/^guest_5$/, functionB],
    [/^guest_.*$/, functionC]
    //...
  ]);
};

const onButtonClick = (identity, status) => {
  let action = [...actions()].filter(([key, value]) =>
    key.test(`${identity}_${status}`)
  );
  action.forEach(([key, value]) => value.call(this));
};
```
