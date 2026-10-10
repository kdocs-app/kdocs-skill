# 形状 · 表格

> facet: shape.tables

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `wpp.shape.tables.cell_merge` | 设置形状表格合并单元格 | `slide_id` |
| `wpp.shape.tables.cell_text` | 形状表格单元格文本：支持 查询/设置 | `slide_id`, `verb` |
| `wpp.shape.tables.cell_unmerge` | 设置形状表格取消单元格合并 | `slide_id` |
| `wpp.shape.tables.cells` | 查询形状表格单元格列表 | `slide_id` |
| `wpp.shape.tables.column` | 设置形状表格表格列 | `slide_id` |
| `wpp.shape.tables.dimensions` | 查询形状表格行列数 | `slide_id` |
| `wpp.shape.tables.merged_ranges` | 查询形状表格合并单元格区间 | `slide_id` |
| `wpp.shape.tables.row` | 设置形状表格表格行 | `slide_id` |

---

## 1. wpp.shape.tables.cell_merge

#### 功能说明

设置形状表格合并单元格。

**幂等性**：是 — safe

#### 调用示例

合并表格单元格：

```json
{
  "file_id": "<FILE_ID>",
  "slide_id": 150995203,
  "start_row": 1,
  "start_col": 1,
  "end_row": 1,
  "end_col": 2
}
```

#### 参数说明

- `before_column` (number, 可选): 插入位置前的列号（省略则追加）
- `before_row` (number, 可选): 插入位置前的行号（省略则追加）
- `column` (number, 可选): 列号（1 起）
- `column_index` (number, 可选): 起始列号（1 起）
- `count` (number, 可选): 数量，默认 1
- `end_col` (number, 可选): 结束列（闭区间）
- `end_row` (number, 可选): 结束行（闭区间）
- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `row` (number, 可选): 行号（1 起）
- `row_index` (number, 可选): 起始行号（1 起）
- `shape_id` (number, 可选): 形状 ID
- `slide_id` (number, 必填): 幻灯片 ID
- `start_col` (number, 可选): 起始列（闭区间）
- `start_row` (number, 可选): 起始行（闭区间）
- `text` (string, 可选): 文本内容
- `url` (string, 可选): 文档 URL；与 link_id、file_id 三选一

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {}
}
```


---

## 2. wpp.shape.tables.cell_text

#### 功能说明

形状表格单元格文本：支持 查询/设置。

**幂等性**：是 — safe

#### 调用示例

查询表格单元格文本：

```json
{
  "file_id": "<FILE_ID>",
  "slide_id": 150995203,
  "verb": "query"
}
```

设置表格单元格文本：

```json
{
  "file_id": "<FILE_ID>",
  "slide_id": 150995203,
  "row": 1,
  "column": 1,
  "text": "销售统计",
  "verb": "update",
  "shape_id": 4
}
```

#### 参数说明

- `before_column` (number, 可选): 插入位置前的列号（省略则追加）
- `before_row` (number, 可选): 插入位置前的行号（省略则追加）
- `column` (number, 可选): 列号（1 起）
- `column_index` (number, 可选): 起始列号（1 起）
- `count` (number, 可选): 数量，默认 1
- `end_col` (number, 可选): 结束列（闭区间）
- `end_row` (number, 可选): 结束行（闭区间）
- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `row` (number, 可选): 行号（1 起）
- `row_index` (number, 可选): 起始行号（1 起）
- `shape_id` (number, 可选): 形状 ID
- `slide_id` (number, 必填): 幻灯片 ID
- `start_col` (number, 可选): 起始列（闭区间）
- `start_row` (number, 可选): 起始行（闭区间）
- `text` (string, 可选): 文本内容
- `url` (string, 可选): 文档 URL；与 link_id、file_id 三选一
- `verb` (string, 必填): HTTP 动作：query（查询） / update（设置）。可选值：`query` / `update`

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "data": {
      "cells": [
        {
          "column": 1,
          "row": 1,
          "text": "销售统计"
        }
      ],
      "dimensions": {},
      "shape_id": 8
    }
  }
}
```


---

## 3. wpp.shape.tables.cell_unmerge

#### 功能说明

设置形状表格取消单元格合并。

**幂等性**：是 — safe

#### 参数说明

- `before_column` (number, 可选): 插入位置前的列号（省略则追加）
- `before_row` (number, 可选): 插入位置前的行号（省略则追加）
- `column` (number, 可选): 列号（1 起）
- `column_index` (number, 可选): 起始列号（1 起）
- `count` (number, 可选): 数量，默认 1
- `end_col` (number, 可选): 结束列（闭区间）
- `end_row` (number, 可选): 结束行（闭区间）
- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `row` (number, 可选): 行号（1 起）
- `row_index` (number, 可选): 起始行号（1 起）
- `shape_id` (number, 可选): 形状 ID
- `slide_id` (number, 必填): 幻灯片 ID
- `start_col` (number, 可选): 起始列（闭区间）
- `start_row` (number, 可选): 起始行（闭区间）
- `text` (string, 可选): 文本内容
- `url` (string, 可选): 文档 URL；与 link_id、file_id 三选一

