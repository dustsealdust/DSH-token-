# DSH Token 调优技能集

DeepSeek Harness（DSH）桌面版的**降本 / 排障 / 环境治理**经验包，拆成 6 个可独立引用的 skill。
每个 skill 都自带「适用场景 → 前置条件 → 操作 → 验证 → 风险与边界 → 回滚 → 备注」结构，可直接复制进你的 skills 目录。

许可证：Apache-2.0。

---

## 技能索引

| 技能 | 解决什么 | 触发时机 |
|---|---|---|
| [`dsh-reasoning-cost`](skills/dsh-reasoning-cost/SKILL.md) | 把推理强度固定到 `low`，压掉大部分无谓思考 token | 成本偏高、想统一降本 |
| [`dsh-profile-plugin-install`](skills/dsh-profile-plugin-install/SKILL.md) | desktop profile 被 GUI 独占时，怎么手工把插件装进去 | 手工 `pnpm add` 报 STORE 类错误 |
| [`dsh-plugin-mount-verify`](skills/dsh-plugin-mount-verify/SKILL.md) | 判断插件到底挂载没有（别看 HTTP 状态码） | 装完插件不确定是否生效 |
| [`dsh-slow-turn-forensics`](skills/dsh-slow-turn-forensics/SKILL.md) | 一次 turn 为什么慢 / 为什么被中止，用会话日志定量取证 | turn 卡顿、超时、被 abort |
| [`dsh-home-migration`](skills/dsh-home-migration/SKILL.md) | 系统盘吃紧导致 `ENOSPC` 时，迁移 `$DSH_HOME` 的正确姿势 | C 盘余量告急、工具写文件失败 |
| [`dsh-skill-install`](skills/dsh-skill-install/SKILL.md) | 给 DSH 加自定义技能：放哪、frontmatter 怎么写、分类归谁管 | 想新增/维护一个自定义 skill |

## 通用红线

1. **改 DSH profile / 系统提示词 / preset，优先走 GUI。** 不要用 shell 硬凑；信息不足就停下交回用户。
2. **`cordis.patch.yml` ≠ 系统提示词。** 它只是 loader 补丁数组，里面没有提示词正文。想「加人设」要动的是 agent preset 那一层。
3. **本包不含脚本代改配置。** 只提供机制、判据和回滚路径，改动由人确认后执行。

## 使用方式

```text
skills/
  dsh-reasoning-cost/SKILL.md
  dsh-profile-plugin-install/SKILL.md
  dsh-plugin-mount-verify/SKILL.md
  dsh-slow-turn-forensics/SKILL.md
  dsh-home-migration/SKILL.md
  dsh-skill-install/SKILL.md
```

- 整包用：把 `skills/` 整个拷进你的 skills 目录。
- 单点用：只拷需要的那一个目录。
- 仓库用法：clone 后按文档路径直接引用，无需构建。

## 备注说明

- **脱敏约定**：本仓库内的路径一律写成 `$DSH_HOME`、`<profile>`、`<本机 pnpm store>` 这类占位符，不含用户名、邮箱、账号、token 与真实绝对路径。引用到别处时请沿用该约定。
- **可移植性**：所有判据都是「看某个文件的落盘时间 / 看某个事件字段」，与具体机器无关；只有占位符需要按本机替换。
- **版本敏感项**：pnpm 报错码与锁文件新鲜度策略可能随版本变化；若报错文本对不上，先按「现象」而不是「错误码」对照。
- **维护**：新增经验请新增 skill 目录，不要往已有 skill 里堆无关内容；本表的索引行与技能目录必须同步。
