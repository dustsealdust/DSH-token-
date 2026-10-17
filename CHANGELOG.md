# 更新记录

本仓库的技能按「目录即版本」维护：每个 skill 一个目录，README 索引表与目录必须同步。

## v1.1.0 — 2026-10-17

- 新增 `dsh-skill-install`：DSH 用户级技能安装与维护（存放位置、frontmatter、分类归谁管、避坑）
- `dsh-home-migration`：补充「迁移会一并带走 skills/storages/sessions」的说明
- README 索引表与目录同步

## v1.0.0 — 2026-10-17

首次整理为可复用技能包，共 6 个 skill：

**DSH 降本 / 排障 / 环境治理（原有 5 个）**

- `dsh-reasoning-cost`：把推理强度固定到 `low`，压掉无谓思考 token
- `dsh-profile-plugin-install`：desktop profile 被 GUI 独占时手工装插件
- `dsh-plugin-mount-verify`：判断插件是否真的挂载（不看 HTTP 状态码）
- `dsh-slow-turn-forensics`：用多帧 zstd 会话日志定量定位慢 turn
- `dsh-home-migration`：`$DSH_HOME` 迁移的正确姿势（C 盘 ENOSPC 场景）

**新增**

- `mod-localization-cn`：Paradox 游戏（EU5 / CK3 / V3 / HOI4 等）Mod 汉化流水线
  —— 覆盖度扫描 → 翻译 → 写入前硬校验 → 追加复核 → 发布封装；
  含 BOM/格式雷区、标记白名单、术语表约定与 Steam 创意工坊发布约定

**文档**

- README 增加技能索引表（技能 / 解决什么 / 触发时机）
- 明确脱敏约定：仓库内路径一律用 `$DSH_HOME`、`<GAME_ROOT>` 等占位符