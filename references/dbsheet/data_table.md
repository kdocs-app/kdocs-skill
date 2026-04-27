# 数据表管理

## 1. dbsheet.get_schema

#### 功能说明

获取多维表格文档的 Schema 信息，包括所有数据表、字段和视图的结构。可指定单个数据表 ID，不填则返回全部。



#### 调用示例

获取全部数据表结构：

```json
{
  "file_id": "string"
}
```

获取指定数据表结构：

```json
{
  "file_id": "string",
  "sheet_id": 1
}
```


#### 参数说明

- `file_id` (string, 必填): 多维表格文件 ID
- `sheet_id` (integer, 可选): 指定数据表 ID，不填则返回所有表
- `reserve_no_permission_sheet` (boolean, 可选): 是否保留无权限的表；默认值：`false`
- `show_very_hidden` (boolean, 可选): 是否显示深度隐藏的表；默认值：`true`
- `include_all_record_ids` (boolean, 可选): 是否返回所有记录 ID；默认值：`false`

#### 返回值说明

```json
{
  "detail": {
    "sheets": [
      {
        "id": 3,
        "name": "数据表",
        "primary_field_id": "B",
        "records_count": 100,
        "record_ids": ["A", "B"],
        "fields": [
          { "id": "B", "name": "名称", "type": "SingleLineText", "description": "字段备注" },
          { "id": "C", "name": "数量", "type": "Number", "description": "字段备注" }
        ],
        "views": [
          { "id": "B", "name": "表格视图", "type": "grid", "records_count": 10 }
        ]
      }
    ],
    "book_type": "db"
  },
  "result": "ok"
}

```

| 字段 | 类型 | 说明 |
|------|------|------|
| `detail.sheets[].id` | integer | 数据表 ID |
| `detail.sheets[].name` | string | 数据表名称 |
| `detail.sheets[].primary_field_id` | string | 主字段 ID |
| `detail.sheets[].records_count` | integer | 总记录数 |
| `detail.sheets[].record_ids` | array | 所有记录 ID（需开启 `include_all_record_ids`） |
| `detail.sheets[].fields` | array | 字段列表 |
| `detail.sheets[].views` | array | 视图列表 |
| `detail.book_type` | string | 文档类型标识，固定为 db |
| `result` | string | ok 表示成功 |


---

## 2. dbsheet.create_sheet

#### 功能说明

在多维表格文档中创建新的数据表，支持同时指定初始视图和字段。



#### 操作约束

- **后置验证**：get_schema 确认数据表已创建

#### 调用示例

创建带初始字段的数据表：

```json
{
  "file_id": "string",
  "name": "新数据表",
  "views": [
    {
      "name": "默认视图",
      "type": "Grid"
    }
  ],
  "fields": [
    {
      "name": "名称",
      "type": "SingleLineText"
    },
    {
      "name": "状态",
      "type": "SingleSelect",
      "items": [
        {
          "value": "待处理"
        },
        {
          "value": "已完成"
        }
      ]
    }
  ]
}
```


#### 参数说明

- `file_id` (string, 必填): 多维表格文件 ID
- `name` (string, 必填): 数据表名称
- `sync_type` (string, 可选): 同步类型；默认值：`None`
- `after_sheet_id` (integer, 可选): 插入到指定数据表之后
- `before_sheet_id` (integer, 可选): 插入到指定数据表之前
- `views` (array, 可选): 初始视图列表，每项包含 `name` 和 `type`
- `fields` (array, 可选): 初始字段列表，每项包含 `name` 和 `type`

#### 返回值说明

```json
{
  "detail": {
    "sheet": {
      "id": 6,
      "name": "新数据表",
      "primary_field_id": "L",
      "fields": [
        { "id": "L", "name": "名称", "type": "SingleLineText" }
      ],
      "views": [
        { "id": "J", "name": "默认视图", "type": "Grid" }
      ]
    }
  },
  "result": "ok"
}

```

| 字段 | 类型 | 说明 |
|------|------|------|
| `detail.sheet.id` | integer | 新建数据表 ID |
| `detail.sheet.name` | string | 数据表名称 |
| `detail.sheet.primary_field_id` | string | 主字段 ID |
| `detail.sheet.fields` | array | 字段列表 |
| `detail.sheet.views` | array | 视图列表 |
| `result` | string | ok 表示成功 |


