# 形状 · 插入

> facet: shape.items

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `wpp.shape.items.line` | 设置形状：描边（需 slide_id、shape_id | `slide_id` |
| `wpp.shape.items.lines` | 插入在指定幻灯片批量插入线条（批量 lines[]；需 slide_id） | `lines`, `slide_id` |
| `wpp.shape.items.picture` | 插入在指定幻灯片插入图片形状（单条平铺：image_url + 位置尺寸 | `image_url`, `slide_id` |
| `wpp.shape.items.tables` | 插入在指定幻灯片批量插入表格（批量 tables[]；需 slide_id） | `slide_id`, `tables` |
| `wpp.shape.items.textboxes` | 插入在指定幻灯片批量插入文本框（批量 textboxes[] | `slide_id`, `textboxes` |

---

## 1. wpp.shape.items.line

#### 功能说明

设置形状：描边（需 slide_id、shape_id；可设 line_color/line_weight）。

**幂等性**：是 — safe

#### 调用示例

设置形状描边：

```json
{
  "file_id": "<FILE_ID>",
  "slide_id": 150995203,
  "shape_id": 2,
  "line_color": 16711680,
  "line_weight": 2
}
```

#### 参数说明

- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `line_color` (number, 可选): 线条颜色
- `line_weight` (number, 可选): 线宽
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
        "line"
      ]
    }
  }
}
```


---

## 2. wpp.shape.items.lines

#### 功能说明

插入在指定幻灯片批量插入线条（批量 lines[]；需 slide_id）。

**幂等性**：是 — safe

#### 调用示例

批量插入线条：

```json
{
  "file_id": "<FILE_ID>",
  "slide_id": 150995203,
  "lines": [
    {
      "begin_x": 0,
      "begin_y": 0,
      "end_x": 100,
      "end_y": 100
    }
  ]
}
```

#### 参数说明

- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `lines` (array, 必填): 待插入的线条列表（必填）；每项可设 begin_x/y、end_x/y
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `slide_id` (number, 必填): 幻灯片 ID
- `url` (string, 可选): 文档 URL；与 link_id、file_id 三选一

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "data": {
      "count": 1,
      "shape_ids": [
        13
      ]
    }
  }
}
```


---

## 3. wpp.shape.items.picture

#### 功能说明

插入在指定幻灯片插入图片形状（单条平铺：image_url + 位置尺寸；需 slide_id）。

**幂等性**：是 — safe

#### 调用示例

插入图片形状（白名单域名）：

```json
{
  "file_id": "<FILE_ID>",
  "slide_id": 150995203,
  "image_url": "https://qn.cache.wpscdn.cn/kdocs/mobile/touch/apple-152.png",
  "left": 300,
  "top": 100,
  "width": 200,
  "height": 120
}
```

#### 参数说明

- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `height` (number, 可选): 高度
- `image_url` (string, 必填): 图片 URL（必填）
- `left` (number, 可选): 左边距
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
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
      "shape_id": 15
    }
  }
}
```


---

## 4. wpp.shape.items.tables

#### 功能说明

插入在指定幻灯片批量插入表格（批量 tables[]；需 slide_id）。

**幂等性**：是 — safe

#### 调用示例

批量插入表格：

```json
{
  "file_id": "<FILE_ID>",
  "slide_id": 150995203,
  "tables": [
    {
      "rows": 2,
      "columns": 3,
      "left": 400,
      "top": 200
    }
  ]
}
```

#### 参数说明

- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `slide_id` (number, 必填): 幻灯片 ID
- `tables` (array, 必填): 待插入的表格列表（必填）；每项可设 rows、columns、left、top
- `url` (string, 可选): 文档 URL；与 link_id、file_id 三选一

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "data": {
      "count": 1,
      "shape_ids": [
        14
      ]
    }
  }
}
```


---

## 5. wpp.shape.items.textboxes

#### 功能说明

插入在指定幻灯片批量插入文本框（批量 textboxes[]；需 slide_id）。

**幂等性**：是 — safe

#### 调用示例

批量插入文本框：

```json
{
  "file_id": "<FILE_ID>",
  "slide_id": 150995203,
  "textboxes": [
    {
      "text": "批次B文本框",
      "left": 100,
      "top": 400,
      "width": 300,
      "height": 60
    }
  ]
}
```

#### 参数说明

- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `slide_id` (number, 必填): 幻灯片 ID
- `textboxes` (array, 必填): 待插入的文本框列表（必填）；每项可设 orientation、位置尺寸、text
- `url` (string, 可选): 文档 URL；与 link_id、file_id 三选一

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "data": {
      "count": 1,
      "shape_ids": [
        11
      ]
    }
  }
}
```
