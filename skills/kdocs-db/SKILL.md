# 多维表格（dbt）（工具路由索引）

本文件包含金山文档 Skill 多维表格的操作说明。

**适用范围**：支持数据表、视图、字段和记录的增删改查、记录筛选与分页查询、表单视图配置、父子记录关联、分享与高级权限管理、仪表盘及其组件管理、Webhook 订阅，以及智能文档和富文本字段内的块操作。

**不适用范围**：独立智能文档、独立表单、Excel、云盘文件管理、通用创建上传。遇到上述域外需求、需要借助其他组件定位文件或准备外部资源，或无法判断请求应由哪个组件处理时，[返回主 Skill](../../SKILL.md)安排，不自行切换子 Skill。

以下工具不可逆，调用前必须向用户确认；参数与边界以 `kdocs-cli help <tool>` 的工具卡 为准：

`dbsheet.delete_sheet`、`dbsheet.delete_view`、`dbsheet.delete_fields`、`dbsheet.delete_records`、`dbsheet.sheet_batch_delete`、`dbsheet.dashboard_delete`、`dbsheet.dashboard_clear`、`dbsheet.dashboard_web_extension_delete`、`dbsheet.permission_delete_roles_async`、`dbsheet.innerdoc_block_delete`、`dbsheet.innerdoc_block_batch_delete`

> **工具填参**：下表每行 `<tool>` 选定后须 `kdocs-cli help <tool>`（可批量：`kdocs-cli help dbsheet.get_schema dbsheet.create_sheet`）。

## 一、数据表管理

> 数据表的 Schema 查询、增删改与批量操作

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `dbsheet.get_schema` | 获取文档结构（表/字段/视图） | `url`, `link_id`, `file_id` |
| `dbsheet.create_sheet` | 创建数据表 | `url`, `link_id`, `file_id`, `name`, `fields` |
| `dbsheet.update_sheet` | 修改数据表名称 | `url`, `link_id`, `file_id`, `sheet_id` |
| `dbsheet.delete_sheet` | 删除数据表 | `url`, `link_id`, `file_id`, `sheet_id` |
| `dbsheet.sheet_batch_create` | 批量创建工作表 | `url`, `link_id`, `file_id`, `body` |
| `dbsheet.sheet_batch_delete` | 批量删除工作表 | `url`, `link_id`, `file_id`, `body` |
| `dbsheet.get_schema_detail` | 获取完整 Schema（含侧边栏智能文档和富文本字段的 content_id） | `url`, `link_id`, `file_id` |

## 二、视图管理

> 视图的增删改查与列表

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `dbsheet.create_view` | 创建视图 | `url`, `link_id`, `file_id`, `sheet_id`, `name`, `type` |
| `dbsheet.update_view` | 更新视图配置 | `url`, `link_id`, `file_id`, `sheet_id`, `view_id` |
| `dbsheet.delete_view` | 删除视图 | `url`, `link_id`, `file_id`, `sheet_id`, `view_id` |
| `dbsheet.views_list` | 列出视图 | `url`, `link_id`, `file_id`, `sheet_id` |
| `dbsheet.views_get` | 获取单个视图 | `url`, `link_id`, `file_id`, `sheet_id`, `view_id` |

## 三、字段管理

> 字段的增删改

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `dbsheet.create_fields` | 批量创建字段 | `url`, `link_id`, `file_id`, `sheet_id`, `fields` |
| `dbsheet.update_fields` | 批量更新字段 | `url`, `link_id`, `file_id`, `sheet_id`, `fields` |
| `dbsheet.delete_fields` | 批量删除字段 | `url`, `link_id`, `file_id`, `sheet_id`, `fields` |

## 四、记录操作

> 记录的增删改查

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `dbsheet.create_records` | 批量创建记录 | `url`, `link_id`, `file_id`, `sheet_id`, `records` |
| `dbsheet.update_records` | 批量更新记录 | `url`, `link_id`, `file_id`, `sheet_id`, `records` |
| `dbsheet.list_records` | 分页遍历记录（支持筛选） | `url`, `link_id`, `file_id`, `sheet_id` |
| `dbsheet.get_record` | 获取单条记录 | `url`, `link_id`, `file_id`, `sheet_id`, `record_id` |
| `dbsheet.delete_records` | 批量删除记录 | `url`, `link_id`, `file_id`, `sheet_id`, `records` |
| `dbsheet.records_list` | 列举记录 | `url`, `link_id`, `file_id`, `sheet_id`, `fields` |
| `dbsheet.records_search` | 检索多条记录 | `url`, `link_id`, `file_id`, `sheet_id`, `records` |

