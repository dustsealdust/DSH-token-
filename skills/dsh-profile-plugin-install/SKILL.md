---
name: dsh-profile-plugin-install
description: 在 desktop profile 被 GUI 插件管理器独占的情况下，手工用 pnpm 把插件装进 profile 并接入 bundles。
---

# desktop profile 手工装插件

## 适用场景

- 目标包在 npm / GitHub 上存在，但不想或不能通过 GUI 市场安装。
- 手工 `pnpm add` 报 `ERR_PNPM_UNEXPECTED_STORE` / `UNEXPECTED_VIRTUAL_STORE`，或报锁文件新鲜度相关错误。

## 前置条件

- 已知 profile 目录、本机 pnpm store 路径、profile 的 `node_modules/.pnpm` 路径。
- 已 `git status` / 备份 profile 的 `package.json` 与锁文件。

## 背景机制

profile 的 `dsh.profile.bundles` 与 `pnpm-lock.yaml` 由 App 插件管理器 / GUI 市场独占：

- 外部 DSH CLI 会**明确拒绝**管理 desktop profile。
- 市场安装会**整份重写**这两个文件，覆盖外部直接编辑。

所以要手工装，命令必须补齐两个前缀：

```bash
pnpm add <pkg> \
  --store-dir <本机 pnpm store 绝对路径> \
  --virtual-store-dir <profile>/node_modules/.pnpm \
  --config.minimumReleaseAge=0
```

- **缺 store / virtual-store 前缀**：报 `ERR_PNPM_UNEXPECTED_STORE` / `UNEXPECTED_VIRTUAL_STORE`，因为 `node_modules` 里记录的仍是旧路径（典型情形是 `$DSH_HOME` 迁移过、原路径成了联接）。
- **缺 `--config.minimumReleaseAge=0`**：锁文件里若存在处于 24h 新鲜度封锁期内的依赖，安装会被拦；`pnpm-workspace.yaml` 的 `minimumReleaseAgeExclude` 条目压不住它。

装完要**同时**写进 `dependencies` 和 `dsh.profile.bundles`。

## 验证

转交 `dsh-plugin-mount-verify`：看对应插件的 storages 文件是否被新建 / 刷新。

## 风险与边界

- GUI 市场后续安装仍可能重写 `bundles` 与锁文件，覆盖你的手工改动 —— 手工装完记得记录，便于复现。
- 不要在 profile 里装与 DSH 运行时重复版本的核心依赖。

## 回滚

从 `dependencies` 与 `dsh.profile.bundles` 移除该行，删除对应 `node_modules` 目录，或直接还原备份的 `package.json` + 锁文件。

## 备注

- 迁移过 `$DSH_HOME` 的机器尤其容易踩 store 路径坑，命令里的两个路径前缀是**硬性**的。
- GitHub 源包（npm 上没有）同样适用，安装时给完整仓库地址。
