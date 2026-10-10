# 形状 · 组合与几何

> facet: shape.items

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `wpp.shape.items.fill` | 设置形状：填充（需 slide_id、shape_id | `slide_id` |
| `wpp.shape.items.geometry` | 设置形状：位置与尺寸（需 slide_id 与 shape_id | `slide_id` |
| `wpp.shape.items.group` | 设置形状：组合（组合或取消组合选中形状） | `slide_id` |
| `wpp.shape.items.group_children` | 查询形状：组合子形状（需 slide_id 与 shape_id | `slide_id` |
| `wpp.shape.items.name` | 设置形状：名称 | `name`, `slide_id` |
| `wpp.shape.items.reorder_in_group` | 设置形状：组内排序 | `child_shape_ids`, `slide_id` |
| `wpp.shape.items.shapes` | 插入在指定幻灯片批量插入选自图形（批量 shapes[]（auto_shape_ | `shapes`, `slide_id` |
| `wpp.shape.items.ungroup` | 设置形状：取消组合 | `slide_id` |
| `wpp.shape.items.update_group_props` | 设置形状：组合属性 | `props`, `slide_id` |

---

## 1. wpp.shape.items.fill

#### 功能说明

设置形状：填充（需 slide_id、shape_id；可设 fill_color）。

**幂等性**：是 — safe

#### 调用示例

设置形状填充：

```json
{
  "file_id": "<FILE_ID>",
  "slide_id": 150995203,
  "shape_id": 2,
  "fill_color": "FF0000"
}
```

#### 参数说明

- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `fill` (object, 可选): 填充对象
- `fill_color` (string, 可选): 填充颜色
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
      "shape_id": 3,
      "updated_properties": [
        "fill"
      ]
    }
  }
}
```


---

## 2. wpp.shape.items.geometry

#### 功能说明

设置形状：位置与尺寸（需 slide_id 与 shape_id；可设 left/top/width/height）。

**幂等性**：是 — safe

#### 调用示例

设置形状位置尺寸：

```json
{
  "file_id": "<FILE_ID>",
  "slide_id": 150995203,
  "shape_id": 2,
  "left": 120,
  "top": 100,
  "width": 600,
  "height": 150
}
```

#### 参数说明

- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `height` (number, 可选): 高度
- `left` (number, 可选): 左边距
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `shape_id` (number, 可选): 形状 ID
- `slide_id` (number, 必填): 幻灯片 ID
- `top` (number, 可选): 上边距
- `url` (string, 可选): 文档 URL；与 link_id、file_id 三选一
- `width` (number, 可选): 宽度

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "data": {
      "shape_id": 2,
      "updated_properties": [
        "left",
        "top",
        "width",
        "height"
      ]
    }
  }
}
```


---

## 3. wpp.shape.items.group

#### 功能说明

设置形状：组合（组合或取消组合选中形状）。

**幂等性**：是 — safe

#### 参数说明

- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `shape_id` (number, 可选): 形状 ID（与 shape_ids 二选一，单形状时用于 target）
- `slide_id` (number, 必填): 幻灯片 ID
- `url` (string, 可选): 文档 URL；与 link_id、file_id 三选一

#### 返回值说明

```json
{
  "new_shape_id": {
    "type": "integer"
  },
  "shape_id": {
    "type": "integer"
  },
  "updated_properties": {
    "type": "array",
    "items": {
      "type": "string"
    }
  }
}
```


---

## 4. wpp.shape.items.group_children

#### 功能说明

查询形状：组合子形状（需 slide_id 与 shape_id；返回组合内子形状列表）。

**幂等性**：是 — safe

#### 参数说明

- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `shape_id` (number, 可选): 形状 ID
- `slide_id` (number, 必填): 幻灯片 ID
- `url` (string, 可选): 文档 URL；与 link_id、file_id 三选一

#### 返回值说明

```json
{
  "children": {
    "type": "array",
    "items": {
      "type": "object",
      "properties": {
        "height": {
          "type": "number"
        },
        "left": {
          "type": "number"
        },
        "name": {
          "type": "string"
        },
        "shape_id": {
          "type": "integer"
        },
        "top": {
          "type": "number"
        },
        "type": {
          "type": "integer"
        },
        "width": {
          "type": "number"
        },
        "z_order_position": {
          "type": "integer"
        }
      }
    }
  },
  "count": {
    "type": "integer"
  },
  "shape_id": {
    "type": "integer"
  }
}
```


---

## 5. wpp.shape.items.name

#### 功能说明

设置形状：名称。

**幂等性**：是 — safe

#### 调用示例

设置形状名称：

```json
{
  "file_id": "<FILE_ID>",
  "slide_id": 150995203,
  "shape_id": 2,
  "name": "实测标题"
}
```

#### 参数说明

- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `name` (string, 必填): 名称
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
      "shape_id": 2,
      "updated_properties": [
        "name"
      ]
    }
  }
}
```


---

## 6. wpp.shape.items.reorder_in_group

#### 功能说明

设置形状：组内排序。

**幂等性**：是 — safe

#### 参数说明

- `child_shape_ids` (array, 必填): 组内子形状 ID 的有序列表
- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `shape_id` (number, 可选): 形状 ID
- `slide_id` (number, 必填): 幻灯片 ID
- `url` (string, 可选): 文档 URL；与 link_id、file_id 三选一

#### 返回值说明

```json
{
  "new_shape_id": {
    "type": "integer"
  },
  "shape_id": {
    "type": "integer"
  },
  "updated_properties": {
    "type": "array",
    "items": {
      "type": "string"
    }
  }
}
```


---

## 7. wpp.shape.items.shapes

#### 功能说明

插入在指定幻灯片批量插入选自图形（批量 shapes[]（auto_shape_type + 位置尺寸）；需 slide_id）。

**幂等性**：是 — safe

#### 调用示例

批量插入自选图形：

```json
{
  "file_id": "<FILE_ID>",
  "slide_id": 150995203,
  "shapes": [
    {
      "auto_shape_type": 1,
      "left": 500,
      "top": 400,
      "width": 120,
      "height": 80
    }
  ]
}
```

#### 参数说明

- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `shapes` (array, 必填): 待插入的 AutoShape 列表（必填）；每项需 auto_shape_type、left、top、width、height
- `slide_id` (number, 必填): 幻灯片 ID
- `url` (string, 可选): 文档 URL；与 link_id、file_id 三选一

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "count": 1,
    "shape_ids": [
      12
    ]
  }
}
```


---

## 8. wpp.shape.items.ungroup

#### 功能说明

设置形状：取消组合。

**幂等性**：是 — safe

#### 参数说明

- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `shape_id` (number, 可选): 形状 ID
- `slide_id` (number, 必填): 幻灯片 ID
- `url` (string, 可选): 文档 URL；与 link_id、file_id 三选一

#### 返回值说明

```json
{
  "new_shape_id": {
    "type": "integer"
  },
  "shape_id": {
    "type": "integer"
  },
  "updated_properties": {
    "type": "array",
    "items": {
      "type": "string"
    }
  }
}
```


---

## 9. wpp.shape.items.update_group_props

#### 功能说明

设置形状：组合属性。

**幂等性**：是 — safe

#### 调用示例

设置组合属性：

```json
{
  "file_id": "<FILE_ID>",
  "slide_id": 150995203,
  "shape_id": 2,
  "props": {
    "name": "组合A",
    "visible": true
  }
}
```

设置组合属性（props 缺省仍 code=0）：

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
- `props` (object, 必填): 组合属性对象，如 name、visible
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
      "shape_id": 2
    }
  }
}
```
