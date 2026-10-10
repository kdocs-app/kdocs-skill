# wps.texts.line_spacing

#### 功能说明

设置在线文字文档文本的段落行距。

**幂等性**：是 — safe

> 节下正文批量行距：`search` 定位节标题与下一节边界 → `scope=ranges` 一次写入 → `format` 回读；`spacing_rule` 见 `enums.md`（0=单倍/1=1.5倍/2=双倍）。详见 `references/workflows/wps-section-body-format.md`（场景 3）。
> 选区固定磅值行距：`search` + `range` → `spacing_rule="4"` + `spacing_value=N`（磅）；详见 `references/workflows/wps-paragraph-align-indent.md`（场景 5）。

#### 支持功能

| 功能简述 | 详情 |
|--------------|------|
| 按段落设置行距 | [texts-line_spacing/paragraphs_update.md](texts-line_spacing/paragraphs_update.md) |
| 按字符区间设置行距 | [texts-line_spacing/ranges_update.md](texts-line_spacing/ranges_update.md) |
