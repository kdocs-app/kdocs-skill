# wps.headings.heading_table

## 1. wps.headings.heading_table

#### 功能说明

按标题章节插入完整表格（TABLE）：表题 + 表头 + 数据行 + 边框。

**幂等性**：否 — unsafe

#### 调用示例

标题章节末尾插入表格：

```json
{
  "file_id": "<FILE_ID>",
  "heading_text": "SmokeHeadingSlice",
  "position": "HEADING_END",
  "rows": 3,
  "cols": 3,
  "table_full_data": {
    "caption": "示例表题",
    "headers": [
      "列1",
      "列2",
      "列3"
    ],
    "rows": [
      {
        "cells": [
          "a",
          "b",
          "c"
        ]
      }
    ]
  }
}
```

#### 参数说明

- `file_id` (string, 三选一必填: `url` / `link_id` / `file_id`): 文件 id；与 url、link_id 三选一
- `link_id` (string, 三选一必填: `url` / `link_id` / `file_id`): 分享 id；与 url、file_id 三选一
- `url` (string, 三选一必填: `url` / `link_id` / `file_id`): 文档 URL；与 link_id、file_id 三选一
- `heading_text` (string, 必填): 锚点标题文本（子串匹配）
- `position` (string, 可选): 插入位置：HEADING_END(章节末尾,缺省) / HEADING_BEGIN(章节开头) / HEADING_BEFORE(锚点标题之前)。可选值：`HEADING_BEFORE` / `HEADING_BEGIN` / `HEADING_END`
- `rows` (number, 必填): 表格行数（>=1）
- `cols` (number, 必填): 表格列数（>=1）
- `table_full_data` (object, 必填): 完整表格数据：{caption?, headers?:[...], rows?:[{cells:[...]}], enable_borders?}

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "heading_index": 42,
    "table_index": 3,
    "table_rows": 3,
    "table_cols": 3
  }
}
```

| 字段 | 类型 | 说明 |
|------|------|------|
| `code` | number | 0 表示成功 |
| `data` | object | 业务数据 |
| `message` | string | 结果说明 |
