# 对话内正文新建在线 Markdown

> 将对话中的 Markdown 正文新建为在线 Markdown，并把图片转存为 kdocsmd://

**适用场景**：Agent 生成或用户粘贴 Markdown，需保存为在线 Markdown（非 otl），且正文可含本地路径/base64/外链图

**触发词**：保存成 md、新建在线 md、写成 markdown 文件、存为 md、创建 md 文档

- 正文在对话/API content 中，无本地 .md 文件
- 云盘文件后缀须为 .md（非 otl/docx）

**工具链**：`search_files`（查重）→ `create_file_with_content`（仅占位）→ `upload_attachment` → `upload_replace_file`

## 涉及工具

| 工具 | 服务 | 用途 |
|------|------|------|
| `search_files` | drive | 按目标文档名查重；若命中仅含占位内容的文件则复用 file_id，若已是完整正文则走覆盖流程或向用户确认 |
| `get_file_info` | drive | search 命中时确认后缀与 size，判断是否仅为占位内容 |
| `read_file` | drive | 仅当需判断是否仅为占位内容时读取在线 Markdown 正文（不传 format） |
| `create_file_with_content` | drive | 仅当 search 无可复用占位文件时创建占位；name 不带后缀，file_extension=md |
| `upload_attachment` | drive | 对正文中每张待转存图上传至该 file_id，得 object_id |
| `upload_replace_file` | drive | 写入图片已改写为 kdocsmd:// 的最终 UTF-8 正文 |

## 执行流程

> 🎯 **与「本地文件上传为在线 Markdown」的区别**：正文来自对话/`content` 字符串，不是读取本地磁盘文件（本地文件见 `references/workflows/upload-md-raw.md`）。
> **与「上传 md 默认转为智能文档」的区别**：本流程 `file_extension=md`，云盘仍为 `.md`（默认 otl 见 `references/workflows/md-upload-to-otl.md`）。
> 图片转存规则（解析 src、失败保留原文、kdocsmd 协议）见 `references/markdown.md`。

**用户说法示例**：「把下面内容保存成在线 md」「生成 README 存到云盘」「这段 markdown 建成 .md 文件」。

### 查重与复用

与「本地文件上传为在线 Markdown」相同：先 `search_files`；仅为占位内容（小 size / 正文仅 `#`）→ **复用 `file_id`**；已写入完整正文 → 走「覆盖已有在线 Markdown」（`references/workflows/replace-cloud-md.md`）；仅无可复用占位时才 `create_file_with_content`（**禁止**未查重就创建）。

### 重试规则

已取得 `file_id` 后，**禁止**再次创建占位；失败只对该 `file_id` 做 `upload_attachment` + `upload_replace_file`。

```
步骤 1: 准备 Markdown 正文
        - 来自 Agent 生成或用户粘贴的 UTF-8 文本
        - 不得把整文件当二进制 Base64 传给 content

步骤 2: search_files 查重 / 复用

步骤 3: 占位创建（仅无可复用占位时，须先拿到 file_id）
        create_file_with_content:
          - name: "文档名"（不带 .md 后缀）
          - file_extension: md
          - content: "#\n" 或极简占位（后续 upload_replace_file 覆盖全文）
        → 记录 data.file_id、data.link_url

步骤 4: 图片转存（共用规则见 references/markdown.md）
        - 解析 ![](...) 与 <img src="...">
        - 本地路径 / data: / https: → upload_attachment
        - 成功 → kdocsmd://{object_id}；失败 → 保留原 src

步骤 5: upload_replace_file
        - file_id: 步骤 3 的 file_id
        - content_base64: 最终正文 UTF-8 Base64
        - 目标为 .md 时一般不传 content_format

步骤 6: 向用户展示 link_url；若有图片 warnings 一并说明
```

本地已有 `.md` 文件 → 「本地文件上传为在线 Markdown」`references/workflows/upload-md-raw.md`。
覆盖已有在线 Markdown → `references/workflows/replace-cloud-md.md`。
导出到本地文件 → `references/workflows/download-cloud-md.md`。
