# 页眉页脚与页码

> 页眉页脚与页码

**适用场景**：写入或删除页眉，在页脚插入/调整页码（阿拉伯、「第 X 页」、罗马数字）及页眉页脚对齐。

- 正文段落对齐 → `references/workflows/wps-paragraph-align-indent.md`

**触发词**：页眉、页脚、页码、罗马数字、删除页眉

## 涉及工具

`wps.header_footers.header_content`、`wps.header_footers.header_alignment`、`wps.header_footers.header_remove`、`wps.header_footers.footer_page_number`、`wps.header_footers.footer_page_number_custom`、`wps.header_footers.footer_content`、`wps.header_footers.footer_alignment`、`wps.header_footers.page_number_style`。
对上表各工具执行 `kdocs-cli help <tool>` 获取参数与约束。
## 执行流程

### 场景 1：页眉内容与对齐

适用：页眉写入指定文字并设置对齐（居中/左/右）。

**执行时由用户指定**：页眉文案、`alignment`（如 center=1）。

1. `wps.header_footers.header_content` update — 文案。
2. `wps.header_footers.header_alignment` update — 对齐（勿 alignment query，常 400100）。
3. `wps.header_footers.header_content` query — 回读文字。
> 执行本场景所列步骤前，先运行 `kdocs-cli help wps.header_footers.header_alignment wps.header_footers.header_content` 获取参数与约束。
### 场景 3：删除页眉/页脚内容

适用：清空页眉文字，正文与其它版式保留。

**执行时由用户指定**：仅页眉或是否含页脚（默认页眉）。

1. `wps.header_footers.header_content` query — 基线。
2. `wps.header_footers.header_content` update — `content=""`（优先于 `wps.header_footers.header_remove` 若仅清文字）。
3. `wps.header_footers.header_content` query — 验证为空。
> 执行本场景所列步骤前，先运行 `kdocs-cli help wps.header_footers.header_content wps.header_footers.header_remove` 获取参数与约束。
