# 数据操作

## 1. sheet.get_range_data

#### 功能说明

获取指定工作表中某个矩形区域内的单元格数据。行列索引均为 0-based。
请求参数使用 `worksheet_id` 和 `range` 对象。
**`range` 必须为对象，即使只读取一个单元格也必须包裹在对象中传入，不可传数组。**



#### 操作约束

- **前置检查**：使用该工具前必须先调用get_sheets_info确认要操作的工作表id，不得自行捏造工作表id。

**幂等性**：是

> 行列索引均为 0-based；读取整张表时先用 sheet.get_sheets_info 获取 range.rowTo / range.colTo 上限
> isCellPic=true 时，单元格为图片；picData（在线文件）和 sha1（本地图片）二选一返回
> originalCellValue 返回公式栏原始值，cellText 返回显示值；fmlaText 仅在含公式时返回

#### 调用示例

读取 A1:F11 区域（前 11 行、前 6 列）：

```json
{
  "file_id": "VsdfG0001234567",
  "worksheet_id": 3,
  "range": {
    "rowFrom": 0,
    "rowTo": 10,
    "colFrom": 0,
    "colTo": 5
  }
}
```


#### 参数说明

- `file_id` (string, 必填): 文件 ID
- `worksheet_id` (integer, 必填): 工作表 ID
- `range` (object, 必填): 选区范围（必须为对象，即使只读取一个单元格也必须包裹在对象中传入，不可传数组），行列索引均为 0-based

#### 返回值说明

```json
{
  "result": "ok",
  "detail": {
    "rangeData": [
      {
        "cellText": "test",
        "colFrom": 0,
        "colTo": 0,
        "isCellPic": false,
        "numFormat": "G/通用格式",
        "originalCellValue": "test",
        "rowFrom": 0,
        "rowTo": 0,
        "fmlaText": "=A1"
      },
      {
        "cellText": "111",
        "colFrom": 1,
        "colTo": 1,
        "isCellPic": false,
        "numFormat": "G/通用格式",
        "originalCellValue": "111",
        "rowFrom": 0,
        "rowTo": 0
      },
      {
        "cellText": "332",
        "colFrom": 0,
        "colTo": 0,
        "isCellPic": false,
        "numFormat": "G/通用格式",
        "originalCellValue": "332",
        "rowFrom": 1,
        "rowTo": 1
      },
      {
        "cellText": "=DISPIMG(\"ID_018590616CC643F796F3CB58682DEA85\",1)",
        "colFrom": 1,
        "colTo": 1,
        "isCellPic": true,
        "numFormat": "G/通用格式",
        "originalCellValue": "=DISPIMG(\"ID_018590616CC643F796F3CB58682DEA85\",1)",
        "picData": "MSM5KAQABI",
        "sha1": "6CC643F796F3CB58682DEA85",
        "rowFrom": 1,
        "rowTo": 1,
        "tag": "attachment"
      }
    ]
  }
}

```

| 字段 | 类型 | 说明 |
|------|------|------|
| `result` | string | 'ok' 表示成功 |
| `detail.rangeData` | array[object] | 单元格数据数组，每项对应选区内一个有数据的单元格 |


---

## 2. sheet.delete_range_data

#### 功能说明

删除指定区域的行或列，删除后将其余内容上移或左移。适用于 Excel（.xlsx）和智能表格（.ksheet）。


#### 操作约束

- **前置检查**：`sheet.get_range_data` 核对拟删行/列范围内现有数据
- **前置检查**：使用该工具前必须先调用get_sheets_info确认要操作的工作表id，不得自行捏造工作表id。
- **用户确认**：删除行或列会移位其余内容且难以恢复，必须向用户确认范围与影响

**幂等性**：是

> 行列索引均为 0-based

#### 调用示例

删除范围数据（默认上移）：

```json
{
  "file_id": "string",
  "worksheet_id": 3,
  "range_data": [
    {
      "col_from": 0,
      "col_to": 0,
      "row_from": 0,
      "row_to": 0
    }
  ],
  "shift_type": "shift_up"
}
```


#### 参数说明