## 五、表单视图

> 表单视图的元数据与字段管理

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `dbsheet.form_list_fields` | 列出表单问题 | `url`, `link_id`, `file_id`, `sheet_id`, `view_id` |
| `dbsheet.form_update_field` | 更新表单问题 | `url`, `link_id`, `file_id`, `sheet_id`, `view_id`, `field_id`, `body` |
| `dbsheet.form_get_meta` | 获取表单元数据 | `url`, `link_id`, `file_id`, `sheet_id`, `view_id` |
| `dbsheet.form_update_meta` | 更新表单元数据 | `url`, `link_id`, `file_id`, `sheet_id`, `view_id`, `body` |

## 六、父子记录

> 层级关系的绑定、解绑、状态与列表

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `dbsheet.parent_disable` | 禁用父子关系（仅前端） | `url`, `link_id`, `file_id`, `sheet_id` |
| `dbsheet.parent_enable` | 启用父子关系（仅前端） | `url`, `link_id`, `file_id`, `sheet_id` |
| `dbsheet.parent_status` | 查询父子关系是否禁用 | `url`, `link_id`, `file_id`, `sheet_id` |
| `dbsheet.parent_bind_children` | 绑定父子记录 | `url`, `link_id`, `file_id`, `sheet_id`, `parent_id`, `body` |
| `dbsheet.parent_list_children` | 查询子记录列表 | `url`, `link_id`, `file_id`, `sheet_id`, `parent_id` |
| `dbsheet.parent_unbind_children` | 解绑父子记录 | `url`, `link_id`, `file_id`, `sheet_id`, `parent_id`, `body` |

## 七、分享视图

> 视图分享的开启、关闭、权限、状态

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `dbsheet.share_open_view` | 打开分享视图 | `url`, `link_id`, `file_id`, `sheet_id`, `view_id`, `body` |
| `dbsheet.share_view_status` | 查询视图是否已开启分享 | `url`, `link_id`, `file_id`, `sheet_id`, `view_id` |
| `dbsheet.share_get_link_info` | 查询分享链接信息 | `url`, `link_id`, `file_id`, `sheet_id`, `view_id`, `share_id` |
| `dbsheet.share_close_view` | 关闭分享视图 | `url`, `link_id`, `file_id`, `sheet_id`, `view_id`, `share_id` |
| `dbsheet.share_get_repeatable` | 查询表单是否可重复提交 | `url`, `link_id`, `file_id`, `sheet_id`, `view_id`, `share_id` |
| `dbsheet.share_set_repeatable` | 设置表单是否可重复提交 | `url`, `link_id`, `file_id`, `sheet_id`, `view_id`, `share_id`, `body` |
| `dbsheet.share_update_permission` | 修改分享权限 | `url`, `link_id`, `file_id`, `sheet_id`, `view_id`, `share_id`, `body` |

## 八、高级权限

> 角色与主体的权限管理与异步任务

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `dbsheet.permission_list_roles` | 列举自定义角色 | `url`, `link_id`, `file_id` |
| `dbsheet.permission_query_task` | 获取异步任务结果 | `url`, `link_id`, `file_id`, `task_id` |
| `dbsheet.permission_create_roles_async` | 新增自定义角色（异步） | `url`, `link_id`, `file_id`, `body` |
| `dbsheet.permission_update_roles_async` | 更新自定义角色（异步） | `url`, `link_id`, `file_id`, `body` |
| `dbsheet.permission_delete_roles_async` | 删除自定义角色（异步） | `url`, `link_id`, `file_id`, `body` |
| `dbsheet.permission_list_subjects` | 列举成员（内容权限） | `url`, `link_id`, `file_id`, `cloud_permission_id`, `permission_type` |

## 九、仪表盘

