---
name: dsh-skill-install
description: 给 DSH 新增/维护自定义技能时使用：确定存放位置、SKILL.md frontmatter 写法、分类归属，以及避坑。
---

# DSH 用户级技能安装与维护

## 适用场景

- 想给 DSH 加一个自定义 skill（一条流水线、一份排障手册）。
- 不确定技能放哪、frontmatter 怎么填、分类由谁管。

## 存放位置

- 用户级技能：`$DSH_HOME/skills/<技能名>/SKILL.md`（一个技能一个目录，目录名即技能标识）。
- 典型本机：`DSH_HOME` 指向数据盘，旧盘符路径是指向它的 Junction，可继续读。

## SKILL.md 结构

- YAML frontmatter 必填两项：`name:`（与目录名一致）与 `description:`（写「何时使用」，让加载器知道触发时机）。
- 正文按「适用场景 → 前置条件 → 操作 → 验证 → 风险与边界 → 回滚 → 备注」组织最省心。

## 分类（是逻辑分组，不是文件夹）

- 分类由 skills-manager 插件维护在 `$DSH_HOME/skills-manager/state.json` 的 `categories` / `categoryMembers`（源键 `dsh`）。
- 改分类应在 GUI 里拖拽或交回用户，不要直接手改 state.json。

## 验证

- 放进 `$DSH_HOME/skills/` 后，GUI 技能列表里应能看到该技能及其分类。
- 看不到或名称对不上，多半是 frontmatter 拼写或目录名不一致。

## 风险与边界

- 不要 grep 技能插件打包后的 JS bundle 去找路径：输出巨大且浪费时间；路径直接问用户或看 GUI 显示的根路径。
- 技能是「机制与判据」的载体，不要把一次性进度、临时事实写进 SKILL.md。

## 回滚

删除对应技能目录即可；分类成员随之失效，必要时在 GUI 里清理。

## 备注

- `$DSH_HOME` 迁移会一并带走 `skills/`、`storages/`、`sessions/` 等数据目录，技能无需单独搬运。