- `file_id` (string, 必填): 文件 ID
- `worksheet_id` (integer, 必填): 工作表 ID
- `range_data` (array[object], 必填): 范围数组；调用方需保证参数在行列最大最小值范围内，超过则执行失败。最大值可通过 `sheet.get_sheets_info` 获取
- `shift_type` (string, 可选): 移动方式，默认向上移动。可选值：`shift_up` / `shift_left`；默认值：`shift_up`

#### 返回值说明

```json
{
  "code": 0,
  "msg": "string"
}

```

| 字段 | 类型 | 说明 |
|------|------|------|
| `code` | integer | 响应码 |
| `msg` | string | 错误描述，code 非 0 时返回失败原因 |


---

## 3. sheet.add_row

#### 功能说明

在工作表已使用区域末尾追加一行数据，支持写入文本/公式和图片。
适用于 Excel（.xlsx）和智能表格（.ksheet）。
file_id和worksheet_id为必填参数，不允许为空，worksheet_id必须为数字。
range_data.op_type的枚举值只有cell_operation_type_formula，cell_operation_type_picture，cell_operation_type_format，cell_operation_type_merge四种，不允许自主定义。



#### 操作约束

- **前置检查**：使用该工具前必须先调用get_sheets_info确认要操作的工作表id，不得自行捏造工作表id。

**幂等性**：否 — 重复调用会插入多行，先确认是否已成功

> 行列索引均为 0-based

#### 调用示例

追加一行（文本与图片）：

```json
{
  "file_id": "string",
  "worksheet_id": 3,
  "range_data": [
    {
      "op_type": "cell_operation_type_formula",
      "col": 0,
      "formula": "值1"
    },
    {
      "op_type": "cell_operation_type_formula",
      "formula": "值2"
    },
    {
      "op_type": "cell_operation_type_picture",
      "col": 3,
      "cell_pic_info": {
        "width": 120,
        "height": 120,
        "tag": "sheet_pic_type_url",
        "pic_content": "https://example.com/image.png"
      }
    }
  ]
}
```


#### 参数说明

- `file_id` (string, 必填): 文件 ID
- `worksheet_id` (integer, 必填): 工作表 ID
- `range_data` (array[object], 可选): 新行各列的数据，按列顺序追加至已使用区域末尾，不可为空数组。

#### 返回值说明

```json
{
  "code": 0,
  "msg": "string"
}

```

| 字段 | 类型 | 说明 |
|------|------|------|
| `code` | integer | 响应码 |
| `msg` | string | 错误描述，code 非 0 时返回失败原因 |


---

## 4. sheet.find_range_data

#### 功能说明

遍历并筛选工作表中的记录，支持分页、条件筛选、文本搜索和去重。

**工具选择提示**：
- `sheet.find_range_data`：用于“先筛选再返回结果”，支持 `filter`、`search`、`duplicates`、分页和总数统计。
- `sheet.get_range_data`：用于“直接读取固定矩形范围数据”，不做筛选、搜索、去重或分页。
使用该工具时必须传递worksheet_id和filter参数，worksheet_id必须为数字，若要查询所有数据则filter传空数组但必须传这个参数。
该工具仅适用于：Excel（.xlsx）和智能表格（.ksheet），在操作多维表格（.dbt）时使用"dbsheet"相关工具，不允许使用该工具操作多维表格。



#### 操作约束

- **前置检查**：使用该工具前必须先调用get_sheets_info确认要操作的工作表id，不得自行捏造工作表id。

**幂等性**：是

> 分页说明：通过 `page.page` 递增翻页，`total` 为结果总数。

#### 调用示例

按条件筛选并分页：

```json
{
  "file_id": "string",
  "worksheet_id": 3,
  "range": {
    "col_from": 0,
    "col_to": 10,
    "row_from": 0,
    "row_to": 50000
  },
  "page": {
    "page": 1,
    "page_size": 100
  },
  "filter": {
    "condition": [
      {
        "col": 0,
        "info": [
          {
            "value": "string"
          }
        ],
        "mode": "filter_mode_and"
      }
    ],
    "duplicates": {
      "col": [
        0
      ]
    },
    "search": [
      {
        "col": 0,
        "value": [
          "string"
        ]
      }
    ]
  },
  "ignore_hidden_cell": true,
  "option_cols": [
    0
  ],
  "show_total": true
}
```


