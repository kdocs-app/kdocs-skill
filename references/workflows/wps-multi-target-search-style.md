# 多目标文字样式（检索定位）

> 检索定位后批量改字符样式

**适用场景**：多处离散字符需改色、高亮、字重字形、下划线/删除线/上标等，先用 `wps.texts.search`（或超链接用 `wps.hyperlinks.list`）定位再写入。

- 多个目标词/短语、状态行、超链接外观
- 不按大纲改「所有标题段」→ `wps-heading-by-outline.md`
- 整段对齐、缩进、行距、段边框 → `wps-paragraph-align-indent.md`
**触发词**：改颜色、加高亮、加粗斜体、下划线、删除线、多关键词、清除高亮

## 涉及工具

`wps.texts.search`、`wps.texts.font`、`wps.texts.font_color`、`wps.texts.highlight`、`wps.texts.underline_color`、`wps.texts.range`、`wps.texts.content`（仅插入前缀等）、`wps.hyperlinks.list`。参数见 `references/wps/` 对应说明。

## 执行流程

### 场景 1：多命中改字体色

适用：多处离散文本改为同一字体色（标准色或 `rgb(...)` / `color_index`）。

**执行时由用户指定**：`<词或短语列表>`、`<颜色>`。

1. 对每个**不同**目标文本 `wps.texts.search` 一次（该词多处命中时 `is_all=true`）；禁止 `wps.texts.content` 全文代替定位。
2. 对每个命中区间 `wps.texts.font_color` 设 `<颜色>`。
3. 全部写完后 `wps.texts.font` 对一处代表区间回读。

### 场景 2：多命中改高亮

适用：多个关键词或条目加高亮；或按状态字面定位后**整段/整行**高亮。

**执行时由用户指定**：`<关键词或条目列表>`、`<highlight_color>`；整行时 `<状态或锚点字面>`。

**路径 A — 字符区间**（多处同一词）：

1. 每个不同关键词 `wps.texts.search`（`is_all=true`）→ 各区间 `wps.texts.highlight`。

**路径 B — 命中所在段落**（决议/行动项等条目、或「重点块」）：

1. 每条条目文本 `wps.texts.search` 一次；勿 `wps.texts.content` 全文或 P1…PN 逐段 `wps.texts.range` 扫映射。
2. `wps.texts.highlight`（`scope=paragraphs`，`paragraph_index` 来自命中）。

**路径 C — 状态行**（如含「状态：已完成」的任务行）：

1. `wps.texts.search`（状态字面，`is_all=true`）→ `wps.texts.range` 取命中段落完整字符区间 → `wps.texts.highlight` 设 `<highlight_color>`。

全部写完后一处代表回读。

### 场景 3：多命中组合字符样式

适用：加粗、斜体、字号、字体色、高亮的任意组合；可选在命中前插入前缀标记。

**执行时由用户指定**：`<find_text>` 或词列表、`is_all`、写入属性集合（`bold` / `italic` / `font_size` / 颜色 / 高亮）、可选 `<前缀字面>`。

1. 对每个目标 `wps.texts.search`（同一关键词全文多处 → `is_all=true`）；勿先 `wps.texts.count`/`wps.texts.content`/`wps.texts.structure` 全文探测。
2. 对每处命中按用户要求依次写入（同工具可合并：`wps.texts.font` 一次写 `bold`+`italic`+`font_size`；再 `wps.texts.font_color`；再 `wps.texts.highlight`）。颜色与高亮组合顺序：先 `font_color` 再 `highlight` 或按用户指定。
3. **可选前缀**：若用户要在条款前加标记且段首尚无该标记 → `wps.texts.content` 在命中前插入（0–H 次）。
4. **选区**（prompt 含 `<document_selection>`）：用选区**字面** `wps.texts.search` 一次，须唯一命中；再 `wps.texts.font` / `wps.texts.font_color` 等同场景 3 组合写；0 或多命中须报告。
5. 全部写完后一处代表回读。

### 场景 4：下划线类

适用：单线/波浪线下划线、下划线颜色、去掉下划线并与字色对齐；`_…_` 标记区间加粗。

**执行时由用户指定**：目标短语或 `_` 配对规则、`underline` 样式、`underline_color`、是否仅改可见短语列表。

