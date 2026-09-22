# wps.headings.heading_text_batch

## 1. wps.headings.heading_text_batch

#### 功能说明

按标题章节批量插入文本（HEADING_TEXT）：一次多条，各带格式，用于长文分批写入。

#### 调用示例

文档插入：

```json
{
  "file_id": "<FILE_ID>",
  "items": [
    {
      "heading_text": "SmokeHeadingSlice",
      "text": "smoke batch heading text",
      "heading_level": 1
    }
  ]
}
```

#### 参数说明

- `file_id` (string, 三选一必填: `url` / `link_id` / `file_id`): 文件 id；与 url、link_id 三选一
- `items` (array, 必填): 待插入条目数组：[{heading_text, text, position?, heading_level?}]
- `link_id` (string, 三选一必填: `url` / `link_id` / `file_id`): 分享 id；与 url、file_id 三选一
- `url` (string, 三选一必填: `url` / `link_id` / `file_id`): 文档 URL；与 link_id、file_id 三选一

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "items": [
      {"paragraph_index": 28, "begin": 611, "end": 635}
    ]
  }
}
```

| 字段 | 类型 | 说明 |
|------|------|------|
| `code` | number | 0 表示成功 |
| `data` | object | 业务数据 |
| `message` | string | 结果说明 |