> 仪表盘与图表组件的创建、管理、统计查询：新建/更新/删除/清空看板、组件增删改查、图表列表与统计数据回读

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `dbsheet.dashboard_copy` | 复制仪表盘 | `url`, `link_id`, `file_id`, `dashboard_id`, `body` |
| `dbsheet.dashboard_list` | 列出仪表盘 | `url`, `link_id`, `file_id` |
| `dbsheet.dashboard_create` | 创建仪表盘 | `url`, `link_id`, `file_id`, `name` |
| `dbsheet.dashboard_update` | 更新仪表盘 | `url`, `link_id`, `file_id`, `dashboard_id` |
| `dbsheet.dashboard_delete` | 删除仪表盘 | `url`, `link_id`, `file_id`, `dashboard_id` |
| `dbsheet.dashboard_clear` | 清空仪表盘图表 | `url`, `link_id`, `file_id`, `dashboard_id` |
| `dbsheet.dashboard_component_list` | 列出仪表盘部件 | `url`, `link_id`, `file_id`, `dashboard_id` |
| `dbsheet.dashboard_web_extension_list` | 查询仪表盘部件 | `url`, `link_id`, `file_id`, `dashboard_id` |
| `dbsheet.dashboard_component_create` | 创建仪表盘组件 | `url`, `link_id`, `file_id`, `dashboard_id` |
| `dbsheet.dashboard_web_extension_create` | 创建仪表盘部件 | `url`, `link_id`, `file_id`, `dashboard_id`, `web_extension_type` |
| `dbsheet.dashboard_component_update` | 更新仪表盘组件 | `url`, `link_id`, `file_id`, `dashboard_id`, `web_extension_key` |
| `dbsheet.dashboard_web_extension_update` | 更新仪表盘部件 | `url`, `link_id`, `file_id`, `dashboard_id`, `web_extension_key` |
| `dbsheet.dashboard_component_delete` | 删除仪表盘组件 | `url`, `link_id`, `file_id`, `dashboard_id`, `web_extension_key` |
| `dbsheet.dashboard_web_extension_delete` | 删除仪表盘部件 | `url`, `link_id`, `file_id`, `dashboard_id`, `web_extension_key` |
| `dbsheet.dashboard_chart_list` | 列出仪表盘图表 | `url`, `link_id`, `file_id`, `dashboard_id` |
| `dbsheet.dashboard_chart_data` | 查询图表统计数据 | `url`, `link_id`, `file_id`, `dashboard_id` |

## 十、Webhook 与开放协作

> Webhook 的创建、列表、删除

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `dbsheet.list_webhooks` | 查询全部 Hook 订阅 | `url`, `link_id`, `file_id` |
| `dbsheet.create_webhook` | 创建 Hook 订阅 | `url`, `link_id`, `file_id`, `body` |
| `dbsheet.delete_webhook` | 取消 Hook 订阅 | `url`, `link_id`, `file_id`, `hook_id` |

## 十一、智能文档/富文本字段块操作

> 多维表格智能文档（FlexPaper sheet）与富文本字段内的文档块增删改查与格式转换

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `dbsheet.create_innerdoc_flexpaper` | 在多维表格文件中新建一个智能文档（FlexPaper sheet） | `url`, `link_id`, `file_id`, `name` |
| `dbsheet.innerdoc_block_create` | 创建侧边栏智能文档/富文本字段内的文档块 | `url`, `link_id`, `file_id`, `attachment_id`, `arg` |
| `dbsheet.innerdoc_block_query` | 查询侧边栏智能文档/富文本字段内的单个文档块 | `url`, `link_id`, `file_id`, `attachment_id`, `arg` |
| `dbsheet.innerdoc_block_list` | 批量查询侧边栏智能文档/富文本字段内的文档块列表 | `url`, `link_id`, `file_id`, `attachment_id`, `arg` |
| `dbsheet.innerdoc_block_update` | 更新侧边栏智能文档/富文本字段内的单个文档块 | `url`, `link_id`, `file_id`, `attachment_id`, `arg` |
| `dbsheet.innerdoc_block_batch_update` | 批量更新侧边栏智能文档/富文本字段内的多个文档块 | `url`, `link_id`, `file_id`, `attachment_id`, `arg` |
| `dbsheet.innerdoc_block_delete` | 删除侧边栏智能文档/富文本字段内的单个文档块 | `url`, `link_id`, `file_id`, `attachment_id`, `arg` |
| `dbsheet.innerdoc_block_batch_delete` | 批量删除侧边栏智能文档/富文本字段内的多个文档块 | `url`, `link_id`, `file_id`, `attachment_id`, `arg` |
| `dbsheet.innerdoc_block_convert` | 将 HTML/Markdown 转换为侧边栏智能文档/富文本字段内的文档块结构 | `url`, `link_id`, `file_id`, `attachment_id`, `arg` |