#### 参数说明

- `file_id` (string, 必填): 文件 ID
- `worksheet_id` (integer, 必填): 工作表 ID
- `range` (object, 必填): 筛选区域
- `page` (object, 可选): 分页参数（可选）
- `filter` (object, 必填): 筛选条件；`condition` 传空数组时不筛选，输出全部
- `ignore_hidden_cell` (boolean, 可选): 是否忽略隐藏单元格；默认值：`false`
- `option_cols` (array[integer], 可选): 需返回选项统计的列坐标（相对于 `range`）
- `show_total` (boolean, 可选): 是否返回总数；默认值：`false`

#### 返回值说明

```json
{
  "code": 0,
  "msg": "string",
  "data": {
    "merge_range_data": [
      {
        "cell_text": "string",
        "col_from": 0,
        "col_to": 0,
        "is_cell_pic": true,
        "num_format": "string",
        "original_cell_value": "string",
        "pic_content": "string",
        "pic_data": "string",
        "row_from": 0,
        "row_to": 0,
        "sha1": "string",
        "tag": "string"
      }
    ],
    "option_col": [
      {
        "col": 0,
        "texts": [
          {
            "count": 0,
            "origin": "string",
            "text": "string"
          }
        ]
      }
    ],
    "range_data": [
      {
        "cell_text": "string",
        "col_from": 0,
        "col_to": 0,
        "is_cell_pic": true,
        "num_format": "string",
        "original_cell_value": "string",
        "pic_content": "string",
        "pic_data": "string",
        "row_from": 0,
        "row_to": 0,
        "sha1": "string",
        "tag": "string"
      }
    ],
    "result_type": 0,
    "total": 0
  }
}

```

| 字段 | 类型 | 说明 |
|------|------|------|
| `code` | integer | 响应码 |
| `msg` | string | 错误描述，code 非 0 时返回失败原因 |
| `data.merge_range_data` | array[object] | 当前区域包含的合并单元格数据 |
| `data.option_col` | array[object] | 选项结果 |
| `data.range_data` | array[object] | 区域数据，格式与 `get_range_data` 一致；未设置筛选条件时返回全部数据 |
| `data.result_type` | integer | 本次是否成功，`0` 失败，`1` 成功 |
| `data.total` | integer | 结果总数 |


---

## 5. sheet.get_attachment_url

#### 功能说明

上传附件到文件，返回上传结果与对象标识（`object_id`）。
使用 `multipart/form-data` 方式上传。

支持普通上传，以及 `local_cover`（本地官方推荐模板）和 `user_cover`（用户上传封面图）场景。



> 请求需使用 `multipart/form-data` 提交参数
> `url` 与 `file` 必须二选一
> 当 `source_type=local_cover` 时，必须传 `cover_id`
> 当 `source_type=user_cover` 时，必须传 `scale`
> 本工具为单次全量上传，无分片机制

#### 调用示例

普通上传（URL）：

```json
{
  "file_id": "12345",
  "filename": "6789.jpg",
  "url": "https://img.qwps.cn/example.jpg",
  "Content-Type": "multipart/form-data"
}
```

local_cover 上传：

```json
{
  "file_id": "12345",
  "filename": "cover.jpg",
  "source_type": "local_cover",
  "cover_id": "xxxxx",
  "Content-Type": "multipart/form-data"
}
```

user_cover 上传并带 map_id：

```json
{
  "file_id": "12345",
  "filename": "avatar.jpg",
  "source_type": "user_cover",
  "scale": 80,
  "map_id": "placeholder-001",
  "file": "<binary>",
  "Content-Type": "multipart/form-data"
}
```


#### 参数说明

