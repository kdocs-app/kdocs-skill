# wps.texts.format

#### 功能说明

在线文字文档文本的段落格式：支持 查询/设置。查询返回当前段落格式（对齐、缩进、行距等实际值）；设置按段落或字符区间写入格式。

**幂等性**：是 — safe

> **标题段设色定位**：`wps.texts.structure` 不可用时，**只对 P1 调用一次** paragraphs query 查 outline_level；若 P1 为 1–9 即已找到标题段，立即转 `font_color` 设色。**禁止**为「确认无其他标题」而对 P2…PN 逐段 query（形似标题的正文段 outline_level 为 10，不是标题段）。详见 `wps.md` → 标题段自定义色设置。
> **样式清除验证**：清除/套用样式后须用 `wps.texts.style` 回读 `style_name`/`outline_level`；**禁止**无 `key` 的 paragraphs query（400100 TEXT_FORMAT key required）。
> **段前/段后间距**：`scope=paragraphs` update 时 `key=SpaceBefore` 或 `SpaceAfter`、`value=N`（磅，字符串）；回读用 paragraphs query 的 `space_before`/`space_after`。详见 `references/workflows/wps-paragraph-align-indent.md`（场景 6）。

#### 支持功能

| 功能简述 | 详情 |
|--------------|------|
| 按段落查询格式信息 | [texts-format/paragraphs_query.md](texts-format/paragraphs_query.md) |
| 按段落设置格式 | [texts-format/paragraphs_update.md](texts-format/paragraphs_update.md) |
| 按字符区间设置格式 | [texts-format/ranges_update.md](texts-format/ranges_update.md) |