## 入参形态

经 **kdocs-cli** 调用时，下游 API 仍是 **JSON**（`records[]`、`fields[]`、`filter` 等）。标量用 `--file-id` / `--sheet-id`（见主 Skill「调用格式」）；批量表与筛选用下面的领域选项。

### kdocs-cli

| 场景 | 推荐 | 禁止 |
|------|------|------|
| 建/改字段 | 先写字段 MD 表，再 `--fields-file` / `create_sheet` 的 `--fields`；或 JSON `fields[]` | 在 `--file` JSON 里直接贴 MD 表字符串 |
| 建/改记录 | 先写记录 MD 表，再 `--records-file`（`;` 多选由 CLI 解析）；或 JSON `records[]` | 在 JSON 字段里直接贴整张 MD 表 |
| 条件列举记录 | 命令行**重复** `--criterion`，可选 `--filter-mode AND` 或 `OR` | 把 criterion 文本塞进 JSON；勿把筛选写成 MD 文件 |

```shell
kdocs-cli dbsheet create-records --file-id FID --sheet-id 1 --records-file ./rows.md
kdocs-cli dbsheet list-records --file-id FID --sheet-id 2 \
  --criterion "状态:Equals:进行中" \
  --filter-mode AND
```

MD 表：首行为表头（列名即字段名）；数据行 → `records[].fields` 或字段定义。多选等用分号：`紧急;重要`。`update_records` 表须含 `id` 列；空单元格在更新语义下表示清空该字段（见 `SKILL.md` 工具表，参数用 `kdocs-cli help dbsheet.update_records` 等）。复杂字段若 MD 解析不符预期，改用 JSON `records[]` / `fields[]`，或先 `dbsheet.get_schema` 再构造 JSON。

空单元格要清空：在 `update_records` 的 `fields` 里对该字段显式传空值（按字段类型，见对应 YAML），勿省略键指望“擦掉”。

## 工具组合速查

| 用户需求 | 推荐工具组合 |
|----------|-------------|
| 已有 .dbt 内新增数据表 | `dbsheet.create_sheet`（`fields` 必填；`views` 可省略，缺省补默认 Grid 视图） |
| 读多维表正文 | `read_file`（返回 content_format 为 dbsheet_records，含 schema、records） |
| 多维表格读结构/数据 | `dbsheet.get_schema` → `dbsheet.list_records` / `dbsheet.get_record` |
| 多维表格增删改记录 | 见「写入前字段映射」+「新增记录与全空记录复用」→ `dbsheet.update_records` / `dbsheet.create_records`；用户明确要求删除时 `dbsheet.delete_records` |
| 本地图片/文件写入附件字段 | `upload_attachment`（`file_id` 为目标 `.dbt`）→ `dbsheet.update_records`（`uploadId`=返回的 `object_id`，`source`=`upload_ks3`，`type` 见下节；勿用 `Cloud`） |
| 行内 Note 富文本 / 侧边栏智能文档 | 必读 `references/workflows/note-and-sidebar.md`（串行、延迟回读、勿中途点侧栏；侧边栏只建 1 份、不绑行） |
| 新建看板（仪表盘） | `dbsheet.dashboard_create` → 返回 `detail.sheet.id` 即 dashboard_id |
| 复制已有看板 | `dbsheet.dashboard_list` → `dbsheet.dashboard_copy` |
| 加图表组件 | `dbsheet.get_schema`（数据源 sheet_id/field_id）→ `dbsheet.dashboard_component_create`（`statistics_module_setting` + `chart_info`）逐个创建，layout 自行排布 |
| 改组件配置 | `dbsheet.dashboard_component_list`（拿 webExtensionKey）→ `dbsheet.dashboard_component_update` |
| 删组件 / 清空看板 | `dbsheet.dashboard_component_delete`（单个）/ `dbsheet.dashboard_clear`（全部，须确认） |
| 回读验证 | `dbsheet.dashboard_chart_list` → `dbsheet.dashboard_chart_data`（核对统计数值） |
| 改看板名称/图标 | `dbsheet.dashboard_update` |

