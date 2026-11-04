# 参数写法由接口给出，不要凭记忆

每个参数能用哪种值写法，由 `auto_sheet.list_actions` 返回的 `value_kinds` 决定：

| `value_kinds` | 含义 |
|---------------|------|
| `original` | 直接传字面量 |
| `$var` | 可引用前序节点输出或触发数据字段 |
| `$sheet` / `$field` | 可引用数据表 / 字段 |
| `$file` / `$folder` | 可引用文件 / 文件夹 |
| `insert_values` | 字段值映射结构（较复杂） |
| `template_string` | 可混排文本与变量 |

**不要凭记忆假想参数名或写法**。需要某动作的参数细节时，先查 `auto_sheet.list_actions`。

> ⚠️ **`list_actions` 只给"写法"，不给"可选值"**。有些参数的合法取值来自后端表，
> 模板里的下拉是动态的、`list_actions` 拿不到，例如：
> 图像识别的 `ocr_type` / `ocr_word_list`（见 `auto_sheet.get_context` 的 `ocr_types`）、
> 日程的 `select_schedule_creator`（只有 `last_update_user` / `trigger`）。
> 这些必须按文档或 `get_context` 给的值写，猜必错。

> **账号类参数还有一层**：它们**不在** `list_actions` 的 `params` 里，而在动作模板的 `auth` 块
> （`list_actions` 会以 `auth_params` 告诉你有哪些）。可用账号在 **`get_context` 的 `accounts`** 小节，
> **按动作**给出（含账号 id、名称、类型）。
> - 写法：`"auth": { "<参数名>": <账号 id> }`（如 v9 的「由谁发送消息」）；
> - 若某动作显示"需要账号但一个都没配置"，**先让用户在编辑器里添加账号**，
>   不要编造账号 id，也不要假装能发出去。
# 触发器目录

`auto_sheet.get_context` 返回的 `triggers` 是 8 类触发器的静态目录，含每类需要的字段：

| 类型 | 说明 | 主要字段 |
|------|------|----------|
| `manual` | 手动触发 | 无 |
| `cron` | 定时触发 | `cron_type`（必填）、`time_formats`、按类型分支的 `single_date` / `every_week_day` / `every_month_day` |
| `db_event` | 数据表事件 | `event.type`、`event.sheet_id`、`event.field_ids` |
| `event` | 表格事件 | `event.type`、`event.sheet_id` |
| `webhook` | Webhook 触发 | `webhook.body_type`、`webhook.auth.method` |
| `button` | 点击按钮触发 | `button.sheet_id`、`button.btn_field`（已有按钮列）或 `button.new_button`（新建按钮列） |

`button.sheet_id` 与其它触发器同口径：数字 id 与 `{"$sheet": "表名"}` 都吃。按钮列**二选一**：

- `button.btn_field`：绑定**已有**按钮列（工具只建流程，一个字段都不动）；
- `button.new_button: {name, text?}`：**新建**按钮列（仅当用户明确要求时）—— 工具先
  `dbsheet.createFields` 建一列 `type=Button`、`customConfig={"actionType":0}`（=「触发自动化流程」，
  这是点击能触发流程的前提），再用回填的字段 id 当 `btn_field`。`dry_run` 不写表结构，只回
  「将新建按钮列…」的预警并给一个占位 `btn_field`，所以新建按钮列的流程也能安全预演。

按钮**只负责触发流程**：点击后要改数据（如把状态改成「已入库」）必须再挂写动作
（`db_update_rows` + `search_range={"filter_type":"ever_result","ever_result":{"$var":"trigger_dataframe"}}`）：

