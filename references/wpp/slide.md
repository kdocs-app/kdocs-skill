# 演示文稿与页面

> 从外部 PPTX 导入幻灯片等工作流

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `wpp.import_slides` | 将外部 PPTX 的指定页面导入到已有演示文稿 | `url`\|`link_id`\|`file_id`, `object_url`, `slide_idx`, `source_idxs` |

---

## 1. wpp.import_slides

#### 功能说明

从外部 PPTX 文件（通过 URL 获取）中选取指定幻灯片，导入到已有在线演示文稿的指定位置。

**适用于**：将其他 PPTX 文件中的页面合并到已有在线演示文档中。

**幂等性**：否 — 重复调用会再次导入同一组页面，重试前先确认目标文稿中是否已导入成功

> object_url 域名需在服务端白名单内，否则返回 `validate_object_url` 错误

#### 调用示例

导入源 PPTX 第 1 页到目标文稿第 1 页位置：

```json
{
  "link_id": "cupf4t47i6Fx",
  "object_url": "https://example.com/slides/demo.pptx",
  "slide_idx": 0,
  "source_idxs": [
    0
  ]
}
```

导入源 PPTX 第 1、3、5 页到目标文稿第 2 页位置（原第 2 页及之后后移）：

```json
{
  "link_id": "cupf4t47i6Fx",
  "object_url": "https://example.com/slides/demo.pptx",
  "slide_idx": 1,
  "source_idxs": [
    0,
    2,
    4
  ]
}
```

#### 参数说明

- `url` (string, 三选一必填: `url` / `link_id` / `file_id`): 金山文档 URL
- `link_id` (string, 三选一必填: `url` / `link_id` / `file_id`): 分享链接 ID
- `file_id` (string, 三选一必填: `url` / `link_id` / `file_id`): 文件 ID
- `object_url` (string, 必填): 源 PPTX 文件的下载 URL
- `slide_idx` (integer, 必填): 插入位置，目标文稿中的幻灯片索引，**从 0 开始**。导入的页面占据该索引位置，原位置及之后的页面顺序后移。用户说"插入到第 n 页"时传 n-1。超出范围时自动尾插
- `source_idxs` (array, 必填): 要导入的源文件幻灯片索引数组，**从 0 开始**

#### 返回值说明

```json
{
  "result": "ok",
  "detail": {
    "res": [
      {
        "cmdName": "docTemplate",
        "code": 0,
        "errName": "S_OK",
        "msg": "execute result"
      }
    ]
  }
}

```

| 字段 | 类型 | 说明 |
|------|------|------|
| `result` | string | 请求成功为 `ok` |
| `detail.res[].cmdName` | string | 命令名，如 `docTemplate` |
| `detail.res[].code` | integer | `0` 表示该条执行成功 |
| `detail.res[].errName` | string | 如 `S_OK` |