### 本地文件写入附件字段（`type` 取值）

`upload_attachment` 只负责拿到 `object_id`；写入记录时附件字段的 `type` 必须按**小写扩展名**填，否则多维表前端点击附件会提示「此文件类型不支持打开，可下载后查看」（写入本身会成功，不报错）：

- **文档**：传小写扩展名，如 `doc`/`docx`/`xls`/`xlsx`/`ppt`/`pptx`/`pdf`/`txt`/`otl`/`dbt`；
- **图片**：传 `image/xxx`（如 `image/png`），可直接复用 `upload_attachment` 返回的 `new_content_type`；
- **禁止**传 `application/...` 等 MIME 类型；
- **必须小写**：`type` 区分大小写，前端按原值匹配白名单，`DOC`/`Doc` 同样无法打开；
- `.zip`/`.md`/`.json` 等不在预览白名单内的扩展名本就不支持在线打开（提示同上），属预期行为，不要为此改写 `type`；
- **存量数据修复**：此前误写 MIME 的附件**无需重新上传**，用 `dbsheet.update_records` 把该附件的 `type` 改为对应小写扩展名（`uploadId` 等其余字段保持不变）即可恢复打开。

`new_content_type` **不代表**附件字段该填的值（`.doc` → `application/octet-stream`、`.docx` → `application/zip`），**只有图片能直接作为 `type`**；文档类必须按扩展名写。
附件字段完整取值格式见 `kdocs-cli help dbsheet.create_fields` 的 Attachment 节。

```json
{ "uploadId": "<object_id>", "fileName": "报告.doc", "size": 9216, "source": "upload_ks3", "type": "doc" }
```

### 单选/多选建列（create_fields）

向 `.dbt` 预置 `SingleSelect`/`MultipleSelect` 选项时：

1. **扁平 `items`**：写在 `fields[].items`，**禁止** `fields[].data.items`（可 200 但 `items: null`）。
2. 创建后 **`dbsheet.get_schema`** 确认该列 `items` 非空，再 `create_records`/`update_records`。
3. 录入选项 **value** 字符串须与 `items[].value` 一致（或开启 `autoAddItem`）。

详见 `kdocs-cli help dbsheet.create_fields`。

有限枚举（状态、类别、班组、产线等）**必须** `SingleSelect`/`MultipleSelect` + `items`。禁止只灌 CSV/`records`、不声明 `type`：平台会建成 `MultiLineText`，界面无选项、查找引用也容易算不出值。

### 关联与查找引用

跨表关联、查找引用、统计引用时：

2. **Lookup 必须 `prefer_id: true`**，`linkField`/`lookupField` 用 `get_schema` 的字段 id。用字段名 + `prefer_id: false` 会报 `400100` Internal error。先建关联列并 `get_schema` 拿到 id，再创建 Lookup。
3. 查找引用：`type: "Lookup"` + `lookupType: 3` + `aggregation: "Origin"`。统计（如领用数量合计）：`lookupType: 2` + `aggregation: "Sum"`（不要对统计字段用 `Origin`）。被引用的枚举列建议也是 `SingleSelect`。
4. **`isAuto` 不可当作已生效**：`create_fields`/`update_fields` 即使返回 ok，`get_schema` 仍常为 `isAuto: false`、`filter: null`。此时用 `update_records` 写入关联记录 id 数组（`["id1","id2"]`）。禁止仅凭请求里写了 `isAuto: true` 宣称自动关联成功。
5. **Lookup 是否建成功以 `get_schema` 为准**：须 `type: Lookup`、`lookupType: 3`、`linkField`/`lookupField` 为非空字段 id。`list_records`/`get_record` 常不返回 Lookup 计算值（键缺失 ≠ 未配置成功）。交付前确认：关联列已有记录 id；schema 查找列配置正确。统计列（`lookupType: 2`）若回读全 `0` 再查关联与 `aggregation`。

### 写入前字段映射

向已有表增/改记录时：