```json
{
  "trigger": {
    "type": "button",
    "button": {
      "sheet_id": {"$sheet": "采购入库"},
      "new_button": {"name": "入库通过", "text": "入库通过"}
    }
  },
  "steps": [
    {"key": "s1", "action": "db_update_rows", "label": "状态改为已入库",
     "params": {
       "sheet_id": {"$sheet": "采购入库"},
       "search_range": {"filter_type": "ever_result", "ever_result": {"$var": "trigger_dataframe"}},
       "key_values": {"状态": "已入库"}
     }}
  ]
}
```
| `wps_approval_create` | 轻审批新增数据时 | 无 |
| `wps_approval_change_status` | 轻审批状态变更时 | 无 |
# 节点里的 `$raw` 与 `opaque_steps`

`auto_sheet.get_flow` 读出的流程里，凡含复杂配置（变量引用、字段值映射、分支条件、模板字符串等）的节点，
无法在 DSL 中无损展开，会以 `{"$raw": <原始配置>}` 形式原样返回，节点标识记入 `opaque_steps`。

对这类节点：**保持原样**。可以阅读、可以向用户解释，但不要改写其中的 `$raw` 内容。

保存（`auto_sheet.save_flow`）时服务端会把这些节点与当前配置**逐字节比对**，
改一个字都会被拒绝（`unsafe_operation`）；删除这类节点同样会被拒绝。
需要调整时，请用该动作支持的参数写法重建该节点，或让用户在界面里改。

> **`$raw` 的语义是"原样保留"，不是"随便写"**：少数参数会以 `{"$raw": …}` 出现（包括**白名单内动作**，
> 例如界面建的图像识别流程里 `file_link` 的变量引用）。这类 `$raw` 只有**与库里那份逐字节相同**才被接受 ——
> 它能让"读出来再存回去"成立，但**不能**用来改配置。要改就按该动作的正常参数写法重写这个参数。
# 只读节点（`read_only_steps`）——白名单外动作

用户的流程是用界面建的，里面**可能有本工具不支持的动作**（邮件、脚本、日程、旧版消息动作等）。
这类节点读出来时长这样：

```jsonc
{ "key": "s3", "action": "send_email", "read_only": true,
  "read_only_reason": "动作 send_email 不在本期支持范围内：只能原样保留，不能改写或删除；如需改动请在编辑器里操作",
  "opaque": true, "params": { "to": "…" } }
```

规矩只有一条：**原样带回**。

- ✅ 可以：读它、向用户解释它、同时修改流程的**其它**部分（流程名、触发器、其它节点、增删支持的节点）；
- ❌ 不可以：改写它的任何字段、删除它、把它复制成第二个（会报 `unknown_action`）。

即"**能保留、不能新建、不能改**"。这条通道的目的很实际：以前只要流程里有一个我们没纳管的动作，
**连改流程名都会失败**；现在可以放心改别的部分。

> 若用户要的正是那个动作（例如"帮我加一个发邮件的步骤"），如实说明本工具暂不支持该动作，
> 建议在界面里添加，**不要**试图绕过校验。
# 系统变量（`${...}` 的一半）

正文、条件里除了引用前序节点（`$var`），还可以引用**运行时变量**。完整清单以
`auto_sheet.get_context` 的 `system_variables` 为准（含写法与约束），高频的几个：

| 变量 | 含义 | 使用条件 |
|---|---|---|
| `${trigger_time}` | 本次触发时间 | **任何触发器都有值** |
| `${var_time_current}` | 当前时间 | 通用 |
| `${var_time_recent_day.N}` | N 天前（如 `${var_time_recent_day.3}`） | 通用；**必须带数字** |
| `${var_trigger_user}` | 触发本次流程的人 | 仅记录/表格事件、按钮、轻审批触发 |
| `${var_trigger_button_user}` | 点击按钮的人 | 仅按钮触发 |
| `${trigger_dataframe.<列id>}` | 触发的那条记录 | 仅记录/表格事件、按钮、轻审批触发 |
| `${var_sheet_name}` | 触发所在工作表名 | 同上 |
| `${btn_field}` / `${approval_name}` | 按钮字段 id / 审批名称 | 仅按钮 / 仅轻审批触发 |
| `${trigger_webhook}` / `${trigger_webhook_info}` | Webhook 请求内容 / 元信息 | 仅 Webhook 触发 |
| `${var_dashboard.<数字id>}` | 仪表盘内容 | 多维表格；**必须带数字 id** |