**路径 A — 红色等单色下划线**（合同日期、金额等多处）：

1. 每个目标短语 `wps.texts.search` → `wps.texts.font`（`underline=1`）→ `wps.texts.underline_color`（直接写，勿 query）。

**路径 B — 波浪线**（错误内容标记）：

1. 错误短语字面 `wps.texts.search`（唯一命中）→ `wps.texts.font`（`underline=11`）→ `wps.texts.underline_color` 设红。

**路径 C — 去黑线、红字红线下划线对齐**：

1. 对每个待改**可见短语** `wps.texts.search`（勿搜颜色形容词字面）；黑线区间 `wps.texts.font` `underline=0`；红字精确区间 `underline=1` + `wps.texts.underline_color` 红（区间不含后续标点/黑字）。

**路径 D — `_…_` 加粗**：

1. `wps.texts.search`（`find_text="_"`, `is_all=true`）；本地配对相邻 `_`（间距≤30）为标记区间，孤立 `_` 跳过 → 各区间 `wps.texts.font` `bold=true`。

全部写完后一处代表回读。

### 场景 5：超链接样式

适用：给已有超链接加下划线、改蓝字与蓝色下划线等。

**执行时由用户指定**：链接识别方式（通常 `wps.hyperlinks.list`）、`underline`、`font_color`、`underline_color`。

1. `wps.hyperlinks.list` 一次得各链接字符区间；勿再用 `wps.texts.search` 重复定位。
2. 单线下划线：每区间 `wps.texts.font`（`underline=1`）。
3. 蓝字+蓝下划线：每区间依次 `wps.texts.font_color` 蓝 → `wps.texts.font` `underline=1` → `wps.texts.underline_color` 蓝。
4. 一处代表回读。

### 场景 6：删除线

适用：条款或段落加删除线（含双删除线表废弃）。

**执行时由用户指定**：`<锚点字面>`（如条款编号）、是否扩至整段。

1. `wps.texts.search` 定位锚点；若用户要整段废弃且 search 仅命中编号 → `wps.texts.range` 取段落完整区间。
2. `wps.texts.font`：`strike_through` 或 `double_strike_through=true`。
3. 一处回读。

### 场景 7：选区与单点字形

适用：`<document_selection>` 去色保留加粗、斜体、上标；或去掉正文误加粗而保留标题加粗。

**执行时由用户指定**：`<selected_text>` 或表达式字面、`superscript` / `italic` / 去色+保留 `bold`；正文段抽样索引（取消误加粗时）。

**路径 A — 选区去色保留加粗**：

1. 选区字面 `wps.texts.search`（唯一）→ `wps.texts.font_color` 默认黑/automatic → 同区间 `wps.texts.font` `bold=true`（勿 `wps.texts.clear_format`）。

**路径 B — 选区斜体等单属性**：

1. 选区字面 `wps.texts.search`（唯一）→ `wps.texts.font` 写 `italic=true`（或用户指定属性）。

**路径 C — 上标**：

1. `wps.texts.search` 目标表达式或指数部分字面；若 500002 则按 `wps.texts.content` / 返回区间再定位指数字符 → `wps.texts.font` `superscript=true`。

**路径 D — 取消非重点加粗**：

1. 勿 `wps.texts.count`/`wps.texts.structure`/`wps.texts.content` 全文探测；对若干**正文段** `wps.texts.font` query `bold`，仍为 true 则写 `bold=false`；代表标题段回读确认仍 bold。

### 场景 8：清除高亮

适用：全文或大范围去掉高亮，保留加粗/颜色等其他格式。

**执行时由用户指定**：全文 vs 按 `wps.texts.search` 抽验的代表词。

1. 勿 `wps.texts.clear_format`（会清掉非高亮格式）；勿 `wps.texts.content` 全文拉取。
2. `wps.texts.count` 得段落数 N；对每段 `wps.texts.font`（`scope=paragraphs`, query）读 `highlight_color`，必要时对段落 ranges 再 query。
3. 仍带高亮的区间 `wps.texts.highlight`（`scope=ranges`，`highlight_color=0`）。
4. 可选：对已知标题字面 `wps.texts.search` + `wps.texts.font` 抽样确认无高亮。
