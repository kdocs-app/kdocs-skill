# wps.images.scale

## 1. wps.images.scale

#### 功能说明

设置嵌入图片宽高缩放百分比（IMAGE_SCALE，对应 ScaleWidth/ScaleHeight）。

**幂等性**：是 — safe

> 与 wps.images.resize（绝对磅值）不同；重置 100% 时传 scale_width=100、scale_height=100。

#### 调用示例

文档设置：

```json
{
  "file_id": "<FILE_ID>",
  "index": 5,
  "scale_height": 1,
  "scale_width": 1
}
```

#### 参数说明

- `file_id` (string, 三选一必填: `url` / `link_id` / `file_id`): 文件 id
- `index` (number, 必填): 图片索引，从 1 开始
- `link_id` (string, 三选一必填: `url` / `link_id` / `file_id`): 分享 id
- `scale_height` (number, 必填): 高度缩放百分比（如 100 表示 100%）
- `scale_width` (number, 必填): 宽度缩放百分比
- `url` (string, 三选一必填: `url` / `link_id` / `file_id`): 文档 URL

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "index": 5,
    "scale_width": 6.666666507720947,
    "scale_height": 6.666666507720947
  }
}
```

| 字段 | 类型 | 说明 |
|------|------|------|
| `code` | number | 0 表示成功 |
| `data` | object | 业务数据 |
| `message` | string | 结果说明 |
