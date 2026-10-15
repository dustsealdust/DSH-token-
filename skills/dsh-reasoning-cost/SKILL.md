---
name: dsh-reasoning-cost
description: 把 DeepSeek Harness 的推理强度统一固定到 low，压低无效思考 token 与响应延迟。
---

# 降低 DSH 推理成本

## 适用场景

- 使用量偏高，想在不牺牲主要任务质量的前提下压成本。
- 不同 provider / route 的推理强度不一致，导致某些入口仍在花高价思考。

## 前置条件

- 能定位到 profile 的 loader 补丁文件（`cordis.patch.yml`）。
- 已备份该文件。

## 操作步骤

三处一起压，避免任何一条 route 漏掉：

1. `agent-default-model`：顶层 `reasoningEffort: low`。
2. `llm-deepseek`：顶层 `reasoningEffort: low`，并按模型列 `reasoningEfforts`。
3. `llm-pi-ai`：**每条 route** 加 `reasoning: low`，各模型列 `reasoningEfforts`。

改前各留一份带时间戳的备份（如 `.bak-before-low`、`.bak-before-piai`）。

## 验证

- 重新读取补丁文件，确认三处都在，且没有残留的 `high` / 未声明 route。
- 实际发一轮对话，观察响应是否仍按预期工作；再回看用量是否下降。

## 风险与边界

- `low` 会降低复杂推理任务的深度。遇到需要长链路推理的任务，按需在单个会话里临时提高，不要全局回退。
- 该文件是 loader 补丁数组，**不含提示词正文**；改它不会改变人设或系统提示词。

## 回滚

把备份文件换回原位，或删除新增的三处字段。回滚不涉及任何数据迁移。

## 备注

- 成本感知优先于「配置好看」：宁可按需求临时提高，也不要全局常驻 high。
- 若用量仍高，先做 `dsh-slow-turn-forensics` 判断是否属于「多次串行小往返」，再决定优化方向。
