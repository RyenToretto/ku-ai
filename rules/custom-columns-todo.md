# 自定义列 TODO 同步规则说明

## 设计意图

该规则用于约束 `xuanhu-ai` 项目中所有「自定义列」相关任务都必须同步维护 `docs/custom-columns/TODO.md`，避免问题修复、设计债务和后续迭代散落在聊天记录或临时文档中。

## 适用场景

- 修改 `src/mixins/tableControl.js`
- 修改 `src/components/globalComponents/TableWrap.vue`
- 修改 `src/components/globalComponents/doTable/**/*.vue`
- 修改 `docs/custom-columns/**`
- 在业务页面接入、调整、修复自定义列能力
- 讨论或规划自定义列相关问题修复与后续迭代

## 执行要求

- 当前阶段优先处理当前实现问题，暂不推进新功能迭代。
- 未完成任务使用 `⭕️` 标记。
- 已完成任务使用 `✅` 标记。
- 任务开始前检查 `docs/custom-columns/TODO.md`。
- 任务完成后同步更新 `docs/custom-columns/TODO.md`。

## 对应规则文件

- `custom-columns-todo.mdc`
