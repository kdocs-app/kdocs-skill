# 内置样式与批量 ranges

> 套用标题1、清除样式、全文间距字号

**适用场景**：在指定页或全文范围套用内置样式名，或通过 ranges 批量加大字号、字符间距。

- `wps.styles.data` 与段落/ranges 写入配合
- 清除全文样式、字体族替换 → 见 `references/workflows/wps-clear-and-replace-font.md`
**触发词**：套用标题1、清除样式、全文间距字号

## 涉及工具

`wps.texts.search`、`wps.texts.page_number`、`wps.texts.range`、`wps.texts.style`、`wps.texts.count`、`wps.texts.structure`、`wps.texts.format`、`wps.texts.font`、`wps.texts.line_spacing`、`wps.styles.data`。见 `references/wps/` 对应说明。

## 执行流程

### 场景 1：指定页段落套用内置样式

适用：用户要求把**第 N 页**某段文字设为指定内置样式（如「把第三页"第1章 背景"设为"标题1"样式，其余内容样式不动」）。

1. `wps.texts.search` 一次 — prompt 中**目标段字面**（如「第1章 背景」）；须唯一命中。
2. `wps.texts.range`：用候选 `paragraph_index` 查该段字符区间；当 `range_begin ≤ search 的 begin < range_end` 时，该 `paragraph_index` 即目标段，**立即停止**，勿再查其它段。优先二分缩小候选，**勿**从 1 线性逐段 scan；**勿**把固定查询次数（如 ⌈log₂N⌉）当成必须完成的调用次数。
3. `wps.texts.page_number` 一次 — 确认 `paragraph_index` 在 prompt 指定页（如 page=3）。
4. `wps.texts.style` 一次 — 写前回读。
5. `wps.styles.data` 一次 — `scope=paragraphs`，`style_name`=目标样式（内置名如「标题 1」「标题 2」「正文」）。
6. `wps.texts.style` 一次 — 写后回读 `style_name` / `outline_level`。
7. （可选）邻段 `wps.texts.style` 抽检 1–2 段 — 确认「其余不动」。

> 执行本场景所列步骤前，先运行 `kdocs-cli help wps.styles.data wps.styles.list wps.texts.count wps.texts.page_number wps.texts.range wps.texts.search wps.texts.style` 获取参数与约束。禁止 `wps.styles.definition` 试探、`wps.texts.content` 全文扫段。

### 场景 2：全文间距+字号加大（ranges 批量）

适用：用户要求全文「间距再大一点，字体也大一点」（无选区、无指定节）。

1. `wps.texts.count` 一次 — 段落数 N。
2. `wps.texts.structure` 一次 — 批量 `paragraph_indexes` 筛 outline_level=1 标题 vs 正文段，得标题/正文字符区间（或配合 `wps.texts.range` 取 begin/end）。
3. `wps.texts.format` / `wps.texts.font` **各一次** — 代表段基线 query。
4. `wps.texts.font` + `wps.texts.line_spacing` **各一次** — `scope=ranges` 对正文字符区间加大字号与行距（如 12→14pt、1.5 倍→双倍）；标题区间单独写一次。
5. `wps.texts.format` 一次 — 代表正文/标题回读。

> 不要逐段 `wps.texts.font`×N + `wps.texts.line_spacing`×N。执行本场景所列步骤前，先运行 `kdocs-cli help wps.texts.count wps.texts.font wps.texts.format wps.texts.line_spacing wps.texts.range wps.texts.structure` 获取参数与约束。
