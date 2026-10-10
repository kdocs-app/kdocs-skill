# 演示文稿 · 节

> facet: presentation.sections

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `wpp.presentation.sections.count` | 查询演示文稿分节分节总数 |  |
| `wpp.presentation.sections.data` | 演示文稿分节分节实体：支持 删除/插入 | `name`, `section_index`, `verb` |
| `wpp.presentation.sections.list` | 查询演示文稿分节分节列表 |  |
| `wpp.presentation.sections.name` | 演示文稿分节分节名称：支持 查询/设置 | `name`, `section_index`, `verb` |
| `wpp.presentation.sections.order` | 设置演示文稿分节分节顺序 | `section_index`, `to_pos` |
| `wpp.presentation.sections.slide_ids` | 查询演示文稿分节分节幻灯片列表 |  |

---

## 1. wpp.presentation.sections.count

#### 功能说明

查询演示文稿分节分节总数。

**幂等性**：是 — safe

#### 调用示例

查询演示文稿分节总数：

```json
{
  "file_id": "<FILE_ID>"
}
```

复核演示文稿分节总数：

```json
{
  "file_id": "<FILE_ID>"
}
```

复核演示文稿分节总数：

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
    "data": {
      "count": 2,
      "has_sections": true
    }
  }
}
```


---

## 2. wpp.presentation.sections.data

#### 功能说明

演示文稿分节分节实体：支持 删除/插入。

**幂等性**：是 — safe

#### 调用示例

插入演示文稿分节（指定幻灯片下标）：

```json
{
  "file_id": "<FILE_ID>",
  "verb": "insert",
  "name": "批次A测试节",
  "slide_index": 1
}
```

删除演示文稿分节：

```json
{
  "file_id": "<FILE_ID>",
  "verb": "delete",
  "section_index": 1
}
```

#### 参数说明

- `delete_slides` (boolean, 可选): 是否连同幻灯片一起删除（默认 false）
- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `name` (string, 必填): 名称
- `section_index` (number, 必填): 分节下标（1 起；delete 时必填；insert 可与 slide_index 二选一）
- `slide_index` (number, 可选): 幻灯片下标（1 起）
- `url` (string, 可选): 文档 URL；与 link_id、file_id 三选一
- `verb` (string, 必填): HTTP 动作：delete（删除） / insert（插入）。可选值：`delete` / `insert`

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "data": {
      "first_slide": 1,
      "name": "章节一",
      "section_id": "section_1",
      "section_index": 1
    }
  }
}
```


---

## 3. wpp.presentation.sections.list

#### 功能说明

查询演示文稿分节分节列表。

**幂等性**：是 — safe

#### 调用示例

查询演示文稿分节列表：

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
    "data": {
      "count": 2,
      "has_sections": true,
      "sections": [
        {
          "first_slide": 1,
          "name": "章节一",
          "section_id": "section_1",
          "section_index": 1,
          "slide_ids": [
            150995200,
            150995201
          ],
          "slides_count": 2
        }
      ]
    }
  }
}
```


---

## 4. wpp.presentation.sections.name

#### 功能说明

演示文稿分节分节名称：支持 查询/设置。

**幂等性**：是 — safe

#### 调用示例

查询演示文稿分节名称：

```json
{
  "file_id": "<FILE_ID>",
  "verb": "query"
}
```

设置演示文稿分节名称：

```json
{
  "file_id": "<FILE_ID>",
  "verb": "update",
  "name": "批次A节改名",
  "section_index": 1
}
```

#### 参数说明

- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `name` (string, 必填): 名称
- `section_index` (number, 必填): 分节下标（1 起）
- `url` (string, 可选): 文档 URL；与 link_id、file_id 三选一
- `verb` (string, 必填): HTTP 动作：query（查询） / update（设置）。可选值：`query` / `update`

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "data": {}
  }
}
```


---

## 5. wpp.presentation.sections.order

#### 功能说明

设置演示文稿分节分节顺序。

**幂等性**：是 — safe

#### 调用示例

设置演示文稿分节顺序：

```json
{
  "file_id": "<FILE_ID>",
  "section_index": 1,
  "to_pos": 1
}
```

#### 参数说明

- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `section_index` (number, 必填): 分节下标（1 起）
- `to_pos` (number, 必填): 目标位置（1 起）
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

## 6. wpp.presentation.sections.slide_ids

#### 功能说明

查询演示文稿分节分节幻灯片列表。

**幂等性**：是 — safe

#### 调用示例

查询演示文稿分节幻灯片列表：

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
    "data": {
      "count": 2,
      "has_sections": true,
      "sections": [
        {
          "section_index": 1,
          "slide_ids": [
            150995200,
            150995201
          ],
          "slides_count": 2
        }
      ]
    }
  }
}
```
