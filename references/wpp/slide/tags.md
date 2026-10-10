# 幻灯片 · 标签

> facet: slide.tags

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `wpp.slide.tags.add` | 添加幻灯片标签（需 slide_id、name，ADD 时可传 value） | `slide_id` |
| `wpp.slide.tags.delete` | 删除幻灯片标签（需 slide_id 与 name） | `slide_id` |
| `wpp.slide.tags.tags` | 查询幻灯片标签：标签列表（需 slide_id；可按 name 过滤） | `slide_id` |

---

## 1. wpp.slide.tags.add

#### 功能说明

添加幻灯片标签（需 slide_id、name，ADD 时可传 value）。

**幂等性**：是 — safe

#### 调用示例

为幻灯片添加标签：

```json
{
  "file_id": "<FILE_ID>",
  "slide_id": 150995203,
  "name": "标签E1",
  "value": "v1"
}
```

#### 参数说明

- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `name` (string, 可选): 名称
- `slide_id` (number, 必填): 幻灯片 ID
- `url` (string, 可选): 文档 URL；与 link_id、file_id 三选一
- `value` (string, 可选): value

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "data": {
      "slide_id": 150995203
    }
  }
}
```


---

## 2. wpp.slide.tags.delete

#### 功能说明

删除幻灯片标签（需 slide_id 与 name）。

**幂等性**：是 — safe

#### 调用示例

删除幻灯片标签：

```json
{
  "file_id": "<FILE_ID>",
  "slide_id": 150995203,
  "name": "标签E1"
}
```

#### 参数说明

- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `name` (string, 可选): 名称
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

## 3. wpp.slide.tags.tags

#### 功能说明

查询幻灯片标签：标签列表（需 slide_id；可按 name 过滤）。

**幂等性**：是 — safe

#### 调用示例

查询幻灯片标签列表：

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
    "data": {
      "count": 1,
      "tags": [
        {
          "name": "标签1",
          "value": "v1"
        }
      ]
    }
  }
}
```