---

## 3. dbsheet.update_sheet

#### 功能说明

修改数据表的名称或主字段设置。


#### 操作约束

- **前置检查**：get_schema 确认目标数据表存在

#### 调用示例

重命名数据表：

```json
{
  "file_id": "string",
  "sheet_id": 6,
  "name": "新名称"
}
```


#### 参数说明

- `file_id` (string, 必填): 多维表格文件 ID
- `sheet_id` (integer, 必填): 目标数据表 ID
- `name` (string, 可选): 新名称
- `prefer_id` (boolean, 可选): 是否使用字段 ID 作为 key
- `primary_field` (string, 可选): 主字段名称

#### 返回值说明

```json
{
  "detail": {
    "sheet": {
      "id": 6,
      "name": "新名称",
      "primary_field_id": "L",
      "fields": [],
      "views": []
    }
  },
  "result": "ok"
}

```

| 字段 | 类型 | 说明 |
|------|------|------|
| `detail.sheet.id` | integer | 数据表 ID |
| `detail.sheet.name` | string | 数据表名称 |
| `result` | string | ok 表示成功 |


---

## 4. dbsheet.delete_sheet

#### 功能说明

删除多维表格中的指定数据表。


#### 操作约束

- **前置检查**：get_schema 核对拟删数据表的名称和内容
- **用户确认**：删除数据表不可恢复，必须向用户确认数据表名称和 ID
- **禁止**：未经用户在对话中明确同意，禁止调用

#### 调用示例

删除数据表：

```json
{
  "file_id": "string",
  "sheet_id": 6
}
```


#### 参数说明

- `file_id` (string, 必填): 多维表格文件 ID
- `sheet_id` (integer, 必填): 要删除的数据表 ID

#### 返回值说明

```json
{
  "detail": {
    "sheet": { "id": 6 }
  },
  "result": "ok"
}

```

| 字段 | 类型 | 说明 |
|------|------|------|
| `detail.sheet.id` | integer | 已删除的数据表 ID |
| `result` | string | ok 表示成功 |


---

## 5. dbsheet.sheet_batch_create

#### 功能说明


**前置条件**：有创建数据表权限；单次批量条数与字段结构以文档上限为准。



#### 操作约束

- **后置验证**：建议 dbsheet.get_schema 核对

#### 调用示例

批量建表：

```json
{
  "file_id": "string",
  "body": {
    "sheets": []
  }
}
```


#### 参数说明

- `file_id` (string, 必填): 多维表格文件 ID
- `body` (object, 必填): 须含 sheets 数组

**body 根级必填**

| 字段 | 类型 | 说明 |
|------|------|------|
| `sheets` | array | 每个元素描述一个待建数据表（名称、字段、视图等），子字段以接口约定为准（batch-create-sheet） |


#### 返回值说明

```json
{
  "result": "ok",
  "detail": {}
}

```

| 字段 | 类型 | 说明 |
|------|------|------|
| `result` | string | ok 表示成功 |
| `detail` | object | 创建结果 |


---

## 6. dbsheet.sheet_batch_delete

#### 功能说明


**前置条件**：确认目标 `sheet_ids` 内数据均可删除；不可逆。



#### 操作约束

- **用户确认**：删除后表及记录不可恢复
- **幂等**：否

#### 调用示例

批量删除：

```json
{
  "file_id": "string",
  "body": {
    "sheet_ids": [
      2,
      3
    ]
  }
}
```


#### 参数说明

- `file_id` (string, 必填): 多维表格文件 ID
- `body` (object, 必填): 须含 sheet_ids

**body 根级必填**

| 字段 | 类型 | 说明 |
|------|------|------|
| `sheet_ids` | array[integer] | 待删除数据表 ID 列表 |


#### 返回值说明

```json
{
  "result": "ok",
  "detail": {}
}

```

| 字段 | 类型 | 说明 |
|------|------|------|
| `result` | string | ok 表示成功 |
| `detail` | object | 接口返回详情 |


---

