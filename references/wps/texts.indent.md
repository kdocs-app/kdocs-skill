# wps.texts.indent

#### 功能说明

设置在线文字文档文本的段落缩进。paragraph_style.indent 的 indent_type/unit 必须为字符串：indent_type 取 "1"=左缩进/"2"=右缩进/"3"=首行缩进，unit 取 "1"=磅/"2"=字符；传数字类型会报 400001，其它名称（first_line/char 等）静默回落默认值。

**幂等性**：是 — safe

> 选区首行缩进：`search` 定位 `<selected_text>` → `range` 映射段落 → `indent_type="3"` + `unit="2"`；详见 `references/workflows/wps-paragraph-align-indent.md`（场景 2）。
> 编号列表悬挂缩进：每段两次写 — 首行 `indent_type="3"` `value=-2` + 左 `indent_type="1"` `value=2`（`unit="2"` 字符）；`lists.blocks` 不可用时用 search 定位。详见 `references/workflows/wps-paragraph-align-indent.md`（场景 3）。
> 段落左缩进（厘米）：`search` 定位目标段 → 左缩进 `indent_type="1"`、`unit="1"`（磅）、`indent_value=厘米×28.35`（2cm≈56.7）；**不支持** `unit=cm`。详见 `references/workflows/wps-paragraph-align-indent.md`（场景 4）。

#### 支持功能

| 功能简述 | 详情 |
|--------------|------|
| 按段落设置缩进 | [texts-indent/paragraphs_update.md](texts-indent/paragraphs_update.md) |
| 按字符区间设置缩进 | [texts-indent/ranges_update.md](texts-indent/ranges_update.md) |