1. **必须先** `dbsheet.get_schema` 确认 `sheet_id` 与字段名；禁止用 `read_file` 探 `sheet_id` 或字段结构。
2. 用户口述的字段名对照 schema **现有列**：**列名完全一致则直接用**。无同名列时 **禁止 Agent 自行语义映射**，须 `askUserQuestion` 让用户选择：写入哪个现有列 / 新建该列 / 跳过；`fields` 的 key 必须用用户确认后的表中真实列名。禁止新建与已有列同义的列。
3. 无等价列且确需新列 → **须 `askUserQuestion` 确认**（新建列 / 跳过该字段 / 写入其他列）后再 `dbsheet.create_fields`；禁止先 `create_records` 写入不存在的列碰运气，`Field not found` 失败后再建列**不算**已告知用户。
4. 映射完成后按「新增记录与全空记录复用」选择 `update_records` / `create_records`；**必须**用 `list_records` 或 `get_record` 回读验证，禁止仅凭 `result: ok` 宣称成功。
5. 回复用户时说明口述字段与表中真实列名的对应关系（如「任务名→文本」「状态→单选项」）；若新建了列须一并说明。

### 新增记录与全空记录复用

向已有表**新增** N 条记录时（用户未明确要求追加到表尾）：

1. `list_records` 按默认顺序盘点记录。
2. **全空记录**：`fields` 为 `{}`，或除 `AutoNumber`/`CreatedTime`/`CreatedBy`/`LastModifiedBy`/`LastModifiedTime`/`Formula`/`Lookup` 外所有字段均为空；任一业务字段有值则**跳过**。
3. 从列表头部依次取前 min(N, 全空记录数) 条 → `update_records`（每条须带 `id`）。
4. 仍有剩余 (N−K) 条 → `create_records` 追加（API 只能在表尾创建，无「插入到第几行」参数）。
5. **禁止**用 `delete_records`（含 `mode=all`）清理全空记录；**禁止**覆盖含用户数据的记录。
6. 不要求连续全空记录；全空记录不足时剩余追加到表尾，须在回复中说明。

用户明确说「在末尾/最后追加」时，可跳过步骤 2–3，直接 `create_records`。

---

## 仪表盘（看板）

用户说「做个看板 / 经营大屏 / 数据大屏」时的完整交付链路：**建看板 → 加组件 → 回读验证 → 推送结果**。

### 推荐工具组合

| 用户需求 | 推荐工具组合 |
|----------|-------------|
| 新建看板 | `dbsheet.dashboard_create`（`name`）→ 返回 `detail.sheet.id` 即 dashboard_id |
| 复制已有看板 | `dbsheet.dashboard_list` → `dbsheet.dashboard_copy` |
| 加图表组件 | `dbsheet.get_schema` → `dbsheet.dashboard_component_create` 逐个创建 |
| 改组件配置 | `dbsheet.dashboard_component_list` → `dbsheet.dashboard_component_update` |
| 删组件 / 清空看板 | `dbsheet.dashboard_component_delete` / `dbsheet.dashboard_clear`（须确认） |
| 回读验证 | `dbsheet.dashboard_chart_list` → `dbsheet.dashboard_chart_data` |
| 改看板名称/图标 | `dbsheet.dashboard_update` |

### 交付流程（销售看板示例）

1. `dbsheet.get_schema` → 数据源 `sheet_id=1`，字段：日期 `E`、销售额 `B`、品类 `A`。
2. `dbsheet.dashboard_create` `name="销售看板"` → `dashboard_id=6`。
3. `dbsheet.dashboard_component_create` 逐个创建（12 列网格布局，典型 `w=4,h=3`，一行 3 个）：
   - 指标卡：`statistics_module_setting={dataSourceSheetId:1, onlyCountDimension:true}` + `chart_info={type:"Value", name:"总销售额", layout:{x:0,y:0,w:4,h:3}}`；
   - 柱状图：`dimensionFieldId:"A"` + `statisticSettingAddList:[{fieldId:"B",statisticOptions:"sum"}]` + `chart_info={type:"BasicBar",…}`；
   - 折线图 / 饼图同理（`pieType:"ring"` 为环形）。
4. `dbsheet.dashboard_chart_list` `dashboard_id=6` → 确认组件全部 `valid=true`。
5. `dbsheet.dashboard_chart_data` `dashboard_id=6, chart_ids=[…]` → 核对统计数值。
6. 回复用户：看板链接（`get_file_link`）+ 组件清单 + 关键数值。

