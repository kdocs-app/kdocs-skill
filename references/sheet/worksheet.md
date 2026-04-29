# 工作表管理

## 1. sheet.get_sheets_info

#### 功能说明

获取指定表格文件的所有工作表信息，包含每个工作表的名称、索引、数据区域范围等。



> rowTo/colTo 比 maxRow/maxCol 更有参考价值，表示实际数据区域

#### 调用示例

获取工作表信息：

```json
{
  "file_id": "string"
}
```


#### 参数说明

- `file_id` (string, 必填): 文件 ID

#### 返回值说明

```json
{
  "sheetsInfo": [
    {
      "isEmpty": false,
      "colFrom": 0,
      "colTo": 5,
      "isVisible": true,
      "maxCol": 16383,
      "maxRow": 1048575,
      "rowFrom": 0,
      "rowTo": 50,
      "sheetId": 3,
      "sheetIdx": 0,
      "sheetName": "Sheet1",
      "sheetType": "et"
    }
  ]
}

```

| 字段 | 类型 | 说明 |
|------|------|------|
| `sheetsInfo[].sheetId` | integer | 工作表 ID |
| `sheetsInfo[].sheetIdx` | integer | 工作表索引 |
| `sheetsInfo[].sheetName` | string | 工作表名称 |
| `sheetsInfo[].sheetType` | string | 工作表类型（见下表） |
| `sheetsInfo[].isEmpty` | boolean | 是否为空 |
| `sheetsInfo[].isVisible` | boolean | 是否可见 |
| `sheetsInfo[].maxRow` | integer | 最大行数（工作表总容量） |
| `sheetsInfo[].maxCol` | integer | 最大列数 |
| `sheetsInfo[].rowFrom` | integer | 数据区域起始行 |
| `sheetsInfo[].rowTo` | integer | 数据区域结束行（比 `maxRow` 更有参考价值） |
| `sheetsInfo[].colFrom` | integer | 数据区域起始列 |
| `sheetsInfo[].colTo` | integer | 数据区域结束列 |

**sheetType 工作表类型：**

| sheetType | 说明 |
|-----------|------|
| `et` | 普通电子表格 |
| `db` | 数据表 |
| `airApp` | 应用表 |
| `oldDb` | 旧的数据表 |
| `dbDashBoard` | 数据表的仪表盘 |
| `etDashBoard` | 普通表格的仪表盘 |
| `workbench` | 工作台 |


---

## 2. sheet.create_airsheet_file

#### 功能说明

新建一个智能表格（.ksheet）文件。

`.ksheet` 不能通过通用 `create_file` 创建，应改用本工具。



**幂等性**：否 — 重复调用会创建多个文件，先确认是否已成功

> 创建完成后，可继续使用 `sheet.get_sheets_info`、`sheet.update_range_data` 等 `sheet.*` 工具操作内容

#### 调用示例

创建智能表格：

```json
{
  "type": "ksheet",
  "tname": "项目任务跟踪表.ksheet"
}
```


#### 参数说明

- `type` (string, 可选): 文件类型；默认值：`ksheet`，仅允许 `ksheet`。可选值：`ksheet`
- `tname` (string, 必填): 智能表格文件名，建议带 `.ksheet` 后缀

#### 返回值说明

```json
{
  "id": "airsheet_xxx",
  "name": "项目任务跟踪表.ksheet"
}

```

| 字段 | 类型 | 说明 |
|------|------|------|
| `id` | string | 新建文件 ID |
| `file_id` | string | 新建文件 ID（部分返回结构可能使用该字段） |
| `name` | string | 文件名 |


---

## 3. sheet.add_sheet

#### 功能说明

在指定表格文件中新增工作表。可指定名称、数量、插入位置和默认列宽。



**幂等性**：否 — 重复调用会创建多个工作表，先确认是否已成功

#### 调用示例

在末尾新增工作表：

```json
{
  "file_id": "string",
  "name": "销售数据",
  "col_width": 0,
  "position": {
    "end": true
  }
}
```

在指定工作表之后插入：

```json
{
  "file_id": "string",
  "name": "新工作表",
  "position": {
    "after_sheet_id": 3
  }
}
```

在指定工作表之前插入：

```json
{
  "file_id": "string",
  "position": {
    "before_sheet_id": 2
  }
}
```


#### 参数说明

