# 验收标准：三层闸门（不需要打开浏览器）

本主题最容易犯的错不是"报错"，而是**服务端全绿、界面是空的** ——
`save_flow` 成功、`validate_flow` 通过、`get_flow` 回读也正常，但打开编辑器会发现
某个参数显示成「请选择成员 / 该参数为必填项」。这类问题**不能靠"人去界面看一眼"来验收**：
使用者往往只有 MCP 工具、没有浏览器。固定走下面三层：

| 闸 | 怎么跑 | 拦什么 | 代价 |
|----|--------|--------|------|
| ① 形状自检 | `auto_sheet.validate_flow`（或 `save_flow` 的 `dry_run`） | 成员列表必须写成 `insert_values` 套 `flat_array`；变量表达式的 `biz_type` 必须落在模板声明的 `allow_types` 内；模板声明的凭证参数必须在 `action.auth` | 零副作用（离线规则） |
| ② 平台服务端校验 | `auto_sheet.set_trigger_status` + `check_only: true` | 平台自己的规则（`ValidateByFlowJson` / `CheckFlowParams`，与界面「配置检查」同源）。返回 `platform_check: passed/failed`；失败时**原因原样带回，且流程不会被开启** | 会**试开 → 立即关回**（不改变原有启用状态）；校验期间触发器短暂开启 |
| ③ 真实运行 + 节点日志 | **触发由用户来**（改一条数据 / 在界面手动运行）；agent 用 `auto_sheet.list_runs` 读节点级日志核对（传 `task_id` 看每个节点的 `exec_status`，`include_body: true` 看消息正文） | "存得下但跑不起来"、参数在运行时被判空、消息正文没渲染出值 | **真实执行**：见下方"为什么不能默认试运行" |

表述规范：过了 ① 只能说"形状没问题"；过了 ② 才能说"平台接受这份配置"；
只有真机跑过一次并看到节点成功，才能说"能跑"。**不要**把"我打开编辑器看了一眼"当成验收依据。
闸③的触发**永远由用户发起**（或用户明确要求 agent 去改某条记录），agent 只负责事后用
`auto_sheet.list_runs` 读运行记录：列表模式看「跑没跑、跑了几次、最近一次成功吗」，详情模式
（传 `task_id`）看「哪个节点失败、消息正文渲染出了什么」。改用户数据前先问清动哪条记录，用完还原。

### 为什么不能默认试运行

平台**没有沙箱**：手动运行就是真实执行。发消息给真人、邮件、群机器人 webhook（消息离开本平台）、
HTTP 请求（可能触发对方业务动作）、写/删记录（删除不可逆）、创建真实日程、执行脚本 —— 都会真的发生。
所以对"影响较大"的流程，试运行本身就是一次事故，不是验证。

**口径（硬约束）**：

- `auto_sheet.save_flow` 的启停语义**按 action 区分**：**新建**时默认落未启用草稿；
  **修改**时不传 `enabled` 会**沿用流程当前状态**（原本启用的仍在跑），显式 `enabled=true`
  需通过确认规则。返回里的 `enabled` / `requires_confirmation` / `next_step` 以实际状态为准 ——
  不要一律说成"只是草稿、尚未生效"。
- 开启触发器（`auto_sheet.set_trigger_status`）**必须由用户二次确认**：工具层强制 `enabled=true` 需带 `confirm=true`，
  没带就直接拒绝、连平台都不会调用。
- **只有用户明确要求**"直接开/直接跑"时，才可以直接带 `confirm=true`。

### 影响面预审（`impact`）—— 用来说明"会造成什么"

`save_flow` 与 `set_trigger_status` 的返回里都有 `impact`（**纯静态，不执行任何动作**）：