三条硬约束：

1. `${var_time_recent_day}`、`${var_dashboard}` **漏写点后段会让 chatflow 编译期直接 panic**
   （报错无法自修复）——服务端已拦，但别写；
2. **触发器不匹配的变量在运行时根本不存在**（如定时触发里引用 `${trigger_dataframe}`，
   会报「选择步骤已删除」）；服务端会给出 warning，看到就要改；
3. 流程还可以有自己的**自定义变量**，形如 `${var_<13位数字>}`（名称与类型见
   `auto_sheet.get_flow` 的 `system_variables`）。

> 与 `$var` 的区别：`$var` 指向**本流程某个节点的输出**（要点后段写输出字段），
> 系统变量是**运行环境给的**（点后段是参数，如天数、仪表盘 id）。
> 两者都写错会**静默取空**（流程能存能启用、回读一切正常、跑起来那块是空的），
> 所以服务端对未知变量名会给出 warning —— 看到别忽略。

## 写链路：先草稿、后启用

写操作有三条必须遵守的规矩：

1. **新建默认落未启用草稿**（`enabled=false`）。服务端会直接拒绝"以 `enabled=true` 新建"，
   所以不要试图一步到位；确认无误后再用 `auto_sheet.set_trigger_status` 开启。
2. **只有 `verified=true` 才算保存成功**。这是服务端落库后回读确认的结果，
   不是写入接口的原始返回。`verified=false` 时必须如实告知用户并核对，不能含糊成"应该好了"。
3. **改流程前先读**：先 `auto_sheet.get_flow` 拿到当前 DSL（含 `opaque_steps`）与 `revision`，
   只改需要改的部分，并把 `revision` 一起回传做乐观锁。若返回冲突，说明别人刚改过，
   要重新读取后再改，**不要**为了"保存成功"而丢掉 `revision`。

创建流程必须带 `idempotency_key`（重试沿用同一个值）；重试时服务端会返回首次创建的 `flow_id`，
不会重复建流程。

**开启与删除都必须先取得用户明确确认**：开启会让流程真实开始发消息/写数据，
删除不可逆。调用前先把流程名回显给用户核对。

---

## 流程 DSL 速查（`auto_sheet.save_flow` 的 `flow`）

> 下面这些写法只对 `save_flow` 的 `flow` 有意义；参数名与必填项仍以 `auto_sheet.list_actions` 的实时返回为准。

## `flow` 结构

```jsonc
{
  "name": "订单超时提醒",          // 必填，≤30 字符（chatflow 硬限制）
  "func_desc": "每天检查超时订单",   // 选填
  "trigger": { "type": "cron", "cron": { "cron_type": "every_day", "time_formats": ["09:00"] } },
  "steps": [                      // 必填，≥1；**数组顺序即执行顺序**
    { "key": "s1", "action": "db_search_rows", "label": "查找内容",
      "params": { "sheet_id": { "$sheet": "订单" } } }
  ]
}
```

## `steps[]` 字段

| 字段 | 必填 | 说明 |
|------|------|------|
| `key` | 是 | 流程内唯一；用于 `$var` / `$template` 引用与错误定位 |
| `action` | 是 | 动作名，须来自 `auto_sheet.list_actions` |
| `label` | 否 | 节点显示名；缺省取动作中文名 |
| `params` | 否 | key 为动作入参名 |
| `children` | 否 | 仅 `branch` / `sub_branch` 使用 |
| `condition` | 否 | 仅 `sub_branch` 使用，与 `params` 同级 |
| `opaque` | 否 | **只读返回**：该节点含我们无法无损重建的配置，保存时必须原样带回 |
| `read_only` | 否 | **只读返回**：该动作不在本期支持范围内（见下节），只能原样带回，不能改写/删除/新建 |
# `$var` 的点后段必须是**列 id**（不是字段名）