#### 返回值说明

```json
{}
```


---

## 4. wpp.shape.tables.cells

#### 功能说明

查询形状表格单元格列表。

**幂等性**：是 — safe

#### 调用示例

查询表格单元格列表：

```json
{
  "file_id": "<FILE_ID>",
  "slide_id": 150995203
}
```

#### 参数说明

- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `shape_id` (number, 可选): 形状 ID
- `slide_id` (number, 必填): 幻灯片 ID
- `url` (string, 可选): 文档 URL；与 link_id、file_id 三选一

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "data": {
      "cells": [
        {
          "column": 1,
          "row": 1
        },
        {
          "column": 1,
          "row": 2
        }
      ],
      "dimensions": {},
      "shape_id": 8
    }
  }
}
```


---

## 5. wpp.shape.tables.column

#### 功能说明

设置形状表格表格列。

**幂等性**：是 — safe

#### 调用示例

设置表格列宽：

```json
{
  "file_id": "<FILE_ID>",
  "slide_id": 150995203,
  "verb": "update",
  "shape_id": "<SHAPE_ID>",
  "column_index": 1,
  "count": 1
}
```

#### 参数说明

- `before_column` (number, 可选): 插入位置前的列号（省略则追加）
- `before_row` (number, 可选): 插入位置前的行号（省略则追加）
- `column` (number, 可选): 列号（1 起）
- `column_index` (number, 可选): 起始列号（1 起）
- `count` (number, 可选): 数量，默认 1
- `end_col` (number, 可选): 结束列（闭区间）
- `end_row` (number, 可选): 结束行（闭区间）
- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `row` (number, 可选): 行号（1 起）
- `row_index` (number, 可选): 起始行号（1 起）
- `shape_id` (number, 可选): 形状 ID
- `slide_id` (number, 必填): 幻灯片 ID
- `start_col` (number, 可选): 起始列（闭区间）
- `start_row` (number, 可选): 起始行（闭区间）
- `text` (string, 可选): 文本内容
- `url` (string, 可选): 文档 URL；与 link_id、file_id 三选一

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {}
}
```


---

## 6. wpp.shape.tables.dimensions

#### 功能说明

查询形状表格行列数。

**幂等性**：是 — safe

#### 调用示例

查询表格行列数：

```json
{
  "file_id": "<FILE_ID>",
  "slide_id": 150995203
}
```

#### 参数说明

- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `shape_id` (number, 可选): 表格形状 ID（必填，否则 dimensions 为空）
- `slide_id` (number, 必填): 幻灯片 ID
- `url` (string, 可选): 文档 URL；与 link_id、file_id 三选一

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "data": {
      "dimensions": {
        "columns": 3,
        "rows": 4
      },
      "shape_id": 8
    }
  }
}
```


---

## 7. wpp.shape.tables.merged_ranges

#### 功能说明

查询形状表格合并单元格区间。

**幂等性**：是 — safe

#### 调用示例

查询表格合并单元格区间：

```json
{
  "file_id": "<FILE_ID>",
  "slide_id": 150995203
}
```

#### 参数说明

- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `shape_id` (number, 可选): 形状 ID
- `slide_id` (number, 必填): 幻灯片 ID
- `url` (string, 可选): 文档 URL；与 link_id、file_id 三选一

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "data": {
      "dimensions": {},
      "shape_id": 8
    }
  }
}
```


---

## 8. wpp.shape.tables.row

#### 功能说明

设置形状表格表格行。

**幂等性**：是 — safe

#### 调用示例

设置表格行：

```json
{
  "file_id": "<FILE_ID>",
  "slide_id": 150995203,
  "row": 2,
  "count": 1
}
```

#### 参数说明

- `before_column` (number, 可选): 插入位置前的列号（省略则追加）
- `before_row` (number, 可选): 插入位置前的行号（省略则追加）
- `column` (number, 可选): 列号（1 起）
- `column_index` (number, 可选): 起始列号（1 起）
- `count` (number, 可选): 数量，默认 1
- `end_col` (number, 可选): 结束列（闭区间）
- `end_row` (number, 可选): 结束行（闭区间）
- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `row` (number, 可选): 行号（1 起）
- `row_index` (number, 可选): 起始行号（1 起）
- `shape_id` (number, 可选): 形状 ID
- `slide_id` (number, 必填): 幻灯片 ID
- `start_col` (number, 可选): 起始列（闭区间）
- `start_row` (number, 可选): 起始行（闭区间）
- `text` (string, 可选): 文本内容
- `url` (string, 可选): 文档 URL；与 link_id、file_id 三选一

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {}
}
```
