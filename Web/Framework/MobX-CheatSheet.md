# MobX CheatSheet

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