| 字段 | 内容 |
|------|------|
| `side_effects` | 副作用清单（会不会通知真人/改数据/请求外部；只读时明确说"只读"） |
| `receivers` | 收件人明细：几个指定成员 / 哪个群 / 邮件地址数 / 外部 webhook（第三方）/ 是否 @所有人；收件人取自记录字段时会标注"运行时才知道是谁" |
| `write_scope` | 写数据范围：仅筛选命中 vs **全表**；删除类标注**不可逆** |
| `risks` | 回环（"新增或修改"触发 + 写同表 = 会再次触发自己）、空值（动态收件人为空 → **静默发 0 条**）、跨流程（本次写入会连带触发同表上**已启用的**其它流程） |
| `runs_on_enable` | 开启后是否会被自动唤起（手动触发为 false） |

**确认流程**：拿到 `impact` → 用人话转述给用户（"这条流程会给你表里 3 位联系人各发一条消息，
且它改的这张表还被另外一条已启用流程监听，会连带触发它"）→ 用户同意后再 `set_trigger_status(enabled=true, confirm=true)`。
`impact` 为空或缺字段时不要编；说明"我看不到跨流程影响"即可。

### 已固化的形状规则（闸①的依据）

| 规则 | 为什么 |
|------|--------|
| 接收人/@人必须是 `insert_values{flat_array[…]}`：变量项写成**表达式对象**（`biz_type: person_array`），真实成员 id 写成**裸字符串** | 写成裸变量表达式时，编辑器把该参数判成空值（真机实测：界面「配置检查」1 个错误，而服务端全绿） |
| 引用记录字段/步骤输出列时带 `each`（接收人参数模板已声明 `default_method`，编译器会自动补） | 缺 `each` 时整列被拼成空串且**不报错** |
| 变量表达式的 `biz_type` 必须在模板 `allow_types` 内 | 写集合外的值，编辑器同样判空 |
| 模板 `auth` 块声明的参数（如 v9 的 `message_app_id`）必须放在 `action.auth` | 落到 `input` 会让编辑器「配置凭证」显示为空 |
| 引用"要改哪条记录"时用 `search_range: {"filter_type":"ever_result","ever_result":{"$var":"trigger_dataframe"}}` | 改"触发的那条记录"的标准写法；**不要**手拼 `record_id = 触发记录`（不被支持） |
| `filter_result` 对**新增记录**触发的订阅侧**不生效**；"字段不为空才触发"要用触发器的 `field_ids` | 新增事件的订阅不带 filter，只有回调侧重读时才用得上 |
# 工具组合速查

| 用户需求 | 推荐工具组合 |
|----------|-------------|
| 了解这个文件能做什么自动化 | `auto_sheet.get_context` |
| 看这个文件有哪些自动化 | `auto_sheet.list_flows` |
| 看某条流程具体做了什么 | `auto_sheet.list_flows` → `auto_sheet.get_flow` |
| 确认某条流程配置有没有问题 | `auto_sheet.list_flows` → `auto_sheet.validate_flow`（闸①）→ `auto_sheet.set_trigger_status` + `check_only: true`（闸②，平台判定） |
| 看某条流程跑没跑、最近一次成功吗 | `auto_sheet.list_flows` → `auto_sheet.list_runs`（列表：task_id / 状态 / 起止时间 / 触发人） |
| 核对某次运行的节点状态与消息正文 | `auto_sheet.list_runs` + `task_id`（节点级 `exec_status`；要核对正文再传 `include_body: true`） |
| 想知道某个需求该用哪个动作 | `auto_sheet.get_context` → `auto_sheet.list_actions`（可用 `query` 语义筛选） |
| 排查“流程没生效” | `auto_sheet.list_flows`（看 `enabled`）→ `auto_sheet.validate_flow` → `auto_sheet.get_flow` |
| 新建一条自动化 | `auto_sheet.get_context` → `auto_sheet.list_actions` → `auto_sheet.save_flow`（可先 `dry_run: true` 试算） |
| 修改已有自动化 | `auto_sheet.get_flow` → `auto_sheet.save_flow`（带 `revision`）→ 看 `verified` |
| 想知道这条流程会造成什么影响 | `auto_sheet.save_flow` / `auto_sheet.set_trigger_status` 返回的 `impact`（纯静态预审：通知谁/改什么/回环/跨流程） |
| 让自动化开始/停止运行 | `auto_sheet.list_flows` → `auto_sheet.validate_flow` → `auto_sheet.set_trigger_status(enabled=true, confirm=true)`（**先取得用户确认**；开启被平台拒绝时，`message` 里是平台给的原因） |
| 删除一条自动化 | `auto_sheet.list_flows`（回显流程名给用户确认）→ `auto_sheet.delete_flow`（`confirm: true`） |

