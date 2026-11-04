# 表格定时回写

> 在表格（.ksheet/.xlsx）里建一条"定时抓取/计算后写回指定区域"的自动化流程

**适用场景**：帮我建个自动化，每天早上把某个表的数据汇总后写到这张表的汇总区

**触发词**：定时写回、每天汇总、定时填表、自动写入表格、表格自动化、定时更新数据

- 目标文件是表格（.ksheet / .xlsx / .xls），不是多维表格
- 用户要的是"把结果写到表格区域"

**工具链**：`auto_sheet.get_context` → `auto_sheet.list_actions` → `auto_sheet.save_flow` → `auto_sheet.validate_flow` → `auto_sheet.set_trigger_status`

## 前置条件

- 已确认目标文件与工作表（不确定就先 auto_sheet.get_context）

## 涉及工具

| 工具 | 服务 | 用途 |
|------|------|------|
| `auto_sheet.get_context` | auto_sheet | 确认这是表格文件、拿到工作表与容量上限 |
| `auto_sheet.list_actions` | auto_sheet | 查表格动作（filter_rows / write_data 等）的真实参数名与写法 |
| `auto_sheet.save_flow` | auto_sheet | 先 dry_run 试算 |
| `auto_sheet.validate_flow` | auto_sheet | 校验配置（重点看写入区域是否明确） |
| `auto_sheet.set_trigger_status` | auto_sheet | 用户确认后开启 |

## 执行流程

**典型说法**：「每天早上把数据汇总写到这张表」「定时把某个区域的内容更新一下」

**表格与多维表格的动作集不通用**

| 文件类型 | `automate_type` | 可用动作 |
|---|---|---|
| 多维表格 `.dbt` | `db_sheet` | `db_search_rows` / `db_insert_rows` / `db_update_rows` / `db_delete_rows` … |
| 表格 `.ksheet` / `.xlsx` / `.xls` | `et_sheet` | `write_data` / `filter_rows` / `create_sheet` … |

消息、通知、延迟、HTTP 请求类动作两类通用。**名称以 `db_` 开头的动作只能用于多维表格**，
`write_data` / `filter_rows` 等只能用于表格——放错会在校验时报 `type_mismatch`。

**写入类参数通常是"区域"而不是"表名"**

表格动作里的目标位置往往是区域（`et_range`），写法与多维表格的 `sheet_id` 不同：

- 优先用 `list_actions` 给出的 `value_kinds` 判断该参数能不能用 `$sheet`
- `$sheet` 给的是工作表标识，适合"某张表"；精确到单元格区域通常要用区域写法（如 `"$A1:C10"`）
- **区域必须明确**：`write_data` 属于危险写，未限定范围会报 `unsafe_operation`

> 不同版本的表格动作参数名差异较大，务必以 `auto_sheet.list_actions` 的实时返回为准，
> 不要照搬其它文档里的参数名。

**编排骨架**

```jsonc
{
  "name": "每日汇总写回",              // ≤30 字符
  "trigger": { "type": "cron",
               "cron": { "cron_type": "every_day", "time_formats": ["08:30"] } },
  "steps": [
    { "key": "s1", "action": "filter_rows",
      "params": { /* 取数：区域 + 条件，参数名见 list_actions */ } },
    { "key": "s2", "action": "write_data",
      "params": { /* 写入：目标区域 + 内容 + 写入方式；区域必须明确 */ } }
  ]
}
```

**执行顺序**

1. `auto_sheet.get_context` → 确认 `automate_type=et_sheet`、工作表与 `limits.max_steps`
2. `auto_sheet.list_actions`（带 `names`）→ 参数名、必填、`value_kinds`
3. `auto_sheet.save_flow` + `dry_run: true` → 看 `diff`
4. `auto_sheet.save_flow` + `action: "create"` + `idempotency_key` → **落草稿**
5. `auto_sheet.validate_flow` → `ok=true`；若报 `unsafe_operation` 说明写入范围没限定，必须补
6. 用户确认后 `auto_sheet.set_trigger_status` + `enabled: true`

**和数据安全的边界**

- 写回会**覆盖目标区域原有内容**，开启前要把"写到哪、覆盖什么"讲清楚，让用户确认
- 定时写回一旦开启会按点真实执行；用户只想要一次性结果时不要建流程（直接取数给他）
- 拿不准目标区域时，宁可先只做取数（`filter_rows`）并让用户确认范围，再补写入节点
