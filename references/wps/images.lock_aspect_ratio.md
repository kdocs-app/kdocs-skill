# wps.images.lock_aspect_ratio

## 1. wps.images.lock_aspect_ratio

#### 功能说明

设置嵌入图片是否锁定纵横比（IMAGE_LOCK_ASPECT_RATIO）。

**幂等性**：是 — safe

#### 调用示例

文档设置：

```json
{
  "file_id": "<FILE_ID>",
  "index": 5,
  "lock_aspect_ratio": true
}
```

#### 参数说明

- `file_id` (string, 三选一必填: `url` / `link_id` / `file_id`): 文件 id
- `index` (number, 必填): 图片索引，从 1 开始
- `link_id` (string, 三选一必填: `url` / `link_id` / `file_id`): 分享 id
- `lock_aspect_ratio` (boolean, 必填): true 锁定，false 不锁定
- `url` (string, 三选一必填: `url` / `link_id` / `file_id`): 文档 URL

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "index": 5,
    "lock_aspect_ratio": -1
  }
}
```

| 字段 | 类型 | 说明 |
|------|------|------|
| `code` | number | 0 表示成功 |
| `data` | object | 业务数据 |
| `message` | string | 结果说明 |
