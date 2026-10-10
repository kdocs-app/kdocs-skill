# 幻灯片 · 单页属性

> facet: slide.items

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `wpp.slide.items.background` | 查询幻灯片：背景（含填充类型与前景色；需 slide_id） | `slide_id` |
| `wpp.slide.items.datetime` | 设置幻灯片：datetime | `slide_id` |
| `wpp.slide.items.footer` | 设置幻灯片：footer | `slide_id` |
| `wpp.slide.items.layout` | 幻灯片：版式：支持 查询/设置（需 slide_id 与 layout 版式索引 | `layout`, `slide_id`, `verb` |
| `wpp.slide.items.master` | 查询幻灯片：关联母版 | `slide_id` |
| `wpp.slide.items.name` | 幻灯片：名称：支持 查询/设置（需 slide_id） | `name`, `slide_id`, `verb` |
| `wpp.slide.items.page_number` | 设置幻灯片：page number | `slide_id` |
| `wpp.slide.items.slide_id` | 查询幻灯片：幻灯片 ID | `slide_id` |
| `wpp.slide.items.transition` | 查询幻灯片：切换动画 | `slide_id` |

---

## 1. wpp.slide.items.background

#### 功能说明

查询幻灯片：背景（含填充类型与前景色；需 slide_id）。

**幂等性**：是 — safe

#### 调用示例

查询幻灯片背景：

```json
{
  "file_id": "<FILE_ID>",
  "slide_id": 150995203
}
```

#### 参数说明

- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `slide_id` (number, 必填): 幻灯片 ID
- `url` (string, 可选): 文档 URL；与 link_id、file_id 三选一

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "item_data": {
      "background": {}
    }
  }
}
```


---

## 2. wpp.slide.items.datetime

#### 功能说明

设置幻灯片：datetime。

**幂等性**：是 — safe

#### 调用示例

设置幻灯片页眉页脚时间：

```json
{
  "file_id": "<FILE_ID>",
  "slide_id": 150995203,
  "hf": {
    "visible": true,
    "text": "2026-09-17",
    "datetime_format": "yyyy-MM-dd"
  }
}
```

#### 参数说明

- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `hf` (object, 可选): 页眉页脚配置：visible、text、datetime_format、display_on_title_slide
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `slide_id` (number, 必填): 幻灯片 ID
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

## 3. wpp.slide.items.footer

#### 功能说明

设置幻灯片：footer。

**幂等性**：是 — safe

#### 调用示例

设置幻灯片页脚文字：

```json
{
  "file_id": "<FILE_ID>",
  "slide_id": 150995203,
  "hf": {
    "visible": true,
    "text": "WPP-BENCH-FOOTER"
  }
}
```

#### 参数说明

- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `hf` (object, 可选): 页眉页脚配置：visible、text、datetime_format、display_on_title_slide
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `slide_id` (number, 必填): 幻灯片 ID
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

## 4. wpp.slide.items.layout

#### 功能说明

幻灯片：版式：支持 查询/设置（需 slide_id 与 layout 版式索引）。

**幂等性**：是 — safe

#### 调用示例

查询幻灯片版式：

```json
{
  "file_id": "<FILE_ID>",
  "slide_id": 150995203,
  "verb": "query"
}
```

设置幻灯片版式：

```json
{
  "file_id": "<FILE_ID>",
  "slide_id": 150995204,
  "verb": "update",
  "layout": 2
}
```

#### 参数说明

- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `layout` (number, 必填): 版式索引
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `slide_id` (number, 必填): 幻灯片 ID
- `url` (string, 可选): 文档 URL；与 link_id、file_id 三选一
- `verb` (string, 必填): HTTP 动作：query（查询） / update（设置）。可选值：`query` / `update`

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "item_data": {
      "layout": {
        "index": 1,
        "name": "标题幻灯片"
      }
    }
  }
}
```


---

## 5. wpp.slide.items.master

#### 功能说明

查询幻灯片：关联母版。

**幂等性**：是 — safe

#### 调用示例

查询幻灯片关联母版：

```json
{
  "file_id": "<FILE_ID>",
  "slide_id": 150995203
}
```

#### 参数说明

- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `slide_id` (number, 必填): 幻灯片 ID
- `url` (string, 可选): 文档 URL；与 link_id、file_id 三选一

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "item_data": {
      "master": {
        "design": "Office Theme"
      }
    }
  }
}
```


---

## 6. wpp.slide.items.name

#### 功能说明

幻灯片：名称：支持 查询/设置（需 slide_id）。

**幂等性**：是 — safe

#### 调用示例

查询幻灯片名称：

```json
{
  "file_id": "<FILE_ID>",
  "slide_id": 150995203,
  "verb": "query"
}
```

设置幻灯片名称：

```json
{
  "file_id": "<FILE_ID>",
  "slide_id": 150995203,
  "verb": "update",
  "name": "样板页A"
}
```

#### 参数说明

- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `name` (string, 必填): 名称
- `slide_id` (number, 必填): 幻灯片 ID
- `url` (string, 可选): 文档 URL；与 link_id、file_id 三选一
- `verb` (string, 必填): HTTP 动作：query（查询） / update（设置）。可选值：`query` / `update`

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "item_data": {}
  }
}
```


---

## 7. wpp.slide.items.page_number

#### 功能说明

设置幻灯片：page number。

**幂等性**：是 — safe

#### 调用示例

设置幻灯片页码：

```json
{
  "file_id": "<FILE_ID>",
  "slide_id": 150995203,
  "hf": {
    "visible": true
  }
}
```

#### 参数说明

- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `hf` (object, 可选): 页眉页脚配置：visible、text、datetime_format、display_on_title_slide
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `slide_id` (number, 必填): 幻灯片 ID
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

## 8. wpp.slide.items.slide_id

#### 功能说明

查询幻灯片：幻灯片 ID。

**幂等性**：是 — safe

#### 调用示例

查询幻灯片 ID：

```json
{
  "file_id": "<FILE_ID>",
  "slide_id": 150995203
}
```

#### 参数说明

- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `slide_id` (number, 必填): 幻灯片 ID
- `url` (string, 可选): 文档 URL；与 link_id、file_id 三选一

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "item_data": {
      "slide_id": 150995203
    }
  }
}
```


---

## 9. wpp.slide.items.transition

#### 功能说明

查询幻灯片：切换动画。

**幂等性**：是 — safe

#### 调用示例

查询幻灯片切换动画：

```json
{
  "file_id": "<FILE_ID>",
  "slide_id": 150995203
}
```

#### 参数说明

- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `slide_id` (number, 必填): 幻灯片 ID
- `url` (string, 可选): 文档 URL；与 link_id、file_id 三选一

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "item_data": {
      "transition": {
        "advance_on_click": true,
        "duration": 0.5
      }
    }
  }
}
```