### 铁律

1. **两类 ID 不可混用**：`dashboard_id` 是仪表盘自身的 sheet id（`sheetType=xlDbDashBoardSheet`）；组件统计配置里的 `dataSourceSheetId` 才是 `get_schema` 返回的数据表 ID。
2. **fieldId 是字段 ID 不是字段名**：`dimensionFieldId`、`statisticSettingAddList[].fieldId` 等必须来自 `get_schema`（如 "B"、"E"），禁止凭字段名捏造。
3. **创建后必须回读**：`dashboard_chart_data` 核对数值；`result.type` 非 `Normal`（Empty/Invalid/DataIsProtected）时如实告知（Empty=数据源无记录），禁止编造数值，禁止仅凭 `result: ok` 宣称成功。
4. **不可恢复操作先确认**：`dashboard_delete`（删整个看板）、`dashboard_clear`（清空全部图表）执行前必须向用户确认。
5. **组件定位参数**：组件增删改用 `dashboard_component_list` 的 `webExtensionKey`；取数用 `dashboard_chart_list` 的 `id`，两者不混用。

### 图表类型与统计选项

- 图表类型（`chart_info.type`）：`Value` 指标卡 · `BasicBar` 柱状 · `BasicLine` 折线 · `HorizBar` 条形 · `BasicPie` 饼图/环形 · `Area` 面积 · `BarLine` 柱线组合 · `Funnel` 漏斗 · `Progress` 进度 · `CountDown` 倒计时 · `MatrixBubble` 矩阵气泡。
- 统计选项（`statisticOptions`）：`count` 计数 · `sum` 求和 · `avg` 平均 · `max`/`min` 最值 · `uniqueCount` 唯一计数 · `filledCount`/`unfilledCount` 填充/未填充计数 · `checkedCount`/`uncheckedCount` 选中/未选中计数 及对应百分比等。
- ⚠ 雷达图、词云、桑基图、关系网络图、嵌入表视图、富文本属插件类组件，暂不支持，用户需要时如实说明。

## 获取记录工具使用指南

| 场景 | 优先工具 | 备用工具 | 说明 |
|------|----------|----------|------|
| 列举数据表所有 / 分页记录 | `dbsheet.records_list` | `dbsheet.list_records` | `records_list` 基于游标分页；若返回错误，改用 `list_records`（页码分页） |
| 查询数据表中某一条记录 | `dbsheet.get_record` | `dbsheet.records_search` | `get_record` 直接按记录 id GET 查询；返回错误时可改用 `records_search` |
| 批量获取指定多条记录 | `dbsheet.records_search` | — | 传入记录 id 列表一次取回多条，无需逐条查询 |

---

## 高频操作流程

| 流程 | 说明 | 详细参考 |
|------|------|----------|
| 读取多维表云文档元信息 | 从多维表记录中提取云文档字段的 file_id，再调用 get_file_info 获取元信息（文件名、大小、类型、修改时间等） | `references/workflows/read-cloud-doc-meta.md` |
| 多维表富文本与侧边栏智能文档 | 在多维表中创建/绑定 Note 富文本字段、写入正文，以及（可选）创建表内侧边栏智能文档并写块 | `references/workflows/note-and-sidebar.md` |

## 错误速查表