---
# 结论表述规范

自动化话题最容易出错的是**把“已配置”说成“已生效”**。请按下列口径表述：

| 事实 | 可以怎么说 |
|------|------------|
| `enabled` 为 `true` 且校验通过 | 「这条流程已启用」 |
| `enabled` 为 `false` | 「这条流程已保存但未启用」/「处于草稿状态」——**还在等你确认要不要开** |
| 用户还没确认开启 | 只说「已建好，尚未运行」，并把 `impact` 里的影响面讲清楚；**不要**说"已生效/已开始提醒" |
| `save_flow` 返回 `verified: true` | 「已保存」（`enabled=false` 时补一句「还是草稿，需要我帮你开启吗」） |
| `save_flow` 返回 `verified: false` | 「写入结果未能确认，我先核对一下」——不要声称已保存 |
| `dry_run` 返回的 `diff` | 「如果创建，会是这些节点……」——这是预览，不是已创建 |
| 校验有 `warnings`（如缺接收人） | 「配置基本完整，但缺少 X，启用前需要补」 |
| 校验有 `errors` | 逐条列出 `message`，并说明对应的节点（可用 `step_id` 对应到 `get_flow` 的节点） |

**不要**在只做了查询或校验的情况下，声称已经帮用户「创建」「修改」或「启用」了流程。

---

## 常见错误处置

| 现象 | 原因 | 处置 |
|------|------|------|
| `type_mismatch` | 动作与文件类型不匹配 | 多维表格用 `db_*` 动作，表格用 `write_data` / `filter_rows` 等 |
| `unsafe_operation` | 修改/删除类动作未限定范围 | 提示用户补充筛选条件；这是数据安全红线，不要绕过 |
| 缺接收人（warning） | 消息动作没有真实接收人 | 说明只能停留在草稿状态，请用户补充接收人后再启用 |
| `trigger_invalid` | 触发器字段缺失 | 对照 `auto_sheet.get_context` 的 `triggers` 目录补齐字段 |
| `missing_actions` 非空（`list_actions`） | 平台侧尚未提供该动作的模板 | 属于平台覆盖缺口，不要据此承诺可用 |
| 找不到流程 | `flow_id` 不属于该文件 | 先 `auto_sheet.list_flows` 核对流程 ID |
| `unsafe_operation`（改写 `$raw` 节点） | 动了 `opaque_steps` 里的节点 | 从 `auto_sheet.get_flow` 原样带回该节点，不要改写 |
| `unknown_reference` | `$var` / 模板引用的 `key` 或字段名写错 | 用 `auto_sheet.get_flow` 核对 `steps[].key`，用 `list_actions` 的 `returns` 核对字段名 |
| 保存被拒：revision 冲突 | 流程在你读取之后被别人改过 | 重新 `auto_sheet.get_flow` 读取最新配置后再改 |
| `missing_required`（缺 `idempotency_key`） | `action=create` 未传幂等键 | 补一个稳定值（如"请求标识 + 流程名"），重试沿用 |
| 运行失败 `WOA_MSG_TEXT_OVERSIZE` | 消息正文超 3w 字符（`\|each` 展开富记录） | 按「正文有 3w 字符上限」缩小查找范围 / 少引用列 / 拆条发送 |