- `file_id` (string, 必填): 文件 ID（路径参数 `{file_id}`）
- `col_width` (number, 可选): 列宽
- `name` (string, 可选): 工作表名
- `position` (object, 必填): 插入位置。子字段 `after_sheet_id`、`before_sheet_id`、`end` 均为可选，但应按业务需要指定其一（或按服务端约定组合），以确定插入到哪个 sheet 之前/之后或是否置于末尾。


**Path 参数：**

- `file_id`（string，必填）：文件 ID

**Body（`application/json`）：**

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `col_width` | number | 否 | 列宽 |
| `name` | string | 否 | 工作表名 |
| `position` | object | 是 | 插入位置 |

**position 对象：**

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `after_sheet_id` | integer | 否 | 在哪个 sheet 之后 |
| `before_sheet_id` | integer | 否 | 在哪个 sheet 之前 |
| `end` | boolean | 否 | 是否插入到最后 |


#### 返回值说明

```json
{
  "sheetId": 4
}

```

| 字段 | 类型 | 说明 |
|------|------|------|
| `sheetId` | integer | 新建的工作表 ID |


---

## 4. sheet.rename_sheet

#### 功能说明

重命名指定工作表。
适用于 Excel（.xlsx）和智能表格（.ksheet）。



**幂等性**：是

#### 调用示例

重命名工作表：

```json
{
  "file_id": "string",
  "sheetId": 1,
  "name": "新名称"
}
```


#### 参数说明

- `file_id` (string, 必填): 文件 ID
- `sheetId` (integer, 必填): 目标工作表 ID
- `name` (string, 必填): 新工作表名称

#### 返回值说明

```json
{}

```


---

## 5. sheet.delete_sheets

#### 功能说明

删除一个或多个工作表。
适用于 Excel（.xlsx）和智能表格（.ksheet）。



#### 操作约束

- **前置检查**：`sheet.get_sheets_info` 核对拟删工作表名称与 sheetId
- **用户确认**：删除工作表不可恢复，必须向用户确认工作表名称和 ID

**幂等性**：是

#### 调用示例

删除多个工作表：

```json
{
  "file_id": "string",
  "sheetIds": [
    1,
    3
  ]
}
```


#### 参数说明

- `file_id` (string, 必填): 文件 ID
- `sheetIds` (array[integer], 必填): 要删除的工作表 ID 列表

#### 返回值说明

```json
{}

```


---

## 6. sheet.copy_worksheet

#### 功能说明

复制指定工作表。

适用于按模板快速生成副本、保留原始工作表不动的场景。



#### 操作约束

- **后置验证**：get_sheets_info 确认副本已创建

**幂等性**：否 — 重复调用会生成多个副本，先确认是否已成功

> 若需要复制首个工作表的特殊布局，可按需设置 `copy_first_sheet`

#### 调用示例

复制指定工作表：

```json
{
  "file_id": "string",
  "worksheet_id": 7
}
```


#### 参数说明

- `file_id` (string, 必填): 文件 ID
- `worksheet_id` (integer, 必填): 工作表 ID
- `copy_first_sheet` (boolean, 可选): 是否复制第一个工作表

#### 返回值说明

```json
{
  "worksheet_id": 9,
  "name": "任务模板 副本"
}

```

| 字段 | 类型 | 说明 |
|------|------|------|
| `worksheet_id` | integer | 新复制出的工作表 ID |
| `name` | string | 新工作表名称 |


---

## 7. sheet.update_worksheet

#### 功能说明

更新工作表名称或调整工作表顺序位置。

支持重命名工作表，也支持相对于另一张工作表前后移动。



**幂等性**：是

> `name` 与移动参数可单独使用，也可按实际接口能力组合使用

#### 调用示例

重命名工作表：

```json
{
  "file_id": "string",
  "worksheet_id": 7,
  "name": "任务总览"
}
```

将工作表移动到另一张表之后：

```json
{
  "file_id": "string",
  "worksheet_id": 7,
  "move_sheet_id": 3,
  "move_type": "sheet_move_type_after"
}
```


#### 参数说明

- `file_id` (string, 必填): 文件 ID
- `worksheet_id` (integer, 必填): 工作表 ID
- `name` (string, 可选): 新工作表名称
- `move_sheet_id` (integer, 可选): 参照工作表 ID
- `move_type` (string, 可选): 移动类型：`sheet_move_type_before` / `sheet_move_type_after`

#### 返回值说明

```json
{}

```


---

