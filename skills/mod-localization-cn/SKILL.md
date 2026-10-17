---
name: mod-localization-cn
description: 汉化 Paradox 游戏（EU5 / CK3 / V3 / HOI4 等）的 Mod：定位基座原文 → 覆盖度扫描 → 翻译 → 写入前硬校验 → 追加复核 → 发布封装。
---

# Paradox Mod 汉化流水线

## 适用场景

- 要给某个 Mod 做中文汉化，或给已有汉化包补漏翻的键。
- 汉化「突然整包失效」，怀疑是文件编码/格式问题。
- 要把汉化包发布到 Steam 创意工坊。

主流程：**英文原文比对 → 用户/译员定稿 → 追加 → 校验 → 发布封装**。
两个必过关卡：**① 覆盖度扫描** 与 **② 写入前硬校验**。

## 前置条件

- 游戏本体目录 `<GAME_ROOT>`；Steam 库 `<STEAM_LIB>`。
- 用户数据目录 `<USER_DATA>`（EU5 为 `D:\Paradox Interactive\Europa Universalis V`，含 `mod\`、`playsets.json`、`logs\`）。
- 基座模组（英文原文）通常来自创意工坊：`<STEAM_LIB>\steamapps\workshop\content\<appid>\<publishedFileId>\main_menu\localization\english\*_l_english.yml`；EU5 的 appid = `3450310`。
- 找不到基座时：查 `<GAME_ROOT>\game\mod\*.mod` 里的 `path=`，以及 `<STEAM_LIB>\steamapps\appmanifest_*.acf` 的 `name` 反查 appid。

## 铁律

1. 中文只放 `main_menu\localization\simp_chinese\...`；写进 `*_english.yml` 不生效（除非走 `localization\replace\<语言>\`）。
2. 本地化 yml 必须 **UTF-8 with BOM**（首三字节 `EF BB BF`）。无 BOM → 该文件整份不被识别 → 汉化「突然全失效」。
3. 一条键必须**单行**；换行写 `\n`（分段 `\n\n`），**引号内绝不允许真实换行**。
4. 键名与标记一律不改：` key:0 "译文"`（1 空格缩进）。
5. 游戏配置/清单类文件（`playsets.json` 等）不由助手改；只改 Mod 自身的 yml 与元数据。

## 操作步骤

### 1. 关卡一：覆盖度扫描

目标：**基座英文键集 − 全部中文源 = 缺失键清单**（输出 `key<TAB>english` 的 TSV）。
「全部中文源」必须包含：游戏本体、所有 `game\mod` 汉化包、工坊所有中文 Mod——只看自己这个汉化包会严重误判。

```powershell
$base='<工坊模组根>'; $game='<GAME_ROOT>'
$en=@{}   # 英文键 -> 原文
foreach($f in (Get-ChildItem -LiteralPath $base -Recurse -File -Filter *.yml |
               Where-Object { $_.FullName -match '\\english\\' })){
  foreach($l in (Get-Content -LiteralPath $f.FullName -Encoding UTF8)){
    if($l -match '^\s*([A-Za-z0-9_.\-]+)\s*:\s*0?\s*"(.*)"\s*$'){
      if(-not $en.ContainsKey($matches[1])){ $en[$matches[1]]=$matches[2] }
    }
  }
}
$zh=@{}   # 已被覆盖的键
foreach($root in @("$game\game","$game\game\mod","<STEAM_LIB>\steamapps\workshop\content\<appid>")){
  Get-ChildItem -LiteralPath $root -Recurse -File -Filter *.yml -ErrorAction SilentlyContinue |
    Where-Object { $_.FullName -match 'simp_chinese' -or $_.Name -match '_l_simp_chinese\.yml$' } |
    ForEach-Object {
      foreach($l in (Get-Content -LiteralPath $_.FullName -Encoding UTF8)){
        if($l -match '^\s*([A-Za-z0-9_.\-]+)\s*:\s*0?\s*"'){ $zh[$matches[1]]=1 }
      }
    }
}
$miss = $en.Keys | Where-Object { -not $zh.ContainsKey($_) } | Sort-Object
"EN=$($en.Count) ZH=$($zh.Count) MISSING=$($miss.Count)"
$miss | ForEach-Object { "$_`t" + $en[$_] } | Set-Content -LiteralPath '<输出>\missing_keys.tsv' -Encoding UTF8
```

把 TSV 交给人（用户/译员）**定稿**——译名不自行拍板。若对方给的键表来源不明，先拿英文键集交叉验证「这些键是否真实存在」，再动笔。

### 2. 翻译约定

- 先建术语表，多轮保持一致。EU5 示例：`conversion` 皈依 / `communal_conversion` 公共皈依 / `voluntary_conversion` 自愿皈依 / `forced_conversion` 强制皈依 / `heathens` 异教徒 / `heretics` 异端 / `dhimmi_estate` 齐米阶层 / `benefice` 圣俸 / `tolerance_law` 宽容法令 / `mortmain` 永久管业 / `witness_doctrine` 见证教义 / `predestination` 预定论 / `justification` 称义 / `ecclesial` 教会论 / `eschatological` 末世论。
- 文风：官方简体中文，庄重简练；引号用「」。
- **标记白名单（逐字保留）**：`[Function('id')]`、`[x|e]`、`[SCOPE.sCountry('actor').GetName]`、`[ROOT.…]`、`#color_green … #!`、`@icon!`、`\n`、`$var$`。
- 原文若整条只是标记（如 `[ROOT.GetLocation.Custom('x')]`），原样保留。

