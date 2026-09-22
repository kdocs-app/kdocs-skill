# wps.headings.heading_text

## 1. wps.headings.heading_text

#### 功能说明

按标题章节插入文本（HEADING_TEXT）：默认章节末尾，支持章节开头/锚点标题之前。

#### 调用示例

文档插入：

```json
{
  "file_id": "<FILE_ID>",
  "heading_level": 1,
  "heading_text": "SmokeHeadingSlice",
  "position": "after",
  "text": "smoke example text"
}
```

#### 参数说明

- `file_id` (string, 三选一必填: `url` / `link_id` / `file_id`): 文件 id；与 url、link_id 三选一
- `heading_level` (number, 可选): 新段若为标题，设级别（0=正文,1-9=Heading）
- `heading_text` (string, 必填): 锚点标题文本（子串匹配）
- `link_id` (string, 三选一必填: `url` / `link_id` / `file_id`): 分享 id；与 url、file_id 三选一
- `position` (string, 可选): 插入位置：HEADING_END(章节末尾,缺省) / HEADING_BEGIN(章节开头) / HEADING_BEFORE(锚点标题之前)。可选值：`HEADING_BEFORE` / `HEADING_BEGIN` / `HEADING_END`
- `text` (string, 必填): 要插入的文本
- `url` (string, 三选一必填: `url` / `link_id` / `file_id`): 文档 URL；与 link_id、file_id 三选一

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "paragraph_index": 28,
    "begin": 611,
    "end": 630
  }
}
```

| 字段 | 类型 | 说明 |
|------|------|------|
| `code` | number | 0 表示成功 |
| `data` | object | 业务数据 |
| `message` | string | 结果说明 |
