# 幻灯片 · 页序与增删

> facet: slide.items

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `wpp.slide.items.batch_slides` | 批量插入多张幻灯片（通过 items[] 一次插入多张幻灯片 | `items` |
| `wpp.slide.items.count` | 查询幻灯片总数（返回演示文稿幻灯片页数） |  |
| `wpp.slide.items.delete` | 删除幻灯片（需 slide_id） | `slide_id` |
| `wpp.slide.items.duplicate` | 设置幻灯片：就地复制（需 slide_id，复制到相邻位置） | `slide_id` |
| `wpp.slide.items.insert` | 插入幻灯片：插入空白页 |  |
| `wpp.slide.items.list` | 查询幻灯片列表（返回全部幻灯片摘要列表） |  |
| `wpp.slide.items.move` | 设置幻灯片：移动位置（需 slide_id 与 to_position） | `slide_id`, `to_position` |

---

## 1. wpp.slide.items.batch_slides

#### 功能说明

批量插入多张幻灯片（通过 items[] 一次插入多张幻灯片，每项可设 layout、index）。

**幂等性**：是 — safe

#### 调用示例

批量插入多张幻灯片：

```json
{
  "file_id": "<FILE_ID>",
  "items": [
    {
      "layout": 1,
      "index": 1
    },
    {
      "layout": 2,
      "index": 2
    }
  ]
}
```

#### 参数说明

- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `items` (array, 必填): 待批量插入的幻灯片列表（必填）；每项可指定 layout（版式索引）、index（插入位置）
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `url` (string, 可选): 文档 URL；与 link_id、file_id 三选一

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "data": {
      "slide_id": 150995206,
      "slide_index": 1
    }
  }
}
```


---

## 2. wpp.slide.items.count

#### 功能说明

查询幻灯片总数（返回演示文稿幻灯片页数）。

**幂等性**：是 — safe

#### 调用示例

查询幻灯片总数：

```json
{
  "file_id": "<FILE_ID>"
}
```

#### 参数说明

- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `url` (string, 可选): 文档 URL；与 link_id、file_id 三选一

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "count_data": {
      "count": 1
    }
  }
}
```


---

## 3. wpp.slide.items.delete

#### 功能说明

删除幻灯片（需 slide_id）。

**幂等性**：是 — safe

#### 调用示例

删除幻灯片：

```json
{
  "file_id": "<FILE_ID>",
  "slide_id": 150995205
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
    "data": {
      "deleted_slide_id": 150995205
    }
  }
}
```


---

## 4. wpp.slide.items.duplicate

#### 功能说明

设置幻灯片：就地复制（需 slide_id，复制到相邻位置）。

**幂等性**：是 — safe

#### 调用示例

就地复制幻灯片：

```json
{
  "file_id": "<FILE_ID>",
  "slide_id": 150995204
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
  "data": {}
}
```


---

## 5. wpp.slide.items.insert

#### 功能说明

插入幻灯片：插入空白页。

**幂等性**：是 — safe

#### 调用示例

插入空白幻灯片：

```json
{
  "file_id": "<FILE_ID>",
  "index": 1,
  "layout": 1
}
```

#### 参数说明

- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `index` (number, 可选): 插入位置索引
- `layout` (number, 可选): 版式索引
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `url` (string, 可选): 文档 URL；与 link_id、file_id 三选一

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "data": {
      "slide_id": 150995204,
      "slide_index": 1
    }
  }
}
```


---

## 6. wpp.slide.items.list

#### 功能说明

查询幻灯片列表（返回全部幻灯片摘要列表）。

**幂等性**：是 — safe

#### 调用示例

查询幻灯片列表：

```json
{
  "file_id": "<FILE_ID>"
}
```

#### 参数说明

- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `url` (string, 可选): 文档 URL；与 link_id、file_id 三选一

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "list_data": {
      "count": 1,
      "slides": [
        {
          "layout": 1,
          "name": "幻灯片 1",
          "slide_id": 150995203,
          "slide_index": 1
        }
      ]
    }
  }
}
```


---

## 7. wpp.slide.items.move

#### 功能说明

设置幻灯片：移动位置（需 slide_id 与 to_position）。

**幂等性**：是 — safe

#### 调用示例

移动幻灯片位置：

```json
{
  "file_id": "<FILE_ID>",
  "slide_id": 150995204,
  "to_position": 1
}
```

#### 参数说明

- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `slide_id` (number, 必填): 幻灯片 ID
- `to_position` (number, 必填): 移动目标位置
- `url` (string, 可选): 文档 URL；与 link_id、file_id 三选一

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {}
}
```