### 3. 关卡二：写入前硬校验

| 检查项 | 判据 |
|---|---|
| 编码 | UTF-8 **with BOM**（`EF BB BF`） |
| 格式 | `^\s*[A-Za-z0-9_.\-]+:0 ".*"$`，键前 1 空格 |
| 单行 | 引号内无真实换行（用 `\n`） |
| 查重 | 新键与文件内既有键零冲突；一旦冲突立即中止，不覆盖 |
| 标记 | 与英文原文逐条比对：标记数量、括号闭合、`#color… #!` 成对 |
| 空译 | 无空字符串（原文为空除外） |

写盘时用 `New-Object System.Text.UTF8Encoding($true)`——**`UTF8Encoding($false)` 会静默抹掉 BOM**，这是本流程最大的坑。

### 4. 追加与复核

追加后立刻复核：首三字节 = `EF BB BF`、行数增量 = 新增条数、畸形行 = 0（语言头行除外）、唯一键数、无重复；再抽查 2–3 条与英文原文对照。

### 5. 发布封装（Steam 创意工坊）

- 上传用**用户数据目录**那份副本（`<USER_DATA>\mod\<模组名>\`）；`game\mod` 那份仅本地测试。
- `descriptor.mod`：`name=` / `tags={ "Translation" }` / `supported_version="1.*.*"`（兼容游戏版本范围，**不是 Mod 版本**）。
- `.metadata\metadata.json`：`id` 用无空格/无方括号的小写 slug；`version` 是 Mod 版本（如 `1.0.0`）；`supported_game_version` 是目标游戏版本；`tags` 含 `Translation`；`short_description` 必填。
- `.metadata\thumbnail.png`：**必须是真 PNG**（JPEG 改后缀会失效）。
- **同名两份 Mod 只能启用一份**，否则键重复打架。

## 验证

- 覆盖度扫描前后：`EN` / `ZH` / `MISSING` 三个数字变化符合预期。
- 追加前后：文件字节数与行数增量 = 新增条数 × 每行长度量级；`MISSING` 归零。
- 进游戏实测：目标界面不再出现英文或原始键名。

## 风险与边界

- 误删 BOM = 整包汉化失效（影响面远大于单条错误）。
- 改写超出「Mod 自身 yml / 元数据」范围（游戏配置、playsets）越权。
- 把不同版本 Mod 的译文硬套进来，会造成「键在但英文对不上」的错配。

## 回滚

- 改动前先备份目标 yml（复制一份 `.bak`）。
- 发布前保留一份可用的旧版副本，出问题直接换回。

## 备注

- 历史坑：① 去 BOM 导致汉化整份失效；② 引号内真实换行使该条解析失败；③ 中文写进 `*_english.yml`；④ 只对比自己这个汉化包就算缺键；⑤ 用户给的键表来自别的版本；⑥ 全盘搜键名过慢（先用 appid / 工坊目录缩小范围）。
- 老式 Python 核对脚本（`l_english` 与 `l_simp_chinese` 键差集）仍可用：原文有而无译 = 漏翻，有译而原文无 = 多余，空译单独报。