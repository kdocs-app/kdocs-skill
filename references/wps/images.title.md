# wps.images.title

## 1. wps.images.title

#### 功能说明

设置嵌入图片标题（IMAGE_TITLE，对应 setInlineShapeProps key=Title）。

**幂等性**：是 — safe

> 读取标题用 wps.images.info；index 变更规则同 wps.images.list。

#### 调用示例

文档设置：

```json
{
  "file_id": "<FILE_ID>",
  "index": 5,
  "title": "SmokeCC"
}
```

#### 参数说明

- `file_id` (string, 三选一必填: `url` / `link_id` / `file_id`): 文件 id
- `index` (number, 必填): 图片索引，从 1 开始
- `link_id` (string, 三选一必填: `url` / `link_id` / `file_id`): 分享 id
- `title` (string, 必填): 图片标题
- `url` (string, 三选一必填: `url` / `link_id` / `file_id`): 文档 URL

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "index": 5,
    "title": "SmokeCC"
  }
}
```

| 字段 | 类型 | 说明 |
|------|------|------|
| `code` | number | 0 表示成功 |
| `data` | object | 业务数据 |
| `message` | string | 结果说明 |
