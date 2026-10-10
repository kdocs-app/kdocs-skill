# 形状 · 标签

> facet: shape.tags

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `wpp.shape.tags.add` | 添加形状标签 | `slide_id` |
| `wpp.shape.tags.delete` | 删除形状标签 | `slide_id` |
| `wpp.shape.tags.tags` | 查询形状标签：标签列表（需 slide_id 与 shape_id） | `slide_id` |

---

## 1. wpp.shape.tags.add

#### 功能说明

添加形状标签。

**幂等性**：是 — safe

#### 调用示例

添加形状标签：

```json
{
  "file_id": "<FILE_ID>",
  "slide_id": 150995203,
  "shape_id": 2,
  "name": "tag-b1",
  "value": "batch-b"
}
```

#### 参数说明

- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `name` (string, 可选): 名称
- `shape_id` (number, 可选): 形状 ID
- `slide_id` (number, 必填): 幻灯片 ID
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

## 2. wpp.shape.tags.delete

#### 功能说明

删除形状标签。

**幂等性**：是 — safe

#### 调用示例

删除形状标签：

```json
{
  "file_id": "<FILE_ID>",
  "slide_id": 150995203,
  "shape_id": 2,
  "name": "tag-b1"
}
```

#### 参数说明

- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `name` (string, 可选): 名称
- `shape_id` (number, 可选): 形状 ID
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

## 3. wpp.shape.tags.tags

#### 功能说明

查询形状标签：标签列表（需 slide_id 与 shape_id）。

**幂等性**：是 — safe

#### 调用示例

查询形状标签列表：

```json
{
  "file_id": "<FILE_ID>",
  "slide_id": 150995203,
  "shape_id": 2
}
```

复核形状标签列表（名称被归一为大写 TAG-B1）：

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
    "data": {
      "count": 1,
      "tags": [
        {
          "name": "TAG-B1",
          "value": "batch-b"
        }
      ]
    }
  }
}
```
