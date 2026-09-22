# wps.headings.heading_meta

## 1. wps.headings.heading_meta

#### 功能说明

按标题文本定位章节（HEADING_META）：返回标题段号/级别/章节起止段号，可选返回该节全部文本或章节树。

#### 调用示例

文档查询：

```json
{
  "file_id": "<FILE_ID>",
  "heading_text": "SmokeHeadingSlice",
  "properties": []
}
```

#### 参数说明

- `file_id` (string, 三选一必填: `url` / `link_id` / `file_id`): 文件 id；与 url、link_id 三选一
- `heading_text` (string, 必填): 锚点标题文本（子串匹配，取第一个命中）
- `link_id` (string, 三选一必填: `url` / `link_id` / `file_id`): 分享 id；与 url、file_id 三选一
- `properties` (array, 可选): 返回内容：HEADING_META(位置,缺省) / HEADING_TEXT(该节文本) / HEADING_ALL(章节树)
- `url` (string, 三选一必填: `url` / `link_id` / `file_id`): 文档 URL；与 link_id、file_id 三选一

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "heading_paragraph_index": 28,
    "heading_level": 1,
    "section_start": 28,
    "section_end": 28,
    "heading_text": "SmokeHeadingSlice",
    "found": true
  }
}
```

| 字段 | 类型 | 说明 |
|------|------|------|
| `code` | number | 0 表示成功 |
| `data` | object | 业务数据 |
| `message` | string | 结果说明 |
