# 复制粘贴与格式刷

> 复制粘贴与格式刷

**适用场景**：文档内「复制」源段文字到锚点后粘贴（纯文本或保留格式），表格复制为纯文本，或格式刷单段/批量同级标题。无系统剪贴板 API。
**触发词**：纯文本粘贴、保留格式、格式刷、表格粘贴

## 涉及工具

`wps.texts.search`、`wps.texts.content`、`wps.texts.clear_format`、`wps.texts.font`、`wps.texts.format`、`wps.texts.format_copy`、`wps.texts.range`、`wps.texts.structure`、`wps.texts.count`、`wps.tables.count`、`wps.tables.dimensions`、`wps.tables.row_content`、`wps.tables.range`、`wps.tables.create`、`wps.tables.convert_to_text`。见 `references/wps/` 对应说明。

## 执行流程

### 场景 1：粘贴为纯文本

适用：粘贴后只保留文字（含表格复制为纯文本）。

**执行时由用户指定**：源字面、粘贴锚点字面；表格时 `table_index`。

**路径 A — 段落文字**：

1. `wps.texts.search` — 源 begin/end。
2. `wps.texts.search` — 锚点唯一命中。
3. `wps.texts.content` — `scope=ranges`、`verb=insert`，在锚点 end 插入源字面。
4. `wps.texts.clear_format` — 粘贴区间。

**路径 B — 表格**：

1. `wps.tables.count` / `wps.tables.dimensions` → 逐行 `wps.tables.row_content` 拼纯文本。
2. `wps.tables.range` — 表尾 end 为 insert 锚点。
3. `wps.texts.content` insert + `wps.texts.clear_format`。
4. `wps.texts.search` — 代表单元格字面验证双命中（表内+表后）。`wps.tables.create` / `wps.tables.convert_to_text` 报错则勿重试本路径。

### 场景 2：粘贴保留格式

适用：粘贴后保留源段字符格式。

**执行时由用户指定**：源与锚点字面。

1. `wps.texts.search` — 源与锚点。
2. `wps.texts.content` — ranges insert。
3. `wps.texts.font` — 对比粘贴区间与源区间；`wps.texts.content` 段落回读。勿 `paragraphs insert position=after`（行内语义不符）。

### 场景 3：格式刷

适用：把源段/标题格式刷到目标段，或刷到所有同级标题。

**执行时由用户指定**：源字面、目标字面或 `outline_level` 筛选批。

**路径 A — 单段到单段**：

1. `wps.texts.search` — 源、目标 → 各 `wps.texts.range` 得 paragraph_index。
2. `wps.texts.format_copy` — source_paragraph + target 段索引。
3. `wps.texts.format` — 回读目标与源一致。

**路径 B — 批量同级标题**：

1. `wps.texts.count` + `wps.texts.structure` — 分批得目标 outline 段列表。
2. `wps.texts.search` — 源 → `wps.texts.range` 源段 index。
3. `wps.texts.format_copy` — 对各目标（连续段可 target_start/end 一次；非连续各一次）。
4. `wps.texts.format` — 抽样回读。
