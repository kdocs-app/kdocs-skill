# 清除格式与字体替换

> 清除格式、全文样式、相对字号、字体替换、西文字体

**适用场景**：清除选区或全文格式/内置样式，或整体、局部替换字体（含保留加粗、标题与正文分别替换、西文字体）。

- 含 `wps.texts.clear_format`、`wps.styles.*`、批量 `wps.texts.font` 替换等
- 按关键词设色、高亮 → 见 `wps-multi-target-search-style.md`
**触发词**：清除格式、全文样式、相对字号、字体替换、西文字体

## 涉及工具

`wps.texts.search`、`wps.texts.clear_format`、`wps.texts.font`、`wps.texts.font_batch`、`wps.texts.count`、`wps.texts.range`、`wps.texts.structure`、`wps.texts.style`、`wps.styles.data`。见 `references/wps/` 对应说明。

## 执行流程

### 场景 1：选区清除全部格式

适用：用户要求把选区内**所有格式清除**、恢复默认样式。

1. `wps.texts.search` 一次：用 `<selected_text>` **字面**搜索；须唯一命中（0 或多命中须报告，不得盲写）。
2. `wps.texts.clear_format` 对命中区间清除全部格式（`scope=ranges`）。
3. 全部写完后 `wps.texts.font` 一次回读验证 bold/字号/颜色已恢复默认。

> 先读 `references/wps/texts.search.md` 与 `references/wps/texts.clear_format.md`。

### 场景 2：清除全文样式

适用：用户要求清除**全文**所有样式（含标题、正文等），统一恢复为默认/正文样式。

1. `wps.texts.count` 一次 — 段落数 N（用于定位末段）。
2. `wps.texts.range` 一次 — P_N 得全文 `range_end`（字符上界）。
3. `wps.texts.clear_format` 一次 — `scope=ranges`，`begin=0`、`end=range_end`，清除手动格式。
4. `wps.styles.data` 一次 — `scope=ranges`，全文套用内置样式「正文」。
5. `wps.texts.style` **1–2 次** — 写后回读代表段（含原标题段），确认 `style_name=正文`、`outline_level=10`。

> 先读 `references/wps/texts.clear_format.md`、`references/wps/texts-clear_format/ranges_update.md`、`references/wps/styles-data/ranges_update.md`。不要逐段 N 次 `wps.styles.data`；验证样式名用 `wps.texts.style`，不要无 `key` 的 `wps.texts.format` query（400100）。

### 场景 3：标题字号相对放大

适用：用户要求把文档中**所有标题段**（outline_level 1–9）字号放大/缩小 N 磅（如「放大 2 号」= +2pt）。

1. 不要 `wps.texts.count` + 逐段 `wps.texts.format` 扫全文（N 段文档 → N 次 format 属错误路径）。
2. **优先** `wps.texts.structure` 分批传入 `paragraph_indexes`（每批 ≤100），筛 outline_level 1–9 得标题列表（H 段）。全文 200+ 段约 2–3 次 structure。
3. structure 返回 400100 时：**勿** fallback 到全文 format；可再试 1–2 批 structure 或稀疏 format 抽样（≤5 次）推断标题分布，仍禁止 224×format。
4. 对每个标题段 `wps.texts.font`（scope=paragraphs）设 `font_size=当前值+N`（H 次写；若不知当前值可先 1 次代表 query 得基准后批量写）。
5. `wps.texts.font` 一次代表回读。

> 先读 `references/wps/texts.structure.md`、`references/wps/texts.format.md`、`references/wps/texts.font.md`。

### 场景 4：全文字体替换（保留加粗）

适用：用户要求把**全文**字体改成指定字体（如微软雅黑），且**保留**标题等原有加粗。

1. `wps.texts.count` 一次得段落数 N。
2. 对 P1…PN 各调用 `wps.texts.font`（scope=paragraphs）**仅**设 `font_name`（如「微软雅黑」）；不要同时写 bold/italic 等（保留原加粗）。
3. `wps.texts.font` 回读 1–2 次（标题段 + 正文段各 1）确认 font_name 与 bold 保留。

> 先读 `references/wps/texts.font.md`、`references/wps/texts.count.md`。

### 场景 5：正文段落字体替换（保留标题）

适用：用户要求把**正文段落**字体改成指定字体（如宋体），**不修改**标题段（outline_level 1–9）。

1. `wps.texts.count` 一次得段落数 N。
2. **优先** `wps.texts.structure` 分批传入 `paragraph_indexes`（每批 ≤100），筛 outline_level=10 得正文段列表 B。若返回 400100，勿重复调用，走步骤 3。
3. structure 不可用时：对 P1…PN 各调用 `wps.texts.format`（paragraphs query）查 outline_level，筛 outline_level=10 归入 B（N 次；无批量 API 时的 fallback）。
4. 对 B 中每段 `wps.texts.font`（scope=paragraphs）**仅**设 `font_name`（如「宋体」）；不要同时写 bold/italic 等。
5. `wps.texts.font` 回读 1–2 次（正文代表段 + 标题代表段各 1）确认正文已改、标题未改（structure 可用：1+1+|B|+2；structure 不可用：1+1+N+|B|+2）。

> 先读 `references/wps/texts.structure.md`、`references/wps/texts.format.md`、`references/wps/texts.font.md`。

### 场景 6：小标题西文字体替换

适用：用户要求把**小标题**西文字体改成指定字体（如 Arial），且不改动正文中文字体。

1. **可选** `wps.texts.count` 得段落数 N。
2. **优先** `wps.texts.structure` 分批查 outline_level 1–9 定位小标题。若返回 400100，勿重复调用。
3. structure 不可用且抽样 `wps.texts.format`（≤3 次）确认 outline 无法区分标题时：不要 format 扫完全文；改用 `wps.texts.font`（paragraphs query）逐段查 `bold`，筛 `bold=true` 为小标题列表 B（N 次 query）。
4. 对 B 中每段 `wps.texts.font`（scope=paragraphs）**仅**设 `font_name`（如「Arial」）；不要同时写 bold/中文字体属性。
5. `wps.texts.font` 回读 1–2 次（小标题代表段 + 正文代表段各 1）。

> 先读 `references/wps/texts.font.md`、`references/wps/texts.structure.md`。
