# wps.texts.alignment

#### 功能说明

在线文字文档文本的段落对齐：支持 设置/查询。query 读回段落对齐（format_info.alignment，WdParagraphAlignment 数值口径：0=左、1=居中、2=右、3=两端、4=分散）。

**幂等性**：是 — safe

> 节标题下正文两端对齐：`search` 定位标题 → `range` 映射段落 → 标题后连续 outline_level=10 为正文段，`alignment=justify` 批量写入；详见 `references/workflows/wps-section-body-format.md`（场景 2）。勿 range/format 扫全文。

#### 支持功能

| 功能简述 | 详情 |
|--------------|------|
| 查询段落对齐（format_info.alignment） | [texts-alignment/paragraphs_query.md](texts-alignment/paragraphs_query.md) |
| 按段落设置对齐 | [texts-alignment/paragraphs_update.md](texts-alignment/paragraphs_update.md) |
| 按字符区间设置对齐 | [texts-alignment/ranges_update.md](texts-alignment/ranges_update.md) |
