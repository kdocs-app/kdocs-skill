# 目录插入与更新

> 插入目录、更新目录、生成目录

**适用场景**：在标题下插入自动目录、更新已有目录，或按全文标题结构生成目录。

- 主用 `wps.tocs.*`；文档需已有合适标题大纲
**触发词**：插入目录、更新目录、生成目录

## 涉及工具

`wps.texts.search`、`wps.texts.range`、`wps.tocs.data`、`wps.tocs.update`、`wps.tocs.update_all`、`wps.tocs.format`、`wps.tocs.count`。见 `references/wps/tocs.*`。

## 执行流程

### 场景 1：目录标题下插入自动目录（1–3 级）

适用：用户要求在「**目录**」标题**下面****自动生成**文档目录域，显示到 N 级标题（如「显示到3级标题」）。

1. `wps.texts.search` 一次 — 「目录」；取**首命中**正文标题（排除目录域条目重复命中）。
2. `wps.texts.range` 一次 — search begin → `paragraph_index`（P_dir；N>10 用二分，不要连续 range P±1 试探）。
3. `wps.tocs.count` query — count>0 时先 `wps.tocs.data` delete 再 insert。
4. `wps.tocs.data` insert — `scope=paragraphs`、`position=after`、`paragraph_index=P_dir`、`upper_level=1`、`lower_level=3`（按 prompt 调整）。
5. `wps.tocs.update` 一次 — 刷新目录域。
6. `wps.tocs.data` query + `wps.tocs.count` — 回读 level 与 count=1。

> 不要 `wps.lists.apply_with_level` / `wps.lists.multilevel`（手动编号场景）。先读 `references/wps/tocs.data.md`、`references/wps/tocs-data/paragraphs_insert.md`。

### 场景 2：更新文档目录

适用：用户要求**更新/刷新**文档目录（如「更新PRD文档的目录」）。

1. `wps.tocs.count` query 一次 — 判断已有目录域。
2. **count≥1**：`wps.tocs.update`（或 `wps.tocs.update_all`）一次 → `wps.tocs.data` query 回读。
3. **count=0**（仅手动目录行）：改走「目录标题下插入自动目录」workflow（search→range→insert→update→回读）。

> 不要对 count=0 仅调用 update（无效）。先读 `references/wps/tocs.update.md`、`references/wps/tocs.count.md`。

### 场景 3：全文生成目录

适用：用户要求**生成目录**（无锚点段落，如 prompt「生成目录」）。

1. `wps.tocs.count` query 一次 — count>0 则 `wps.tocs.data` delete 旧目录。
2. `wps.tocs.data` insert — `scope=documents`、`upper_level=1`、`lower_level=3`（按。
3. `wps.tocs.update` 一次 — 刷新。
4. `wps.tocs.data` query + `wps.tocs.count` — 回读。

> 。先读 `references/wps/tocs-data/documents_insert.md`。
