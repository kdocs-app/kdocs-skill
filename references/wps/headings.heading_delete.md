# wps.headings.heading_delete

## 1. wps.headings.heading_delete

#### 功能说明

按标题删除整节（HEADING_DELETE）：标题段 + 其下全部内容，保留下一同级标题。

#### 调用示例

文档删除：

```json
{
  "file_id": "<FILE_ID>",
  "heading_text": "SmokeHeadingSlice"
}
```

#### 参数说明

- `file_id` (string, 三选一必填: `url` / `link_id` / `file_id`): 文件 id；与 url、link_id 三选一
- `heading_text` (string, 必填): 锚点标题文本（子串匹配）
- `link_id` (string, 三选一必填: `url` / `link_id` / `file_id`): 分享 id；与 url、file_id 三选一
- `url` (string, 三选一必填: `url` / `link_id` / `file_id`): 文档 URL；与 link_id、file_id 三选一

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "deleted_paragraph_count": 1,
    "success": true
  }
}
```

| 字段 | 类型 | 说明 |
|------|------|------|
| `code` | number | 0 表示成功 |
| `data` | object | 业务数据 |
| `message` | string | 结果说明 |
