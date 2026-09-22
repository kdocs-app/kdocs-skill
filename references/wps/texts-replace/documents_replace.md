# wps.texts.replace

#### 功能说明

查找并替换文本

> L1 由 URL verb 区分能力，请求体勿带 properties（见 ein DocsMeta.OmitProperties）

#### 调用示例

文档替换：

```json
{
  "file_id": "<FILE_ID>",
  "find_text": "SMOKE_SLICE_ANCHOR_2026-09-03",
  "is_all": true,
  "replace_text": "SMOKE"
}
```

#### 参数说明

- `file_id` (string, 三选一必填: `url` / `link_id` / `file_id`): 文件 id；与 url、link_id 三选一
- `link_id` (string, 三选一必填: `url` / `link_id` / `file_id`): 分享 id；与 url、file_id 三选一
- `url` (string, 三选一必填: `url` / `link_id` / `file_id`): 文档 URL；与 link_id、file_id 三选一
- `scope` (string, 必填): 操作范围：documents（整篇文档，缺省） / paragraphs（按段落精确替换）。可选值：`documents` / `paragraphs`
- `find_text` (string, 必填): 查找文本
- `is_all` (boolean, 可选): 是否替换全部
- `replace_text` (string, 可选): 替换文本；传空串 "" 表示删除匹配文本

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {}
}
```

| 字段 | 类型 | 说明 |
|------|------|------|
| `code` | number | 0 表示成功 |
| `data` | object | 业务数据 |
| `message` | string | 结果说明 |

