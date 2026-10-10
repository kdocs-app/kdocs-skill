# 公文字体与书签

> 公文黑体仿宋、添加书签、删除书签

**适用场景**：按公文规范设置标题/正文字体，或按锚点文字添加书签、按名称删除书签。

- `wps.texts.font` ranges + `wps.bookmarks.*`
**触发词**：公文黑体仿宋、添加书签、删除书签

## 涉及工具

`wps.texts.search`、`wps.texts.count`、`wps.texts.structure`、`wps.texts.range`、`wps.texts.font`、`wps.bookmarks.insert`、`wps.bookmarks.delete`、`wps.bookmarks.exists`、`wps.bookmarks.content`。见 `references/wps/` 对应说明。

## 执行流程

### 场景 1：公文标题+正文字体

适用：用户要求按公文标准设置标题**三号黑体**、正文**三号仿宋**（如「按公文标准设置《…通知》的字体…」）。

1. `wps.texts.search` 一次 — 标题 prompt 字面（书名号内或首行标题）；得标题字符区间 `[0,end)`。
2. `wps.texts.count` + `wps.texts.structure` **各一次** — 确认标题段与段落数。
3. `wps.texts.range` 一次 — 文末 `end` 作正文字符区间上界。
4. `wps.texts.font` **两次** — `scope=ranges`：标题区间设 **黑体 16pt**；正文 `[title_end, doc_end)` 设 **仿宋 16pt**。
5. `wps.texts.font` 一次 — 标题/正文代表段回读。

> 三号=16pt。先读 `references/wps/texts.font.md`。不要 逐段扫全文 font。

### 场景 2：按锚点文字添加书签

适用：用户要求在指定文字处添加命名书签（如「对"3.1 核心功能列表"处添加书签叫"重点内容"」）。

1. `wps.texts.search` 一次 — 锚点 prompt 字面；须唯一命中，得 `begin/end`。
2. `wps.bookmarks.insert` 一次 — `scope=ranges`，`bookmark_name`=prompt 中的书签名。
3. `wps.bookmarks.content` 一次 — `verb=query` 回读书签覆盖文字。

> 先读 `references/wps/bookmarks.insert.md`、`references/wps/texts.search.md`。

### 场景 3：按名删除书签

适用：用户要求删除文档中指定名称的书签（prompt 常含先添加再删）。

1. （若.insert` — 按「按锚点文字添加书签」先创建。
2. `wps.bookmarks.delete` 一次 — `bookmark_name`=prompt 字面。
3. `wps.bookmarks.exists` 一次 — 确认 `exists=false`。

> 先读 `references/wps/bookmarks.delete.md`。
