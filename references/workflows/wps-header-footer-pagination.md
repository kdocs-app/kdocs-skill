# 页眉页脚与页码

> 页眉页脚与页码

**适用场景**：写入或删除页眉，在页脚插入/调整页码（阿拉伯、「第 X 页」、罗马数字）及页眉页脚对齐。

- 正文段落对齐 → `wps-paragraph-align-indent.md`

**触发词**：页眉、页脚、页码、罗马数字、删除页眉

## 涉及工具

`wps.header_footers.header_content`、`wps.header_footers.header_alignment`、`wps.header_footers.header_remove`、`wps.header_footers.footer_page_number`、`wps.header_footers.footer_page_number_custom`、`wps.header_footers.footer_content`、`wps.header_footers.footer_alignment`、`wps.header_footers.page_number_style`。见 `references/wps/header_footers.*`。

## 执行流程

### 场景 1：页眉内容与对齐

适用：页眉写入指定文字并设置对齐（居中/左/右）。

**执行时由用户指定**：页眉文案、`alignment`（如 center=1）。

1. `wps.header_footers.header_content` update — 文案。
2. `wps.header_footers.header_alignment` update — 对齐（勿 alignment query，常 400100）。
3. `wps.header_footers.header_content` query — 回读文字。

### 场景 2：页脚页码（格式可配）

适用：页脚插入或调整页码：纯数字居中、「第 X 页」前后缀、或改为罗马数字等。

**执行时由用户指定**：`wps.header_footers.footer_page_number` vs `wps.header_footers.footer_page_number_custom`；罗马数字用 `wps.header_footers.page_number_style`。

**路径 A — 标准页码**：`wps.header_footers.footer_page_number` update → `wps.header_footers.footer_content` query。

**路径 B — 「第X页」**：`wps.header_footers.footer_page_number_custom` → `wps.header_footers.footer_content` query（勿 footer_page_number_custom query）。

**路径 C — 改罗马数字**：`wps.header_footers.footer_content` query → `wps.header_footers.page_number_style` update → `wps.header_footers.footer_content` query。

**路径 D — 页码右对齐**：`wps.header_footers.footer_page_number` 一次 update 设 `alignment=right`（空页脚可直接插入）→ `wps.header_footers.footer_content` query 回读；勿先 `footer_alignment` query。

### 场景 3：删除页眉/页脚内容

适用：清空页眉文字，正文与其它版式保留。

**执行时由用户指定**：仅页眉或是否含页脚（默认页眉）。

1. `wps.header_footers.header_content` query — 基线。
2. `wps.header_footers.header_content` update — `content=""`（优先于 `wps.header_footers.header_remove` 若仅清文字）。
3. `wps.header_footers.header_content` query — 验证为空。

### 场景 4：页脚文字对齐（无页码变更）

适用：仅调整页脚文字或已有页码块的对齐，与场景 2 可合并执行。

**执行时由用户指定**：`wps.header_footers.footer_alignment` 或页码接口自带的 `alignment` 参数。

1. 若已插入页码：在场景 2 的 update 中带 `alignment` 即可。
2. 若独立页脚文案：`wps.header_footers.footer_content` / `wps.header_footers.footer_alignment` update → query 回读。
