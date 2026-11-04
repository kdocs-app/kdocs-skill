# 多维表富文本与侧边栏智能文档

> 在多维表中创建/绑定 Note 富文本字段、写入正文，以及（可选）创建表内侧边栏智能文档并写块

**适用场景**：用户要在 .dbt 里用富文本列展开写长文（结论、行动项等），或在同一多维表内挂一份共用智能文档（规范、说明）

**触发词**：富文本字段、Note、侧边栏智能文档、FlexPaper、多维表写文档、单元格富文本、表内智能文档

- 多维表行内需要可点开编辑的富文本正文
- 多维表内需要一份不绑单行的共用智能文档（侧边栏智能文档）

**工具链**：建/定位 dbt → Note 列 →（逐行串行）create_innerdoc_flexpaper → 绑 fileId → 写块 → sync summary → 延迟回读；（可选）再建 1 份侧边栏 FlexPaper 写块

## 涉及工具

| 工具 | 服务 | 用途 |
|------|------|------|
| `dbsheet.create_innerdoc_flexpaper` | dbsheet | 为行内 Note 或侧边栏新建 FlexPaper，取 content_id（会多出侧栏 sheet，属拿 fileId 的权宜） |
| `dbsheet.update_records` | dbsheet | 行内 Note 将 content_id 写入 fileId 并带 summary |
| `dbsheet.innerdoc_block_create` | dbsheet | 用 attachment_id=content_id/fileId 写入文档块 |

## 执行流程

两条链共用 `innerdoc_block_*`，**attachment_id 来源不同**。禁止空 `fileId`、禁止把纯字符串当 Note 值（易 `400100`）。完整参数见 `SKILL.md` 工具表，填参执行 `kdocs-cli help dbsheet.<tool>`（含 `innerdoc_block_*`、`create_records`、`update_records` 等）。

### 硬约束（一次性写成功）

- **串行**：多行 Note 必须「一行做完并验过」再下一行；禁止并行/连发多格 flexpaper+写块。
- **验收**：`code=0` 不够。写块 + sync summary 后**至少等待 3 秒**，再 `innerdoc_block_list`（解码 results）确认正文含预期关键词；失败则**同一格**重跑「写块→sync→再等→再 list」最多 1 次。最终以延迟回读为准。
- **勿中途打开侧栏**：`create_innerdoc_flexpaper` 会在表内多出一个 FlexPaper sheet（左侧列表可见）。写完全部 Note / 侧边栏并延迟验过之前，**不要**在网页端点开这些 sheet（易把未落稳正文冲成空模板「待添加富文本内容」）。交付后用户再从**表格单元格**点开 Note。
- **侧栏规范只建 1 份**：共用说明走路径 B，勿为每行再建一份侧栏文档。

### A. 行内 Note 富文本（绑到某一格）

1. 建/打开 `.dbt`，`dbsheet.get_schema`（或 `get_schema_detail`）确认 `sheet_id` 与 Note 列名。无 Note 列 → `dbsheet.create_fields`（`type: Note`）。
2. 先写入非 Note 字段：`create_file_with_content` / `create_records` / `update_records`（可暂不填 Note）。
3. **拿到合法 fileId**：对该单元格调用 `dbsheet.create_innerdoc_flexpaper`（`name` 可用议题号等），返回的 **`content_id` 即该格 Note 的 `fileId`**（也是后续 `attachment_id`）。此举会挂出侧栏 sheet，属当前拿合法 `fileId` 的权宜路径。
4. `dbsheet.update_records`：该 Note 字段设为 `{"fileId":"<content_id>","summary":"<短摘要>","modifyDate":"yyyy/mm/dd hh:mm:ss"}`。禁止省略 `fileId` 或只传 summary。
5. `dbsheet.innerdoc_block_create`（或 convert→create）：`attachment_id` = 上一步 `fileId`，写入标题/段落等块。
6. 再 `update_records` **同步 `summary`** → **等待 ≥3s** → `innerdoc_block_list` 回读；失败则同格重写一轮。禁止仅凭 `code=0` 或「刚写完立刻 list 一次」判成功。

### B. 侧边栏智能文档（表内共用 FlexPaper，不绑单行）

1. 同一 `file_id` 上 **仅一次** `dbsheet.create_innerdoc_flexpaper` → 得 `content_id` / `sheet_id`。
2. `innerdoc_block_*`：`attachment_id` = 该 `content_id`（也可经 `get_schema_detail` 回读）。**不要**把此 `content_id` 写入某行 Note。
3. 侧边栏**无需** sync summary。写后同样 **等待 ≥3s** 再 `innerdoc_block_list`。

### 消歧

| 目标 | fileId / attachment_id | 是否 update_records 绑格 | sync summary |
|------|------------------------|--------------------------|--------------|
| 行内 Note | FlexPaper 的 `content_id` | 要 | 要 |
| 侧边栏智能文档 | 同上，但只作 attachment_id | 否 | 否 |

多行 Note：每格单独一轮「create_innerdoc_flexpaper → 绑 fileId → 写块 → sync summary → **延迟回读**」，勿复用其它行的 `fileId`，勿并行。
