# 更新智能文档
调用`otl.block_update`工具，传入智能文档的 `file_id` 和相关参数，可更新指定块，适合局部更新或者更新Markdown数据不支持的内容。

当前支持以下操作：
1. 更新块的内容
2. 更新块的属性
3. 插入表格行
4. 插入表格列
5. 删除表格行
6. 删除表格列
7. 合并单元格
8. 拆分单元格

---

## 参数说明

- `file_id` (string, 必填): 智能文档文件 ID
- `params` (array, 必填): 操作数组，各类型操作参数参考下方

---

## 各类型操作参数

### 更新块的内容

- `operation` (string, 必填): 固定为"update_content"
- `blockId` (string, 必填): 块ID，为"doc"表示更新全文内容，其余子块ID可通过查询文档块获取。
- `content` (array, 必填)： 块的内容，其中可填入的节点类型和属性参考`references/otl/node.md`
- **当 blockId 为 "doc" 时，content 的第一个子节点必须是 type: title**，否则接口报错

#### 调用示例

覆盖文档的标题和正文

```json
{
  "params": [
    {
      "operation": "update_content",
      "blockId": "doc",
      "content": [
        {
          "type": "title",
          "content": [
            {
              "type": "text",
              "content": "文档的标题"
            }
          ]
        },
        {
          "type": "paragraph",
          "content": [
            {
              "type": "text",
              "content": "文档的正文"
            }
          ]
        }
      ]
    }
  ]
}
```

更新段落的文本内容

```json
{
  "params": [
    {
      "operation": "update_content",
      "blockId": "PARA_ID",
      "content": [
        {
          "type": "text",
          "content": "更新的文本内容"
        }
      ]
    }
  ]
}
```

#### 注意事项
1. 块支持的子节点，参考`references/otl/node.md`
2. 当blockId为"doc"时，content的第一个子节点必须是type: title，否则接口报错

### 更新块属性

- `blockId` (string, 必填): 块ID，可通过查询文档块获取。
- `operation` (string, 必填): 固定为"update_attrs"
- `attrs` (object, 必填)： 块的属性，属性参考`references/otl/node.md`，当前不支持设置doc、appComponent、lockBlock这三种块的属性，tableCell的colSpan/rowSpan属性请通过下方表格相关操作设置。

#### 调用示例

更新段落的背景颜色

```json
{
  "params": [
    {
      "operation": "update_attrs",
      "blockId": "PARA_ID",
      "attrs": {
        "color": {
          "backgroundColor": "#FBF5B3"
        }
      }
    }
  ]
}
```

#### 注意事项
1. 更新属性是覆盖操作，不需要更新的属性保持原样传入
2. 当前不支持设置doc、appComponent、lockBlock这三种块的属性
3. 本操作不支持设置tableCell的rowSpan/colSpan属性

### 插入表格行

- `blockId` (string, 必填): 块ID，可通过查询文档块获取，对应块必须是table。
- `operation` (string, 必填): 固定为"insert_table_rows"
- `content` (array, 必填)： tableRow数组，需与表格列数对齐。tableRow节点参考`references/otl/node.md`
- `start` (integer, 非必填)：插入的行索引，默认0

#### 调用示例

```json
{
  "params": [
    {
      "operation": "insert_table_rows",
      "blockId": "TABLE_ID",
      "content": [
        {
          "type": "tableRow",
          "content": [
            {
              "type": "tableCell",
              "content": [
                {
                  "type": "paragraph",
                  "content": [
                    {
                      "type": "text",
                      "content": "1111"
                    }
                  ]
                }
              ]
            },
            {
              "type": "tableCell"
            }
          ]
        },
        {
          "type": "tableRow",
          "content": [
            {
              "type": "tableCell"
            },
            {
              "type": "tableCell"
            }
          ]
        }
      ]
    }
  ]
}
```

#### 注意事项
1. 表格行里的单元格数量，需与表格列数对齐。

### 插入表格列

- `blockId` (string, 必填): 块ID，可通过查询文档块获取，对应块必须是table。
- `operation` (string, 必填): 固定为"insert_table_columns"
- `content` (array, 必填)： tableRow数组，需与表格行数对齐。tableRow节点参考`references/otl/node.md`
- `start` (integer, 非必填)：插入的列索引，默认0

#### 调用示例

```json
{
  "params": [
    {
      "operation": "insert_table_columns",
      "blockId": "TABLE_ID",
      "content": [
        {
          "type": "tableRow",
          "content": [
            {
              "type": "tableCell",
              "content": [
                {
                  "type": "paragraph",
                  "content": [
                    {
                      "type": "text",
                      "content": "1111"
                    }
                  ]
                }
              ]
            },
            {
              "type": "tableCell"
            }
          ]
        },
        {
          "type": "tableRow",
          "content": [
            {
              "type": "tableCell"
            },
            {
              "type": "tableCell"
            }
          ]
        }
      ]
    }
  ]
}
```

#### 注意事项
1. content字段里的表格行数量，要与表格行数对齐。每个表格行的单元格数量也需要一致。

### 删除表格行

- `blockId` (string, 必填): 块ID，可通过查询文档块获取，对应块必须是table。
- `operation` (string, 必填): 固定为"delete_table_rows"
- `count` (integer, 必填)： 删除行数，至少为1
- `start` (integer, 非必填)：删除起始索引，默认0

#### 调用示例

```json
{
  "params": [
    {
      "operation": "delete_table_rows",
      "blockId": "TABLE_ID",
      "count": 2,
      "start": 2
    }
  ]
}
```

### 删除表格列

- `blockId` (string, 必填): 块ID，可通过查询文档块获取，对应块必须是table。
- `operation` (string, 必填): 固定为"delete_table_columns"
- `count` (integer, 必填)： 删除行数，至少为1
- `start` (integer, 非必填)：删除起始索引，默认0

#### 调用示例

```json
{
  "params": [
    {
      "operation": "delete_table_columns",
      "blockId": "TABLE_ID",
      "count": 2,
      "start": 2
    }
  ]
}
```

### 合并单元格

- `blockId` (string, 必填): 块ID，可通过查询文档块获取，对应块必须是table。
- `operation` (string, 必填): 固定为"merge_table_cells"
- `rowSpan` (integer, 必填)： 合并行数，至少为1，与colSpan不可同时为1
- `colSpan` (integer, 必填)： 合并列数，至少为1，与rowSpan不可同时为1
- `startRow` (integer, 非必填)：合并的起始行号，默认0
- `startCol` (integer, 非必填)：合并的起始列号，默认0

#### 调用示例

```json
{
  "params": [
    {
      "operation": "merge_table_cells",
      "blockId": "TABLE_ID",
      "rowSpan": 2,
      "colSpan": 3,
      "startRow": 1,
      "startCol": 1
    }
  ]
}
```

### 拆分单元格

- `blockId` (string, 必填): 块ID，可通过查询文档块获取，对应块必须是table。拆分对应的单元格必须是合并单元格
- `operation` (string, 必填): 固定为"split_table_cell"
- `startRow` (integer, 非必填)：拆分单元格的行号，默认0
- `startCol` (integer, 非必填)：拆分单元格的列号，默认0

```json
{
  "params": [
    {
      "operation": "split_table_cell",
      "blockId": "TABLE_ID",
      "startRow": 1,
      "startCol": 1
    }
  ]
}
```