| 错误特征 | 原因 | 处理方式 |
|----------|------|----------|
| `400100` Unknown enum / 参数值不被识别 | 凭直觉猜测 type 等枚举值，未查文档 | 执行 `kdocs-cli help <tool>`，从 description 中的对照表取正确枚举值，修正后重试一次 |
| `400001` 参数缺失 / 格式错误 | 未确认必填参数或参数类型 | 执行 `kdocs-cli help <tool>`，确认参数名、类型、必填性，补齐后重试 |
| 记录不全 / 需全量或分页 | `read_file` 单次返回 records 有上限 | 概览用 `read_file`；全量/分页/条件筛选用 `dbsheet.records_list` / `list_records` / `records_search` |
| `Field not found` / 字段不存在 | `create_records` 传了 schema 中不存在的列名 | 先 `get_schema`；无同名列须 `askUserQuestion` 确认映射或缺列方案，禁止撞错后静默建列 |
| `SingleSelect`/`MultipleSelect` 响应 `items: null` | 误用 `fields[].data.items` 包装 | 改为扁平 `fields[].items`；`get_schema` 验证选项非空 |
| Lookup `400100` Internal error | `prefer_id: false` 或 `linkField`/`lookupField` 用了字段名 | `prefer_id: true` + 字段 id 后重试 |
| 关联请求 ok 但未自动匹配 | `isAuto` 未落库（回读仍 `false`） | `get_schema` 确认后改 `update_records` 写记录 id；禁止宣称自动关联成功 |
| `list_records` 无 Lookup 键 | 接口常省略计算列 | `get_schema` 核对 Lookup 配置；勿因缺键重建字段 |
| `conflict` / `lock` / 写入冲突 | 并发写入同一数据表的多条记录导致锁竞争 | 指数退避重试（2s → 4s → 8s，最多 3 次）；批量写入时改为串行逐条 `dbsheet.update_records` / `dbsheet.create_records` |
| 新增记录落到表尾很后面 | 表内已有全空记录，直接 `create_records` 只会追加到末尾 | 先 `list_records` 复用全空记录 `update_records`，不足再 `create_records`；见「新增记录与全空记录复用」 |
| 组件创建成功但图表 `Invalid` / 统计为空 | `statisticsModuleSetting` 的 `dataSourceSheetId`/`fieldId` 与实际 schema 不符，或数据源无记录 | 重新 `get_schema` 核对 ID；`dashboard_chart_data` 返回 `Empty` 表示数据源无记录，提示用户先补数据 |

---

## 附录

### 字段类型

| 类型 | 说明 |
|------|------|
| `MultiLineText` | 多行文本 |
| `Number` | 数值 |
| `Currency` | 货币 |
| `Percentage` | 百分比 |
| `Date` | 日期 |
| `Time` | 时间 |
| `Checkbox` | 复选框 |
| `SingleSelect` | 单选项 |
| `MultipleSelect` | 多选项 |
| `Rating` | 等级 |
| `Complete` | 进度条 |
| `Phone` | 电话 |
| `Email` | 电子邮箱 |
| `Url` | 超链接 |
| `Contact` | 联系人 |
| `Attachment` | 附件 |
| `Link` | 关联 |
| `Note` | 富文本 |
| `Address` | 地址 |
| `AutoNumber` | 编号（自动填充） |
| `CreatedBy` | 创建者（自动填充） |
| `CreatedTime` | 创建时间（自动填充） |
| `LastModifiedBy` | 最后修改者（自动填充） |
| `LastModifiedTime` | 最后修改时间（自动填充） |
| `Formula` | 公式（自动计算） |
| `Lookup` | 查找引用（自动计算） |

### 视图类型

| 类型 | 说明 |
|------|------|
| `Grid` | 表格视图 |
| `Kanban` | 看板视图 |
| `Gallery` | 画册视图 |
| `Form` | 表单视图 |
| `Gantt` | 甘特视图 |
| `Calendar` | 日历视图 |

### 筛选规则（filter op）

| 操作符 | 适用字段类型 | 说明 |
|--------|-------------|------|
| `Equals` | 通用 | 等于 |
| `NotEqu` | 通用 | 不等于 |
| `Greater` | 数值、日期 | 大于 |
| `GreaterEqu` | 数值、日期 | 大于等于 |
| `Less` | 数值、日期 | 小于 |
| `LessEqu` | 数值、日期 | 小于等于 |
| `BeginWith` | 文本 | 开头是 |
| `EndWith` | 文本 | 结尾是 |
| `Contains` | 文本 | 包含 |
| `NotContains` | 文本 | 不包含 |
| `Intersected` | 单选、多选 | 选项包含指定值 |
| `Empty` | 通用 | 为空（`values` 可省略） |
| `NotEmpty` | 通用 | 不为空（`values` 可省略） |

### 错误响应

| 情况 | 响应示例 |
|------|---------|
| 命令不支持 | `{"msg":"core not support","result":"unSupport"}` |
| 内核错误 | `{"errno":-1880935404,"msg":"Invalid request","result":"ExecuteFailed"}` |
| HTTP 状态非 200 | 请求本身失败，检查 `file_id` 是否正确及鉴权信息 |
