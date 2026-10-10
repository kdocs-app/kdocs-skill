# 题注、日期域与公式

> 题注、域与公式

**适用场景**：为指定页图/表插入题注（居中、段前距、表格禁跨页），插入签署日期域，或在表下插入公式。
**触发词**：题注、签署日期、公式

## 涉及工具

`wps.images.count`、`wps.images.locate_paragraph`、`wps.images.list`、`wps.tables.count`、`wps.tables.range`、`wps.tables.row_break_across_pages`、`wps.captions.label`、`wps.captions.data`、`wps.texts.search`、`wps.texts.alignment`、`wps.texts.format`、`wps.texts.content`、`wps.texts.range`、`wps.texts.page_number`、`wps.fields.by_range`、`wps.fields.list`、`wps.formulas.math`。见 `references/wps/` 对应说明。

## 执行流程

### 场景 1：图/表题注（含版式）

适用：给指定页或单图/单表加题注，并设图段居中、题注段前距、表格行禁跨页等。

**执行时由用户指定**：页 P 或单对象、`title` 文案、标签「图」/「表」、`SpaceBefore` 磅值、是否 `row_break_across_pages=false`。

**路径 A — 指定页首图**：

1. `wps.images.count`；对 index 循环 `wps.images.locate_paragraph` + `wps.texts.page_number` 至命中页 P 首图即停。
2. `wps.captions.label`（缺则建）→ `wps.captions.data`（`position=1` 下方、`title`）。
3. `wps.texts.alignment` — 题注段居中；`wps.texts.content`/`wps.texts.alignment` 回读。公网若 `wps.images.list` 500002 勿强用。

**路径 B — 单图居中+题注+间距**：

1. `wps.images.count` + `wps.images.locate_paragraph` → 图段 `wps.texts.alignment` 居中。
2. `wps.captions.label` + `wps.captions.data`。
3. `wps.texts.format` — 题注段 `SpaceBefore=N`。

**路径 C — 指定页表格+禁跨页**：

1. `wps.tables.count`/`wps.tables.range` + `wps.texts.page_number` 确认页 P。
2. 各行 `wps.tables.row_break_across_pages` allow=false。
3. `wps.captions.data`（table_index）；题注段为表 end 后段 → `wps.texts.alignment` 居中 + 回读。

### 场景 2：插入日期域

适用：在锚点字面（如「签署日期」）处插入当前日期域。

**执行时由用户指定**：锚点字面、日期格式串（如 `DATE \@ "yyyy年M月d日"`）。

1. `wps.texts.search` — 唯一命中。
2. `wps.texts.content` — 确认标签后尚无日期（可选）。
3. `wps.fields.by_range` — type=31 插入域。
4. `wps.texts.content` + `wps.fields.list` 回读验证。

### 场景 3：表下插入公式

适用：在指定表下方段落插入数学公式且不越页。

**执行时由用户指定**：表锚点字面或页 P、公式 linear 文本。

1. `wps.texts.search` — 表锚点 → `wps.tables.range`。
2. `wps.texts.range` 二分 + `wps.texts.page_number` — 表后首段且在目标页。
3. `wps.formulas.math` insert → query + `wps.texts.page_number` 回读。
