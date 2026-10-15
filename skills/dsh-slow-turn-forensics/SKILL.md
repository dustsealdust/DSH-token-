---
name: dsh-slow-turn-forensics
description: 用多帧 zstd 会话日志定量定位 DSH turn 变慢或被中止的真实原因，而不是靠猜。
---

# 慢 turn 取证

## 适用场景

- 某个 turn 明显卡顿、超时，或被用户中止。
- 想知道时间到底花在「一次长思考」还是「多次小往返」。

## 数据位置

```text
$DSH_HOME/sessions/<cwd-slug>/<session-id>/session.v4.jsonl.zstd
```

注意是**多帧（multi-frame）zstd**，不是单帧。

## 操作步骤

一次性解压只会得到约 185 字节的 header，必须逐帧顺移：

```js
let off = 0;
while (off < buf.length) {
  const r = zlib.zstdDecompressSync(buf.subarray(off), { info: true });
  off += r.engine.bytesWritten;
}
```

解出的 JSONL 事件类型：

| 事件 | 用途 |
|---|---|
| `session` | 会话元信息 |
| `turn/start` / `turn/end` | turn 总时长；`turn/end` 的 `reason.kind` 给出终止原因（如 `aborted`） |
| `step/start` / `step/end` | 逐 step 计时 |
| `tool/call` / `tool/result` | 工具名与命令 |
| `assistant/message` | 模型输出节点 |

所有事件都带 `time`（ms epoch），可据此算相邻事件间隔。

## 验证

- 统计：步数、工具调用次数、单次工具耗时、**最大相邻事件间隔**。
- 判定：若单次工具耗时累加占了绝大部分，而最大间隔很小 → 慢是「N 次串行小往返」叠加，不是一次长思考。反之再去看模型侧。

## 风险与边界

- 日志含本地路径与命令内容，**对外分享前先脱敏**。
- 诊断前先测再下结论：不要凭「感觉慢」直接改配置。

## 回滚

只读分析，无副作用。

## 备注

- 实测样板：一次被中止的 177s turn = 33 步 / 42 次 shell 调用（每次 3–14s 串行），最大相邻事件间隔仅 11.6s。
- 结论直接指向优化方向：减少串行小往返（合并命令、批量操作）比调模型参数更有效。
