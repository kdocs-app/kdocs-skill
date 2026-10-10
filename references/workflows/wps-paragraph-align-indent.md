# 段落对齐、缩进与段边框

> 右对齐、首行缩进、悬挂缩进、行距、段前距、段落边框、底纹

**适用场景**：调整段落版式：对齐（含落款右对齐）、首行/左/悬挂缩进、固定行距、段前距、段落边框或底纹。

- 多先 `wps.texts.search` 锚定段落或已有 `ranges`，再调用 `wps.texts.alignment` / `wps.texts.indent` / `wps.texts.line_spacing` / `wps.texts.border` / `wps.texts.shading` 等
- 某节标题下正文批量字号/对齐/行距 → 见 `wps-section-body-format.md`
- 页眉页脚与页码 → 见 `wps-header-footer-pagination.md`

**触发词**：右对齐、首行缩进、悬挂缩进、行距、段前距、段落边框、底纹

## 涉及工具

`wps.texts.search`、`wps.texts.alignment`、`wps.texts.indent`、`wps.texts.line_spacing`、`wps.texts.format`、`wps.texts.border`、`wps.texts.shading`、`wps.texts.range`、`wps.texts.count`。见 `references/wps/texts.*`。

## 执行流程

### 场景 1：落款日期右对齐

适用：用户要求将落款/签署日期段落右对齐。

1. `wps.texts.search` 一次：`find_text` 用「签署日期」或 prompt 给出的完整日期字面；须唯一命中得段落坐标。
2. `wps.texts.alignment` 对命中段设 `alignment=right`（数值 2）。
3. `wps.texts.alignment` 一次回读验证（。

> 不要 `wps.texts.count`、拆字搜索（月/日/年等单字试探）、`wps.texts.content` 全文概览。先读 `references/wps/texts.search.md` 与 `references/wps/texts.alignment.md`。

### 场景 2：选区首行缩进

适用：prompt 含 `<document_selection>`，用户要求把选区所在段落设首行缩进 N 字符（或磅）。

1. `wps.texts.search` 一次：用 `<selected_text>` **字面**搜索；须唯一命中。
2. `wps.texts.range`：从 P1 起顺序查询，**命中即停**（`range_begin ≤ search.begin < range_end` 得 `paragraph_index`）；不要 count + content 扫全文。
3. `wps.texts.indent` 对命中段设 `indent_type="3"`、`unit="2"`、`indent_value=N`（字符）。
4. `wps.texts.format` 一次回读 `first_line_indent`（。

> 先读 `references/wps/texts.search.md`、`references/wps/texts.range.md`、`references/wps/texts.indent.md`、`references/wps/texts.format.md`。`indent_type`/`unit` 须字符串（"3"/"2"），勿用 first_line/char 等别名。

### 场景 3：编号列表悬挂缩进

适用：用户要求把编号列表（字面编号如 `1.`、`2.` 或 `1）`）设悬挂缩进。

1. `wps.texts.search` + `wps.texts.range` 定位各编号项段落（`is_all` 或逐编号字面）；不要 `wps.lists.blocks`（400100）、`wps.texts.count`、`wps.texts.content` 全文。
2. 对每个编号段**两次** `wps.texts.indent`：先首行 `indent_type="3"`、`unit="2"`、`indent_value=-2`；再左缩进 `indent_type="1"`、`unit="2"`、`indent_value=2`（悬挂 = 负首行 + 正左缩进）。
3. `wps.texts.format` 一次回读代表段 `first_line_indent`/`left_indent`。

> 先读 `references/wps/texts.search.md`、`references/wps/texts.indent.md`、`references/wps/texts.format.md`。每段须两次 indent 写（API 单次仅一种 indent_type）。

### 场景 4：段落左缩进（厘米）

适用：用户要求把某段（如引用段、正文段）左缩进 N 厘米。

1. `wps.texts.search` 一次：用该段 **字面**（引用块首句、邻近标题下首段等）定位；须唯一命中。不要 `wps.texts.count`、`wps.texts.structure`（400100）、`wps.texts.content` 全文、`wps.styles.list` 试探。
2. `wps.texts.indent` 对命中段设左缩进：`indent_type="1"`、`unit="1"`（磅）、`indent_value=N×28.35`（如 2cm≈56.7）。
3. `wps.texts.format` 一次回读 `left_indent`（。