chatflow 运行时取值走 `DataFrame.Load(key)` → 只比对**列 Id**，且取不到时**不报错、返回空值**。
两类文件的列 Id 来源不同：

| 文件类型 | 列 Id | 写法建议 |
|---|---|---|
| 多维表格 `.dbt` | **字段 ID**（如 `B`、`C`） | 可写字段名（如 `s1.负责人`），编译时会自动翻译成字段 ID；写 ID 也可以 |
| 表格 `.ksheet`/`.xlsx` | **A1 列字母**（由列位置推导，如 `B`） | 写 `s1.B` 这种列字母最稳；写表头文字不一定能翻译成列字母 |

系统列（`record_id` / `sheet_id` / `record_link` / `__record_count`）直接原样写。
字段名在当前工作表里找不到时，工具会**直接报错**并列出可用字段，而不是留到运行时静默取空。

> 别把"取哪个字段"和"筛选哪几列"搞混：`db_search_rows.fields` 这类**输入参数**用的是字段**名**
> （按 label 过滤），而 `$var` 的点后段用的是**列 id**。
# 参数值的写法

| 写法 | 含义 | 例子 |
|------|------|------|
| 字面量 | 直接写值 | `"message_content": "订单超时"` |
| `{ "$sheet": "表名" }` | 引用工作表 | `"sheet_id": { "$sheet": "订单" }` |
| `{ "$field": "表名.字段名" }` | 引用字段 | `"fields": [{ "$field": "订单.状态" }]` |
| `{ "$var": "s1.字段名" }` | 引用前序节点的输出字段 | `"person_id_list": { "$var": "s1.负责人" }` |
| `{ "$template": "「${s1.订单号\|each,plain}」超时" }` | 文本模板，`${…}` 内写引用；**引用列必须带 `\|each`**，按记录逐行展开 | 见上例 |
| `{ "$raw": {…} }` | 逃生舱：原样写入底层配置 | 仅少数动作的特定参数允许 |

> `$var` / `$template` 里的引用必须写成 **`step 的 key` + `.` + 输出字段名**。
> 字段名可以是 `auto_sheet.list_actions` 返回的 `returns[].key` 或 `returns[].label`；
> 写错字段会被服务端拒绝（`unknown_reference`），不会静默变成空值。
# 触发器（字段名与底层原生一致）

`type` 可选 `manual` / `cron` / `db_event` / `event` / `webhook` / `button` /
`wps_approval_create` / `wps_approval_change_status`，各自字段要求以
`auto_sheet.get_context` 返回的 `triggers` 目录为准。三条高频要求：

- `cron`：需 `cron.cron_type`（`single`/`every_day`/`every_week_day`/`every_month_day`/`every_hour`/`every_work_day`/`country_work_day`）
  与 `cron.time_formats`（如 `["09:00"]`）；`single` 另需 `single_date`，`every_week_day` 另需 `every_week_day`，`every_month_day` 另需 `every_month_day`
- `db_event`（多维表格）：需 `event.type`（3 新增 / 4 修改 / 5 到达指定时间 / 7 新增或修改）与 `event.sheet_id`；`type=5`/`7` 另需 `event.field_ids`
- `event`（表格）：需 `event.type`（1 修改任意列 / 2 修改指定列，另需 `ranges` / 6）与 `event.sheet_id`
# `$raw` 允许范围

只有下列动作的指定参数允许 `$raw`（其余一律用上表的写法）：

| 动作 | 参数 |
|------|------|
| `db_insert_rows` / `db_update_rows` | `key_values`（**优先用下面的 DSL 写法**，`$raw` 仅在照抄既有流程时用） |
| `send_woa_message_v7` / `v8` / `v9` | `message_buttons` |
| `branch` / `sub_branch` | `condition` |

## 校验失败（`code=400001`）

`data.errors[]` 每项含 `step_id` / `message` / `kind`，可直接用 `step_id`
回到 `auto_sheet.get_flow` 的 `steps[].key` 定位节点并按提示自修复。
