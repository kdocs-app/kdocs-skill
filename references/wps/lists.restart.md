# wps.lists.restart

## 1. wps.lists.restart

#### 功能说明

在指定段落处重新开始列表编号（LIST_RESTART）。

**幂等性**：否 — unsafe

#### 调用示例

段落设置：

```json
{
  "file_id": "<FILE_ID>",
  "paragraph_index": 17
}
```

#### 参数说明

- `file_id` (string, 三选一必填: `url` / `link_id` / `file_id`): 文件 id
- `link_id` (string, 三选一必填: `url` / `link_id` / `file_id`): 分享 id
- `paragraph_index` (number, 必填): 段落索引，从 1 开始
- `url` (string, 三选一必填: `url` / `link_id` / `file_id`): 文档 URL

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "paragraph_index": 17,
    "old_list_string": "1.",
    "new_list_string": "1.",
    "list_level_number": 1
  }
}
```

| 字段 | 类型 | 说明 |
|------|------|------|
| `code` | number | 0 表示成功 |
| `data` | object | 业务数据 |
| `message` | string | 结果说明 |
