# 数据透视表（创建与配置）

> 在 .xlsx/.ksheet 上按固定步骤创建透视表、配置行列值字段并回读验证（透视主路径；勿通读 references/sheet/data.md）

**适用场景**：用户要在云端表格上基于已有明细区域创建数据透视表，或调整行/列/值字段与汇总方式

**触发词**：透视、数据透视、pivot、透视表、行列汇总、分类汇总、数据透视表

- 用户要在 Excel/智能表格上新建或调整数据透视表
- 用户给出数据源区域（如 A1:F500）并要求按维度汇总

**工具链**：`sheet.get_sheets_info` → `sheet.create_pivot_table`（`sourceAddress`）→ `sheet.update_pivot_table`（`add_fields` / `add_data_field`）→ `sheet.get_pivot_tables` → 可选 `sheet.get_range_data` 回读透视区域

## 涉及工具

| 工具 | 服务 | 用途 |
|------|------|------|
| `sheet.get_sheets_info` | sheet | 取目标工作表 sheetId 与 rowTo/colTo，用于拼 sourceAddress 上界 |
| `sheet.create_pivot_table` | sheet | 用 sourceAddress（含表头行）创建空壳透视表；记下 stdout 的 name 作为 tableName |
| `sheet.update_pivot_table` | sheet | add_fields 配置行/列/页字段；add_data_field 添加值字段与 function（默认 sum） |
| `sheet.get_pivot_tables` | sheet | 先不传 tableName 确认 tables[].name；再传 tableName 查 fields 校验布局 |
| `sheet.get_range_data` | sheet | 可选：按 get_pivot_tables 返回的 tableRange 回读透视结果区域 |

## 执行流程

> 🎯 **何时读本文件**：用户要在 `.xlsx` / `.ksheet` 上**创建或改布局**数据透视表。本 workflow 为各渠道 Skill 共用主路径。**勿通读**整份 `SKILL.md` 工具表；某一步参数不明时，在 `SKILL.md` 上表选定工具后执行 `kdocs-cli help sheet.<tool>`。

## 硬性约束

1. **工作表 ID**：必须先 `sheet.get_sheets_info`，`worksheet_id` 只能用返回的 `sheetId`，禁止臆造。
2. **透视表名**：创建成功后 stdout 为 `{ "name": "...", "location": "..." }`；后续所有工具的入参 **`tableName` = 该 `name`**（或 `get_pivot_tables` 第一层 `tables[].name`）。勿把 HTTP 信封里的 `tableName` 字段名与响应 `name` 混淆。
3. **字段名**：`update_pivot_table` / `update_pivot_field` 的 `fieldName`、`*Fields` 列表须与**源数据表头列名**一致；`get_pivot_tables` 字段层以 `sourceName` 为准（见工具卡 notes）。
4. **优先批量布局**：新建后优先 `update_pivot_table` 的 `add_fields` + `add_data_field` 一次配好行列值；细调再用 `sheet.update_pivot_field`。布局不对时**不要**默认 `delete_pivot_table` 重建。
5. **成功判定**：CLI 成功时 stdout 为 **data 对象**（非 `{code,msg,data}` 信封）。若 stdout 含 `error` 键视为失败；同时结合 stderr 与 exit code（见 SKILL 全局约定）。

## 固定步骤（压缩试错轮次）

**1. 边界** — `sheet.get_sheets_info(file_id=…)` → 选定 `worksheet_id`，记下数据区 `rowTo`（0-based 末行）。

**2–3. 创建并布局** — `create_pivot_table` 的 `sourceAddress` 含表头（如 `A1:E100`）；stdout 的 `name` 作为后续 `tableName`。随后固定两次 `update_pivot_table`：先 `updateType=add_fields`（`rowFields` / `columnFields` / `pageFields` 为源表列名，CLI 传 JSON 数组字符串），再 `updateType=add_data_field`（`fieldName` + `function`，默认 `sum`）。

```bash
kdocs-cli sheet get-sheets-info --file-id FILE_ID
kdocs-cli sheet create-pivot-table --file-id FILE_ID --worksheet-id SHEET_ID --sourceAddress A1:E100
kdocs-cli sheet update-pivot-table --file add-fields.json
kdocs-cli sheet update-pivot-table --file add-data-field.json
```
（`add-fields.json` / `add-data-field.json` 为完整参数对象；字段含义见上文与 `kdocs-cli help sheet.update_pivot_table`。）

**4. 校验** — `sheet.get_pivot_tables`（仅 `file_id` + `worksheet_id`）确认 `tables`；再 `tableName=…` 查 `fields`。可选 `updateType=refresh` 后 `sheet.get_range_data` 读透视区域。

## 工具索引（按需打开工具卡，勿通读 data.md）

| 步骤 | 工具 |
|------|------|
| 创建 | `sheet.create_pivot_table` |
| 批量行列值 | `sheet.update_pivot_table` |
| 单字段细调 | `sheet.update_pivot_field` |
| 结构查询 | `sheet.get_pivot_tables` |
| 删除（慎用） | `sheet.delete_pivot_table` |
