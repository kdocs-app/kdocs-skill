# 节下正文批量格式

> 某节下正文改字号、两端对齐、行距

**适用场景**：用户指定某一章节/小标题下的正文，批量统一字号、两端对齐或行距。

- 先 `wps.texts.search` 锚定节标题，再对该节正文段落批量 `wps.texts.format` / `wps.texts.font` / `wps.texts.line_spacing`
- 单段缩进、边框、底纹 → 见 `wps-paragraph-align-indent.md`
**触发词**：某节下正文改字号、两端对齐、行距

## 涉及工具

`wps.texts.search`、`wps.texts.range`、`wps.texts.font`、`wps.texts.alignment`、`wps.texts.line_spacing`、`wps.texts.format`。见 `references/wps/texts.*`。

## 执行流程

### 场景 1：节下正文字号批量设置

适用：用户要求把某节标题（如「1 背景」）**下方正文段落**字号统一改成指定字号（如小四=12pt）。

1. `wps.texts.search` 一次：定位节标题得 `paragraph_index`（如 P3）。
2. 不要 `count` + 逐段 `content` 扫全文；正文段 = 标题后连续 outline_level=10 的段落直至下一同级/更高级标题（H 段）。
3. 对每正文段 `wps.texts.font` 设 `font_size=12`（H 次，scope=paragraphs）。
4. `wps.texts.font` 一次代表回读。

> 先读 `references/wps/texts.search.md`、`references/wps/texts.format.md`、`references/wps/texts.font.md`。

### 场景 2：节下正文两端对齐

适用：用户要求把某节标题（如「1 背景」）**下方正文段落**设为两端对齐。

1. `wps.texts.search` 一次：定位节标题得字符坐标。
2. `wps.texts.range` 一次：将 search 命中映射为 `paragraph_index`（标题段）。
3. 不要 `count` + 逐段 `content` 扫全文；正文段 = 标题后连续 outline_level=10 的段落直至下一节标题（可用 一次 `wps.texts.format` query 邻段推断边界，不要 format 扫 P10 以降全文）。
4. 对每正文段 `wps.texts.alignment` 设 `alignment=justify`（数值 3，H 次）。
5. `wps.texts.alignment` 一次代表回读。

> 先读 `references/wps/texts.search.md`、`references/wps/texts.range.md`、`references/wps/texts.format.md`、`references/wps/texts.alignment.md`。

### 场景 3：节下正文行距设置

适用：用户要求把某节标题（如「1 背景」）**下方正文段落**设为指定行距（单倍/1.5 倍/双倍等）。

1. `wps.texts.search` 一次：定位节标题得字符坐标。
2. `wps.texts.search` 一次：定位**下一节**标题（如「2 目标」）得正文区间上界字符坐标。
3. `wps.texts.line_spacing` `scope=ranges`：对正文字符区间设 `spacing_rule`（单倍=0、1.5 倍=1、双倍=2，见 `enums.md`）。不要 `structure`（400100）、逐段 `format` 扫全文、逐段 `line_spacing`。
4. `wps.texts.format` 一次回读代表段 `line_spacing`（。

> 先读 `references/wps/texts.search.md`、`references/wps/texts.line_spacing.md`、`references/wps/texts.format.md`、`references/wps/enums.md`。
