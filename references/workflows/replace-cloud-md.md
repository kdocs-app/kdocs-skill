# 本地 Markdown 覆盖在线 Markdown

> 用本地 .md 全量覆盖已有在线 Markdown，并把图片转存为 kdocsmd://

**适用场景**：用户用本地 md 更新/同步云盘中的在线 Markdown，正文可含本地图、base64 或外链

**触发词**：覆盖 md、覆盖在线 md、同步 md、更新在线 md、用本地 md 更新、同步本地 md

- 本地 .md 全量覆盖已有在线 Markdown

**工具链**：`search_files`（可选）→ `upload_attachment`（逐图）→ `upload_replace_file`

## 涉及工具

| 工具 | 服务 | 用途 |
|------|------|------|
| `search_files` | drive | 用户未给 file_id / 链接时，按文件名搜索目标在线 Markdown |
| `get_file_info` | drive | 确认目标后缀为 .md，并取 file_id |
| `upload_attachment` | drive | 将正文中图片上传至目标 file_id，改写为 kdocsmd:// |
| `upload_replace_file` | drive | 写入转存后的最终 UTF-8 正文 Base64 |

## 执行流程

> 🎯 **核心工具**：`upload_replace_file`。目标文档 **已存在**，直接用其 `file_id` 做 `upload_attachment`。
> 图片规则见 `references/markdown.md`：**一律尝试转为 kdocsmd://**；失败则**保留原 src**。
> **禁止**覆盖 `.otl`（智能文档用 `otl.insert_content` 等）。

**用户说法示例**：「用本地这个 md 更新云盘那份」「覆盖一下在线的 xx.md」。

```
步骤 1: 定位在线 Markdown → 取得 file_id

步骤 2: 读取本地 .md（UTF-8）

步骤 3: 图片转存到该 file_id（见 references/markdown.md）

步骤 4: upload_replace_file(file_id, content_base64=最终正文)
        - 一般不传 content_format

步骤 5: 确认覆盖成功；图片 warnings 一并说明
```

本地文件新建上传 → 「本地文件上传为在线 Markdown」`references/workflows/upload-md-raw.md`。
对话内正文新建 → 「对话内新建在线 Markdown」`references/workflows/create-cloud-md.md`。
参数见 `references/drive/create_and_upload.md`。
