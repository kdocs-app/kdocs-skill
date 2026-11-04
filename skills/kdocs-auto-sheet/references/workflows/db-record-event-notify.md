# 记录变更事件通知

> 在多维表格里建一条"记录发生变化就通知"的事件触发流程（新增/修改/到达指定时间）

**适用场景**：帮我在有人新增记录时自动发消息；或者某个状态被改成"已超时"就通知负责人

**触发词**：新增记录时、修改记录时、状态变了就通知、记录变更提醒、一旦修改、有人填了就通知

- 目标文件是多维表格（.dbt）
- 用户描述的是"由数据变化触发"，而不是"按时间触发"

**工具链**：`auto_sheet.get_context` → `auto_sheet.list_actions` → `auto_sheet.save_flow` → `auto_sheet.validate_flow` → `auto_sheet.set_trigger_status`

## 前置条件

- 已确认要监听哪张表、哪些字段（字段没定就用 auto_sheet.get_context 拿字段清单）

## 涉及工具

| 工具 | 服务 | 用途 |
|------|------|------|
| `auto_sheet.get_context` | auto_sheet | 拿工作表、字段与已有流程 |
| `auto_sheet.list_actions` | auto_sheet | 查通知动作的参数名与接收人写法 |
| `auto_sheet.save_flow` | auto_sheet | 创建事件触发流程（默认草稿） |
| `auto_sheet.validate_flow` | auto_sheet | 校验事件与节点配置 |
| `auto_sheet.set_trigger_status` | auto_sheet | 用户确认后开启（会真实订阅事件） |

## 执行流程

**典型说法**：「有人新增记录就通知我」「状态被改成已超时就提醒负责人」
「谁改了金额那列就发消息给我」

**先决定条件放在哪里，不要默认用分支**：

| 条件形态 | 位置 |
|---|---|
| 非公式字段的简单比较（如 `状态 = 已入库`、`数量 > 100`），多个条件可用 AND/OR | **优先放在触发器筛选 `event.filter_result`**，步骤里只做命中后的动作 |
| 同一次触发要分流执行不同动作 | 用 `branch` / `sub_branch` |
| 公式列条件（如 `总额 = 数量 × 单价`） | 触发器筛选会取改动前快照；用 `delay` 后接 `db_search_rows` 的筛选条件 |
| 严格表达「从 A 改成 B」 | 平台条件只看改后值；如必须实现，需额外回写「上次状态」列 |

触发器筛选的 `filter_result` 与 `queryRanges` 目前是界面 canonical 原样透传，没有 DSL 糖；
`sheetId` 用 `auto_sheet.get_context` 返回的数字 ID；`fieldIds`/`field_ids` 用字段**短 ID**（如 `B`、`C`），不要传表名/字段名。

**组合边界**：type=4「修改记录」+触发器筛选是平台缺陷。`db_search_rows → branch(__record_count) → delay` 链路在手动触发下 `check_only` 与运行均通过；type=3「新增记录」、type=5「到达指定时间」、type=7「新增或修改记录」接同一分支链路时，虽然本地 `validate_flow` 通过，平台 `check_only` 都返回 400001。三类触发器各自只接只读查找的对照均通过，type=7 + 筛选 + 只读查找也通过，因此当前证据指向 type=3/5/7 记录事件触发器与分支组合的边界，不是筛选本身不可用。type=4 不带筛选、后接查找结果分支的验收流程也已运行通过。

```jsonc
{
  "type": "db_event",
  "event": {
    "type": 7,
    "sheet_id": { "$sheet": "订单" },
    "field_ids": ["C", "H"],            // 监听字段；type=7 必须显式给出
    "queryRanges": [
      { "sheetId": 1, "fieldIds": ["B", "C", "H"] }
    ],
    "filter_result": {
      "opType": "And",                   // 同时满足；任一满足用 Or
      "sheetId": 1,
      "filterFields": [
        {
          "fieldType": "Number", "id": "H", "name": "数量",
          "numberFormat": "0_", "op": "Greater", "sheetStId": 1,
          "isHidden": false, "isNoPermission": false,
          "values": [{ "type": "Text", "value": "100" }]
        },
        {
          "fieldType": "SingleSelect", "id": "C", "name": "状态",
          "numberFormat": "@", "op": "Intersected", "sheetStId": 1,
          "isHidden": false, "isNoPermission": false,
          "values": [{ "type": "Text", "value": "已入库" }]
        }
      ]
    }
  }
}
```

**事件触发的结构**（`trigger.type = "db_event"`）：

```jsonc
{
  "type": "db_event",
  "event": {
    "type": 3,                                  // 见下表
    "sheet_id": { "$sheet": "订单" },
    "field_ids": [{ "$field": "订单.状态" }]     // type=5/7 必填；type=4 需同时给 sheet_all_field: 1
  }
}
```

| `event.type` | 含义 | 字段要求 |
|---|---|---|
| `3` | 新增记录时 | `sheet_id` |
| `4` | 修改记录时 | `sheet_id`；`sheet_all_field: 0` + `field_ids: []` = 任意列修改；`sheet_all_field: 1` + `field_ids` = 仅指定列 |
| `5` | 到达记录中的指定时间 | `sheet_id` + `field_ids`（指定日期列） |
| `7` | 新增或修改记录时 | `sheet_id` + `field_ids` |

> 表格文件（`.ksheet`/`.xlsx`）的 `event` 类型是另一套（1 修改任意列 / 2 修改指定列 / 6），
> 放错会直接被服务端拒绝。拿不准就用 `auto_sheet.get_context` 的 `triggers` 目录核对。

**执行顺序**

1. `auto_sheet.get_context` → 工作表与字段（事件要绑到具体表和列）
2. `auto_sheet.list_actions`（带 `names`）→ 通知动作的真实参数名
3. `auto_sheet.save_flow`（`action: "create"` + `idempotency_key`，`enabled` 保持缺省）→ 落草稿
4. `auto_sheet.validate_flow` → `ok=true`
5. 用户确认后 `auto_sheet.set_trigger_status` + `enabled: true`

**和定时预警的区别（选错会返工）**

| 用户意图 | 触发器 | 走哪个流程 |
|---|---|---|
| 「每天/每周 X 点检查一遍」 | `cron` | 多维表格定时预警 |
| 「一有变化就通知」 | `db_event` | 本流程 |
| 「点了按钮才跑」 | `button`（需 `button.sheet_id` + `button.btn_field`） | 本流程（把 trigger 换成 button） |

**通知内容常见写法**

- 接收人：`{"$var": "s1.负责人"}`（引用记录里的负责人字段）；也可以直接给固定用户
- 内容：两种都可用，都**必须带 `|each`**（不加会静默渲染成空）
  - 引用**触发记录**的列（事件触发下才有 `trigger_dataframe`）：
    `{"$template": "「${trigger_dataframe.订单号|each,plain}」状态已变更为 ${trigger_dataframe.状态|each,plain}"}`
  - 引用**查找步骤**查到的列：`{"$template": "超时订单：${s1.订单号|each,plain}"}`
    （按记录逐行展开：N 条记录 → N 条消息）
- 想同时 @ 人：用动作暴露的 `at_type` / `at_person_list` 参数（以 `list_actions` 为准）

**收尾表述**

- 创建后只能说「已保存为未启用草稿」；开启成功后才可说明「以后每次新增/修改都会触发」

## 注意事项

- **auto_sheet.get_context**：事件类型与字段监听都需要真实 sheet_id / field_ids，先用这一步拿到
- **auto_sheet.set_trigger_status**：开启后每次记录变更都会真实触发，必须先取得用户确认