- `file_id` (string, 必填): 文件 ID 或分享 ID
- `filename` (string, 必填): 附件名
- `url` (string, 二选一必填: `url` / `file`): 附件 URL，与 `file` 二选一
- `file` (byte, 二选一必填: `url` / `file`): 附件二进制流，与 `url` 二选一
- `source_type` (string, 可选): 上传内容类型。可选值：`local_cover` / `user_cover`；默认值：`file`
- `source` (string, 可选): 来源，例如 `processon`
- `cover_id` (string, 可选): 封面 ID；当 `source_type=local_cover` 时必填
- `scale` (integer, 可选): 缩略图压缩比；当 `source_type=user_cover` 时必填
- `map_id` (string, 可选): 占位图标志位（mapId）
- `Content-Type` (string, 必填): Header 文件类型，建议 `multipart/form-data`

#### 返回值说明

```json
{
  "result": "ok",
  "object_id": "1234567890",
  "extra_info": {
    "width": 600,
    "height": 400
  },
  "old_content_type": "image/heic",
  "new_content_type": "image/jpeg"
}

```

| 字段 | 类型 | 说明 |
|------|------|------|
| `result` | string | 结果状态，成功为 `ok` |
| `object_id` | string | 上传对象 ID |
| `url` | string | 上传资源 URL（部分场景返回） |
| `extra_info.width` | integer | 图片宽度 |
| `extra_info.height` | integer | 图片高度 |
| `old_content_type` | string | 原始内容类型 |
| `new_content_type` | string | 转换后内容类型 |

响应中的 `object_id` 可作为上传对象标识用于后续引用。
在特定场景（如带 `map_id`）下，响应可能额外返回 `url`。


---

## 6. sheet.update_range_data

#### 功能说明

批量更新单元格区域数据，支持写入公式/内容、图片、格式和合并。
本工具使用下划线参数命名（如 `worksheet_id`、`range_data`、`op_type`）。



#### 操作约束

- **前置检查**：建议先读取目标区域数据，确认覆盖范围与格式影响
- **前置检查**：使用该工具前必须先调用get_sheets_info确认要操作的工作表id，不得自行捏造工作表id。
- **提示**：每项操作都必须提供 row_from/row_to/col_from/col_to 与 op_type

**幂等性**：是

#### 调用示例

写入公式：

```json
{
  "file_id": "string",
  "worksheet_id": 3,
  "range_data": [
    {
      "op_type": "cell_operation_type_formula",
      "row_from": 0,
      "row_to": 0,
      "col_from": 0,
      "col_to": 0,
      "formula": "=A1+B1"
    }
  ]
}
```

写入图片：

```json
{
  "file_id": "string",
  "worksheet_id": 3,
  "range_data": [
    {
      "op_type": "cell_operation_type_picture",
      "row_from": 1,
      "row_to": 1,
      "col_from": 1,
      "col_to": 1,
      "cell_pic_info": {
        "tag": "sheet_pic_type_url",
        "pic_content": "https://example.com/image.png",
        "width": 200,
        "height": 120
      }
    }
  ]
}
```


#### 参数说明

- `file_id` (string, 必填): 文件 ID
- `worksheet_id` (integer, 必填): 工作表 ID
- `range_data` (array[object], 必填): 单元格操作数组

**枚举值：**

- `op_type`：
  - `cell_operation_type_formula`
  - `cell_operation_type_picture`
  - `cell_operation_type_format`
  - `cell_operation_type_merge`
- `merge_type`：
  - `merge_type_center`
  - `merge_type_content`
  - `merge_type_same`
  - `merge_type_columns`
- `cell_pic_info.tag`：
  - `sheet_pic_type_local`
  - `sheet_pic_type_attachment`
  - `sheet_pic_type_url`

**说明：**

- `formula` 仅在 `op_type = cell_operation_type_formula` 时使用。
- `merge_type` 仅在 `op_type = cell_operation_type_merge` 时使用。
- `xf` 仅在 `op_type = cell_operation_type_format` 时使用。
- `cell_pic_info` 仅在 `op_type = cell_operation_type_picture` 时使用。


#### 返回值说明

```json
{
  "code": 0,
  "msg": "string"
}

```

| 字段 | 类型 | 说明 |
|------|------|------|
| `code` | integer | 响应码 |
| `msg` | string | 响应消息 |


---

