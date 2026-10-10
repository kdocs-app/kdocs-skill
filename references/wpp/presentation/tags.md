# 演示文稿 · 标签

> facet: presentation.tags

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `wpp.presentation.tags.add` | 添加演示文稿标签（演示文稿级标签，ADD 时可传 value） |  |
| `wpp.presentation.tags.delete` | 删除演示文稿标签 |  |
| `wpp.presentation.tags.tags` | 查询演示文稿标签：标签列表 |  |

---

## 1. wpp.presentation.tags.add

#### 功能说明

添加演示文稿标签（演示文稿级标签，ADD 时可传 value）。

**幂等性**：是 — safe

#### 调用示例

添加演示文稿标签：

```json
{
  "file_id": "<FILE_ID>",
  "name": "批次A标签",
  "value": "v1"
}
```

#### 参数说明

- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `name` (string, 可选): 名称
- `url` (string, 可选): 文档 URL；与 link_id、file_id 三选一
- `value` (string, 可选): value

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {}
}
```


---

## 2. wpp.presentation.tags.delete

#### 功能说明

删除演示文稿标签。

**幂等性**：是 — safe

#### 调用示例

删除演示文稿标签：

```json
{
  "file_id": "<FILE_ID>",
  "name": "批次A标签"
}
```

#### 参数说明

- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `name` (string, 可选): 名称
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

## 3. wpp.presentation.tags.tags

#### 功能说明

查询演示文稿标签：标签列表。

**幂等性**：是 — safe

#### 调用示例

查询演示文稿标签列表：

```json
{
  "file_id": "<FILE_ID>"
}
```

复核演示文稿标签列表：

```json
{
  "file_id": "<FILE_ID>"
}
```

复核演示文稿标签列表：

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
