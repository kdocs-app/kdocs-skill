# 本地 Markdown 上传为办公文档

> 本地 .md 转为云盘办公文档（默认 otl；亦可 docx/pdf）

**适用场景**：用户附本地 Markdown 或说「上传 md / 传到云盘」且未要求保留 .md

**触发词**：上传 md、传到云盘、上传 markdown、做成智能文档、md 转 otl、上传为 otl

- 本地 .md 或 Markdown 正文需写入云盘办公文档（默认 otl）

**工具链**：`create_file_with_content`（可选 `otl.insert_content` 分段）→ `read_file` 验证

## 涉及工具

| 工具 | 服务 | 用途 |
|------|------|------|
| `list_my_files` | drive | 仅当用户指定目标文件夹时取 drive_id；单级名根目录按名匹配 |
| `create_folder` | drive | 用户指定文件夹且不存在时创建；文件夹定位见 file-locating-guide |
| `create_file_with_content` | drive | name 只写文档名 + file_extension=otl + content（本地 md UTF-8 正文） |
| `otl.insert_content` | otl | content 超过单次上限时：先建空 otl，再分段写入 |
| `read_file` | drive | 上传后 read_file 验证（link_url 或 file_id），通过后再回复用户 |

## 执行流程

> 🎯 **默认**：用户只说「上传这个 md / 传到云盘」且**未**要求保留 markdown → **`create_file_with_content` 创建为 `.otl`**。
> 用户明确要求保留 `.md` → 改走 `references/workflows/upload-md-raw.md`（含图片 kdocsmd 转存）。
> 对话内正文新建在线 Markdown → `references/workflows/create-cloud-md.md`。

**意图路由**

| 用户说法 | 工具 | 云盘结果 |
|----------|------|----------|
| 上传 md / 传到云盘（未指定格式） | `create_file_with_content`（`file_extension=otl`） | `.otl` |
| 智能文档 / otl | `create_file_with_content`（`file_extension=otl`） | `.otl` |
| 转成 Word / docx / PDF | `create_file_with_content`（`file_extension=docx` 或 `pdf`） | `.docx` / `.pdf` |
| 保留 markdown / 上传为 .md | `upload_new_file`（步骤见 `references/workflows/upload-md-raw.md`） | `.md` |

**用户说法示例**：「上传这个 md」「把 md 传到云盘」「做成智能文档」。

```
步骤 1: 读取本地 .md
        - 读取 UTF-8 文本作为 content（不是 Base64，不是 upload_new_file）

步骤 2: 目标文件夹（可选）
        - 用户未指定：省略 drive_id / parent_id（勿先 list_my_files）
        - 用户指定文件夹：见 references/file-locating-guide.md

步骤 3: create_file_with_content
        - name: "xxx"（不带后缀；后缀只通过 file_extension 指定）
        - file_extension: otl（或 docx / pdf）
        - content: <Markdown 正文>

步骤 4: 正文超过单次上限
        - otl：create 空 otl（不传 content）→ otl.insert_content 分段
          （首段 prepend，后续 append；见 references/otl/insert_content.md）
        - **禁止**改走 upload_new_file(name=*.md)
        - docx / pdf：缩小 content 后重试，勿改走 upload_new_file

步骤 5: read_file 验证（必须）
        - 用 create 响应的 link_url 或 file_id 调 read_file
        - 验证通过后再向用户展示 link_url
```

参数见 `references/drive/create_and_upload.md` 中 `create_file_with_content`。
