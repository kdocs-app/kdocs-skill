# 列表与编号

> 列表与编号

**适用场景**：连续段落设项目符号、阿拉伯编号，或多级列表（含目录行十进制、标题级罗马一级等）。

- 仅悬挂缩进、无列表 → `wps-paragraph-align-indent.md`
**触发词**：项目符号、数字编号、多级列表、目录编号

## 涉及工具

`wps.texts.search`、`wps.texts.range`、`wps.texts.structure`、`wps.texts.replace`、`wps.lists.data`、`wps.lists.apply_with_level`、`wps.lists.multilevel`、`wps.lists.number_format`、`wps.lists.blocks`。见 `references/wps/lists.*`。

## 执行流程

### 场景 1：标题下要点圆点项目符号

适用：某标题下方连续要点加圆点列表。

**执行时由用户指定**：标题字面、要点段范围（通常为标题后连续段）。

1. `wps.texts.search` — 标题。
2. `wps.texts.range` — 标题 `paragraph_index`；要点为 P+1…
3. `wps.lists.apply_with_level` 若 400100 → `wps.lists.data`：首要点 `gallery_type=1`、`template_index=1`；后续 `is_continue=true`。
4. `wps.lists.data` query 代表段 — `list_type` 圆点=2。

### 场景 2：连续段落数字编号

适用：多段改为 1、2、3… 编号列表。

**执行时由用户指定**：各段字面；若须先改文案则先 `wps.texts.replace` 再编号。

1. `wps.texts.search` — 定位各段字面。
2. `wps.lists.data` — 首段 `gallery_type=2`、`template_index=1`；后续 `is_continue=true`。
3. `wps.lists.data` query 代表段 — 编号续接。

### 场景 3：多级列表与编号样式

适用：PRD 式「目录」下手动行改十进制多级；或按 outline_level 分块多级列表且一级用大写罗马数字。

**执行时由用户指定**：目录标题「目录」或 H1 分块；`gallery_type`/`template_index`/`level`；罗马一级用 `wps.lists.number_format`（`number_style=1`）。

**路径 A — 目录区十进制**（非 TOC 域）：

1. `wps.texts.search`「目录」→ `wps.texts.range` 得 P_dir。
2. `wps.texts.structure` — P_dir+1…P_dir+K 推断 level；勿全文 `wps.texts.content`。
3. `wps.lists.apply_with_level` 单次 items 含 index+level。
4. 400100 时：对各段 `wps.lists.multilevel`；勿 `wps.tocs.data`（目录域）。

**路径 B — 标题大纲 + 一级罗马**：

1. `wps.texts.structure` 分批；按 H1 **分块**各一次 `wps.lists.apply_with_level`。
2. 每块内对每个 H1 `wps.lists.number_format`（`number_style=1`）。
3. `wps.lists.data` 抽样 H2/H3 确认 level 与编号样式。
