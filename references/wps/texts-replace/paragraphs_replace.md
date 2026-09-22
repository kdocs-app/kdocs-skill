# wps.texts.replace

#### 功能说明

按段落原文精确替换

> L1 由 URL verb 区分能力，请求体勿带 properties（见 ein DocsMeta.OmitProperties）

#### 调用示例

段落替换：

```json
{
  "file_id": "<FILE_ID>",
  "scope": "paragraphs",
  "items": [
    {
      "src": "smoke example text",
      "dst": "smoke example text revised"
    }
  ],
  "skip_heading": true,
  "skip_on_multi_match": true
}
```

#### 参数说明

- `file_id` (string, 三选一必填: `url` / `link_id` / `file_id`): 文件 id；与 url、link_id 三选一
- `link_id` (string, 三选一必填: `url` / `link_id` / `file_id`): 分享 id；与 url、file_id 三选一
- `url` (string, 三选一必填: `url` / `link_id` / `file_id`): 文档 URL；与 link_id、file_id 三选一
- `scope` (string, 必填): 操作范围：documents（整篇文档，缺省） / paragraphs（按段落精确替换）。可选值：`documents` / `paragraphs`
- `items` (array, 必填): 待替换条目数组：[{src:原文整段, dst:润色后文本}]，≤100 条
- `skip_heading` (boolean, 可选): 是否跳过标题段（缺省 true）
- `skip_on_multi_match` (boolean, 可选): 多命中行为：true=整项跳过(缺省) / false=命中全部替换

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "skipped": 1,
    "item_status": [0]
  }
}
```

| 字段 | 类型 | 说明 |
|------|------|------|
| `code` | number | 0 表示成功 |
| `data` | object | 业务数据 |
| `message` | string | 结果说明 |

