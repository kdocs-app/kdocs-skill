# 形状 · 列举与基础

> facet: shape.items

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `wpp.shape.items.delete` | 删除指定形状（需 slide_id 与 shape_id） | `slide_id` |
| `wpp.shape.items.duplicate` | 设置形状：就地复制 | `slide_id` |
| `wpp.shape.items.item` | 查询形状：单条形状详情（需 slide_id 与 shape_id | `slide_id` |
| `wpp.shape.items.list` | 查询形状：形状列表 | `slide_id` |
| `wpp.shape.items.z_order` | 设置形状：叠放顺序 | `slide_id` |

---

## 1. wpp.shape.items.delete

#### 功能说明

删除指定形状（需 slide_id 与 shape_id）。

**幂等性**：是 — safe

#### 调用示例

删除形状：

```json
{
  "file_id": "<FILE_ID>",
  "slide_id": 150995203,
  "shape_id": 3
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
    "deleted_shape_id": 3
  }
}
```


---

## 2. wpp.shape.items.duplicate

#### 功能说明

设置形状：就地复制。

**幂等性**：是 — safe

#### 调用示例

就地复制形状：

```json
{
  "file_id": "<FILE_ID>",
  "slide_id": 150995203,
  "shape_id": 3
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
    "new_shape_id": 1,
    "shape_id": 3
  }
}
```


---

## 3. wpp.shape.items.item

#### 功能说明

查询形状：单条形状详情（需 slide_id 与 shape_id；返回位置、填充、线条等属性）。

**幂等性**：是 — safe

#### 调用示例

查询形状详情：

```json
{
  "file_id": "<FILE_ID>",
  "slide_id": 150995203,
  "shape_id": 2
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
    "item_data": {
      "fill": {
        "fore_color": 0,
        "transparency": 1,
        "type": -2
      },
      "line": {
        "fore_color": 16777215,
        "visible": false,
        "weight": -2147483904
      },
      "name": "Title 1",
      "position": {
        "left": 36,
        "top": 21.62504005432129
      },
      "shape_id": 2,
      "size": {
        "height": 90,
        "width": 648
      },
      "type": 14
    }
  }
}
```


---

## 4. wpp.shape.items.list

#### 功能说明

查询形状：形状列表。

**幂等性**：是 — safe

#### 调用示例

查询幻灯片形状列表：

```json
{
  "file_id": "<FILE_ID>",
  "slide_id": 150995203
}
```

复核插入后的形状列表：

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
    "list_data": {
      "count": 4,
      "shapes": [
        {
          "height": 90,
          "left": 36,
          "name": "Title 1",
          "shape_id": 2,
          "top": 21.62504005432129,
          "type": 14,
          "width": 648
        },
        {
          "height": 43.20000076293945,
          "left": 36,
          "name": "副标题 16",
          "shape_id": 3,
          "top": 230.39999389648438,
          "type": 17,
          "width": 576
        },
        {
          "height": 8.640000343322754,
          "left": 36,
          "name": "装饰条 17",
          "shape_id": 4,
          "top": 180,
          "type": 1,
          "width": 72
        },
        {
          "height": 28.75,
          "left": 36,
          "name": "日期占位符 4",
          "shape_id": 5,
          "top": 500.5,
          "type": 14,
          "width": 168
        }
      ]
    }
  }
}
```


---

## 5. wpp.shape.items.z_order

#### 功能说明

设置形状：叠放顺序。

**幂等性**：是 — safe

#### 调用示例

设置形状叠放顺序：

```json
{
  "file_id": "<FILE_ID>",
  "slide_id": 150995203,
  "shape_id": 2,
  "z_order": "BRING_TO_FRONT"
}
```

#### 参数说明

- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `shape_id` (number, 可选): 形状 ID
- `slide_id` (number, 必填): 幻灯片 ID
- `url` (string, 可选): 文档 URL；与 link_id、file_id 三选一
- `z_order` (string, 可选): 叠放顺序：BRING_TO_FRONT 等

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "data": {
      "shape_id": 2
    }
  }
}
```
