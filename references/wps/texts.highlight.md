# wps.texts.highlight

#### 功能说明

设置在线文字文档文本的段落高亮。

**幂等性**：是 — safe

> 多段落重点内容批量高亮：对每条条目 `search` 定位后，用 `scope=paragraphs` + `paragraph_index` 直接设色（粉色 `highlight_color=pink` → 5）；**禁止**先 `content` 或逐段 `range` 扫全文。详见 `references/workflows/wps-multi-target-search-style.md`（场景 2 路径 B）。
> **全文清除高亮**：`highlight_color=0`（WdNoHighlight）；逐段 `font` 探测后对仍有高亮的区间写 0。流程见 `references/workflows/wps-multi-target-search-style.md`（场景 8）。

#### 支持功能

| 功能简述 | 详情 |
|--------------|------|
| 按段落设置高亮 | [texts-highlight/paragraphs_update.md](texts-highlight/paragraphs_update.md) |
| 按字符区间设置高亮 | [texts-highlight/ranges_update.md](texts-highlight/ranges_update.md) |
