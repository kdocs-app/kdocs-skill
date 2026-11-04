# 多目标文字样式（检索定位）

> 检索定位后批量改字符样式

**适用场景**：多处离散字符需改色、高亮、字重字形、下划线/删除线/上标等，先用 `wps.texts.search`（或超链接用 `wps.hyperlinks.list`）定位再写入。

- 多个目标词/短语、状态行、超链接外观
- 不按大纲改「所有标题段」→ `references/workflows/wps-heading-by-outline.md`
- 整段对齐、缩进、行距、段边框 → `references/workflows/wps-paragraph-align-indent.md`
**触发词**：改颜色、加高亮、加粗斜体、下划线、删除线、多关键词、清除高亮

## 涉及工具

`wps.texts.search`、`wps.texts.font`、`wps.texts.font_batch`、`wps.texts.font_color`、`wps.texts.highlight`、`wps.texts.underline_color`、`wps.texts.range`、`wps.texts.content`（仅插入前缀等）、`wps.hyperlinks.list`。见 `references/wps/` 对应说明（上表工具）。
## 执行流程

### 场景 1：多命中改字体色

适用：多处离散文本改为同一字体色（标准色或 `rgb(...)` / `color_index`）。

**执行时由用户指定**：`<词或短语列表>`、`<颜色>`。

1. 对每个**不同**目标文本 `wps.texts.search` 一次（该词多处命中时 `is_all=true`）；禁止 `wps.texts.content` 全文代替定位。
2. 对每个命中区间 `wps.texts.font_color` 一次，`begin`/`end` 取该命中。不要为换 `paragraph_index` 调用 `wps.texts.range`。
3. 全部写完后 `wps.texts.font` 对一处代表区间回读。
> 执行本场景所列步骤前，先运行 `kdocs-cli help wps.texts.font wps.texts.font_color wps.texts.search` 获取参数与约束。
### 场景 3：多命中组合字符样式

适用：加粗、斜体、字号、字体色、高亮的任意组合；可选在命中前插入前缀标记。

**执行时由用户指定**：`<find_text>` 或词列表、`is_all`、写入属性集合（`bold` / `italic` / `font_size` / 颜色 / 高亮）、可选 `<前缀字面>`。

1. 对每个目标 `wps.texts.search`（同一关键词全文多处 → `is_all=true`）；勿先 `wps.texts.count`/`wps.texts.content`/`wps.texts.structure` 全文探测。
2. 对每处命中写入。同一区间要同时设置 `bold`、`italic`、`font_size` 时，调用一次 `wps.texts.font_batch`，用 `font_items` 列出各项；不要拆成多次 `wps.texts.font`。不要一次 `font_batch` 提交全部命中：当前参数没有区间数组。颜色与高亮仍分别调用 `wps.texts.font_color`、`wps.texts.highlight`。`begin`/`end` 取该命中，不要为换 `paragraph_index` 调用 `wps.texts.range`。
3. **可选前缀**：若用户要在条款前加标记且段首尚无该标记 → `wps.texts.content` 在命中前插入（0–H 次）。
4. **选区**（prompt 含 `<document_selection>`）：用选区**字面** `wps.texts.search` 一次，须唯一命中；再 `wps.texts.font` / `wps.texts.font_color` 等同场景 3 组合写；0 或多命中须报告。
5. 全部写完后一处代表回读。
> 执行本场景所列步骤前，先运行 `kdocs-cli help wps.texts.content wps.texts.count wps.texts.font wps.texts.font_batch wps.texts.font_color wps.texts.highlight wps.texts.range wps.texts.search wps.texts.structure` 获取参数与约束。
### 场景 5：超链接样式

适用：给已有超链接加下划线、改蓝字与蓝色下划线等。

**执行时由用户指定**：链接识别方式（通常 `wps.hyperlinks.list`）、`underline`、`font_color`、`underline_color`。

1. `wps.hyperlinks.list` 一次得各链接字符区间；勿再用 `wps.texts.search` 重复定位。
2. 单线下划线：每区间 `wps.texts.font`（`underline=1`）。
3. 蓝字+蓝下划线：每区间依次 `wps.texts.font_color` 蓝 → `wps.texts.font` `underline=1` → `wps.texts.underline_color` 蓝。
4. 一处代表回读。
> 执行本场景所列步骤前，先运行 `kdocs-cli help wps.hyperlinks.list wps.texts.font wps.texts.font_color wps.texts.search wps.texts.underline_color` 获取参数与约束。
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
> 执行本场景所列步骤前，先运行 `kdocs-cli help wps.texts.clear_format wps.texts.content wps.texts.count wps.texts.font wps.texts.font_color wps.texts.search wps.texts.structure` 获取参数与约束。