> 先读 `references/wps/texts.search.md`、`references/wps/texts.indent.md`、`references/wps/texts.format.md`。`unit` 仅磅/字符，厘米须换算磅。

### 场景 5：选区固定行距

适用：prompt 含 `<document_selection>`，用户要求把选区段落行距改为**固定磅值**（如 28 磅）。

1. `wps.texts.search` 一次：`<selected_text>` 字面唯一命中。
2. `wps.texts.range`：映射 `paragraph_index`（命中即停）。
3. `wps.texts.line_spacing`：`spacing_rule="4"`（固定值）、`spacing_value=N`（磅，如 28）。
4. `wps.texts.format` 一次回读 `line_spacing`（。

> 节下多段倍率行距见「节下正文行距设置」。先读 `references/wps/texts.line_spacing.md`、`references/wps/enums.md`。

### 场景 6：选区段前间距

适用：prompt 含 `<document_selection>`，用户要求把选区所在段落段前间距设为 N 磅（或段后间距，用 `SpaceAfter`）。

1. `wps.texts.search` 一次：`<selected_text>` 字面唯一命中。
2. `wps.texts.range`：从 P1 起顺序查询，**命中即停**（`range_begin ≤ search.begin < range_end` 得 `paragraph_index`）。
3. `wps.texts.format` paragraphs update：`key=SpaceBefore`（段后则 `SpaceAfter`）、`value=N`（磅，字符串）。
4. `wps.texts.format` paragraphs query 一次回读 `space_before`（。

> 先读 `references/wps/texts.search.md`、`references/wps/texts.range.md`、`references/wps/texts.format.md`、`references/wps/texts-format/paragraphs_update.md`。无独立 `paragraph_spacing` 工具，不要 `wps.texts.line_spacing` 冒充段间距。

### 场景 7：选区段落边框

适用：prompt 含 `<document_selection>`，用户要求给选区所在段落加边框（四边单线、双线、虚线等）。

1. `wps.texts.search` 一次：`<selected_text>` 字面唯一命中（0 或多命中须报告，不得盲写）。
2. **优先** `wps.texts.count` 一次得段落数 N。
3. `wps.texts.range` 映射 `paragraph_index`（`range_begin ≤ search.begin < range_end`）：
 - **N>10**：对 `[1,N]` **二分**查 mid 段，比较区间缩边界，约 ⌈log₂N⌉ 次；不要从 P1 顺序扫到 P21+（长文档线性扫描属错误路径）。
 - **N≤10**：从 P1 顺序查，**命中即停**。
4. `wps.texts.border` update：`border_style.all` 设样式（单线 `line_style=1`；**双线红色** `line_style=7` + `color=FF0000`；虚线=3）；不要 `wps.texts.char_border`（字符边框非段落边框）。
5. 写后 `wps.texts.border` query 回读四边 `border_info`。

> 先读 `references/wps/texts.search.md`、`references/wps/texts.range.md`、`references/wps/texts.border.md`、`references/wps/texts-border/paragraphs_update.md`、`references/wps/enums.md`（WdLineStyle）。

### 场景 8：段落浅灰底纹

适用：用户要求给含某段字面（如「此处用于测试」）的**段落**加浅灰色底纹（段落 shading，**非** highlight）。

1. `wps.texts.search` 一次：用 prompt 中的**目标字面**搜索；须唯一命中。
2. **优先** `wps.texts.count` 一次得段落数 N。
3. `wps.texts.range` 映射 `paragraph_index`（同「选区段落边框」：N>10 二分，N≤10 顺序命中即停）。
4. `wps.texts.shading` update：`key=BackgroundPatternColorIndex`、`value=16`（wdGray25 浅灰）；不要 `wps.texts.highlight`（高亮非底纹）。
5. 写后 `wps.texts.shading` query 回读 `background_pattern_color_index`。

> 先读 `references/wps/texts.search.md`、`references/wps/texts.range.md`、`references/wps/texts.shading.md`、`references/wps/texts-shading/paragraphs_update.md`。
