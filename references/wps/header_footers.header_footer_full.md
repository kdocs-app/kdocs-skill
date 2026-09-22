# wps.header_footers.header_footer_full

## 1. wps.header_footers.header_footer_full

#### 功能说明

某类型页眉/页脚一次全量（HEADER_FOOTER_FULL）：内容 + 字体 + 对齐 + 奇偶页/首页开关。

**幂等性**：是 — safe

#### 调用示例

文档设置：

```json
{
  "file_id": "<FILE_ID>",
  "content": "smoke example text",
  "header_footer_type": 1,
  "section_index": 1
}
```

#### 参数说明

- `file_id` (string, 三选一必填: `url` / `link_id` / `file_id`): 文件 id；与 url、link_id 三选一
- `header_footer_type` (number, 可选): header footer type
- `link_id` (string, 三选一必填: `url` / `link_id` / `file_id`): 分享 id；与 url、file_id 三选一
- `section_index` (number, 可选): 节索引，从 1 开始
- `url` (string, 三选一必填: `url` / `link_id` / `file_id`): 文档 URL；与 link_id、file_id 三选一
- `full_data` (object, 可选): 页眉页脚一次全量（HEADER_FOOTER_FULL）：{content?, font_style?:{font_name,font_size}, alignment?, different_odd_even?, different_first?, is_header?(缺省 true=页眉)}

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
