# 多维表格定时预警

> 在多维表格里建一条"定时扫描 + 命中就通知"的自动化流程（如每天 9 点检查超时/临期记录并提醒负责人）

**适用场景**：帮我建一个自动化，每天早上 9 点查一下快到期还没处理的记录，提醒负责人

**触发词**：每天提醒、定时提醒、定时预警、到期提醒、定时检查、自动提醒、每天早上

- 目标文件是多维表格（.dbt）
- 用户要的是"按时间自动跑 + 命中条件就通知"

**工具链**：`auto_sheet.get_context` → `auto_sheet.list_actions` → `auto_sheet.save_flow`（可先 dry_run）→ `auto_sheet.validate_flow` → `auto_sheet.set_trigger_status`

## 前置条件

- 已确认目标文件与工作表（不确定就先 auto_sheet.get_context）

## 涉及工具

| 工具 | 服务 | 用途 |
|------|------|------|
| `auto_sheet.get_context` | auto_sheet | 拿文件类型、工作表与字段、已有流程与容量上限 |
| `auto_sheet.list_actions` | auto_sheet | 查"查找内容"与"发送消息"两个动作的真实参数名与可用写法 |
| `auto_sheet.save_flow` | auto_sheet | 先 dry_run 试算，确认编排无误 |
| `auto_sheet.validate_flow` | auto_sheet | 校验落库后的配置 |
| `auto_sheet.set_trigger_status` | auto_sheet | 用户确认后开启 |

## 执行流程

**典型说法**：「帮我建个自动化，每天早上 9 点查一下快到期还没处理的记录，提醒负责人」
「每天下班前检查一遍逾期订单，发消息给负责人」

**完整编排**（多维表格 `.dbt`）：

```jsonc
{
  "name": "订单超时提醒",              // ≤30 字符
  "func_desc": "每天 9 点检查超时订单并通知负责人",
  "trigger": { "type": "cron",
               "cron": { "cron_type": "every_day", "time_formats": ["09:00"] } },
  "steps": [
    { "key": "s1", "action": "db_search_rows",
      "params": { "sheet_id": { "$sheet": "订单" },
                  // 预警类流程**必须真的筛选**：filter_type=all 等于"查全表"，
                  // 上游会把它判为未限定范围；界面上也会显示「满足条件的记录」却没有筛选行
                  "search_range": { "filter_type": "filter",
                                    "filter": { "mode": "AND", "criteria": [
                                      { "field": "状态", "op": "eq", "value": "待处理" } ] } },
                  "fields": ["订单号", "负责人", "状态"] } },
    { "key": "s2", "action": "send_woa_message_v9",
      "params": { "person_id_list": { "$var": "s1.负责人" },
                  "message_title": "订单超时",
                  "message_content": { "$template": "订单 ${s1.订单号|each,plain} 已超时，请及时处理" } } }
  ]
}
```

> 定时触发下**不存在** `trigger_dataframe`，所以正文要引用数据只能引用**前序步骤查到的列**
> （`${s1.订单号|each,plain}`）。引用列**必须带 `|each`**：
> 不加会静默渲染成空（消息照发、节点成功，但那段是空的），
> 所以开启后要核对一次正文与发送条数（N 条记录 → N 条消息），不能只看节点状态。

> 上面的参数名（`search_range` / `fields` / `person_id_list` / `message_content` …）是**示例**，
> 真实参数名与必填项以 `auto_sheet.list_actions` 的返回为准——不同动作/版本会变，不要硬套。

**执行顺序**

1. `auto_sheet.get_context`（可带 `sheet`）→ 拿工作表、字段、`limits.max_steps`、已有流程
2. `auto_sheet.list_actions`（带 `names`）→ 拿参数名、必填、`value_kinds`、`returns`
3. `auto_sheet.save_flow` + `dry_run: true` → 看 `diff` 与 `resolved`，确认节点与触发器描述
4. `auto_sheet.save_flow` + `action: "create"` + `idempotency_key` → **落未启用草稿**
5. `auto_sheet.validate_flow` → `ok=true` 才继续；有 `warnings` 要向用户说明
6. 用户确认后 `auto_sheet.set_trigger_status` + `enabled: true`

**参数写法速记**

| 需求 | 写法 | 例 |
|------|------|----|
| 指定工作表 | `{"$sheet": "表名"}` | `"sheet_id": {"$sheet": "订单"}` |
| 引用字段 | `{"$field": "表名.字段名"}` | `"fields": [{"$field": "订单.状态"}]` |
| 引用前序节点输出 | `{"$var": "s1.<列 id>"}` | `"person_id_list": {"$var": "s1.负责人"}`（多维表格写字段名即可，编译时自动转成字段 ID；表格请写 A1 列字母） |
| 文本里嵌变量 | `{"$template": "… ${s1.字段名} …"}` | 见上例 |
| 复杂结构（少数字段） | `{"$raw": {…}}` | 仅 `list_actions` 标明的参数可用，且必须原样带回 |

> `$var` 的点后段取的是**列 id**（多维表格=字段 ID，表格=A1 列字母），不是筛选用的字段名；
> 写字段名时工具会自动翻译，翻译不了会直接报错并列出可用字段，不会留到运行时静默取空。

**定时触发的坑**

- `cron_type` 用可读枚举：`single` / `every_day` / `every_week_day` / `every_month_day` / `every_hour` / `every_work_day` / `country_work_day`；
  `single` 另需 `single_date`，`every_week_day` 另需 `every_week_day`（如 `"1,3,5"`），`every_month_day` 另需 `every_month_day`
- 时间写 `time_formats: ["09:00"]`（可多个）；`skip_weekend: true` 表示跳过周末
- 只写"每天 9 点"这类时间，不要自己拼 cron 表达式

**消息节点：接收人、正文取值与发送条数**

- 接收人写扁平 id 列表即可（编译期自动套层）；但**必须是发送方所在公司的用户**，
  个人账号 id 会被静默过滤 → 节点成功却发出 0 条
- 正文引用数据**必须带 `|each`**：`{"$template": "「${trigger_dataframe.字段|each,plain}」有更新"}`
  —— 不加会静默渲染成空；字段写名称即可，编译期翻成列 id
- 节点返回「发送 [N] 条消息」：**N>0 才能说"已发送"**；N=0 要如实告知未送达

**收尾表述**

- 第 4 步之后只能说「已保存为未启用草稿」，**不能**说"已经生效"
- 第 6 步成功后要如实说明「流程已开启，会每天 9 点真实运行并发送消息」

## 注意事项

- **auto_sheet.get_context**：传 sheet 可只取该表的字段；这一步同时给出 automate_type 的判定来源
- **auto_sheet.list_actions**：只请求本次要用的动作（names），避免噪音；参数名一律以此为准，不要凭记忆写
- **auto_sheet.save_flow**：dry_run=true 只编译校验并返回 diff，不落库；建议每次新建前都跑一次
- **auto_sheet.validate_flow**：返回 `ok=true` 才算配置完整；“缺接收人/未限定范围”会是 error 或 warning，此时只能停留在草稿
- **auto_sheet.set_trigger_status**：enabled=true 会让流程真实运行，必须先取得用户明确确认
