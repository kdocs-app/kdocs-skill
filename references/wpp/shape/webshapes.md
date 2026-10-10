# 形状 · Web 形状

> facet: shape.webshapes

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `wpp.shape.webshapes.content` | 插入WebShape WebShape 内容（插入 WebShape | `height`, `slide_id`, `width` |

---

## 1. wpp.shape.webshapes.content

#### 功能说明

插入WebShape WebShape 内容（插入 WebShape；html 由服务端上传换 ID）。

**幂等性**：是 — safe

#### 参数说明

- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `height` (number, 必填): 高度
- `html` (string, 可选): HTML 内容（服务端上传换 WebshapeID）
- `left` (number, 可选): 左边距
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `mouse_event_penetration` (boolean, 可选): 鼠标事件穿透
- `need_proportionally_scale_thumbnail` (boolean, 可选): 缩略图等比缩放
- `org_name` (string, 可选): 写入 Tag 的 org 名称
- `resource_type` (string, 可选): WebShape 资源类型：ONLINE / OFFLINE
- `slide_id` (number, 必填): 幻灯片 ID
- `thumbnail_url` (string, 可选): 缩略图 HTTPS URL（可选）
- `top` (number, 可选): 上边距
- `url` (string, 可选): 文档 URL；与 link_id、file_id 三选一
- `width` (number, 必填): 宽度

#### 返回值说明

```json
{
  "shape_id": {
    "type": "integer"
  },
  "slide_id": {
    "type": "integer"
  }
}
```
