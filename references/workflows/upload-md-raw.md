# 本地 Markdown 上传为在线文档（保留 .md）

> 将本地 .md 上传为在线 Markdown，并把图片转存为 kdocsmd://（不转为 otl/docx）

**适用场景**：用户提供本地 Markdown，并明确要求保留 .md 格式；正文可含本地图、base64 或外链图

**触发词**：原样上传 md、保留 markdown、保持 md 格式、上传成 md、不要转智能文档、不要转 otl

- 本地 .md 上传后，云盘文件后缀仍为 .md

**工具链**：`search_files`（查重）→ `upload_new_file`（仅占位）→ `upload_attachment`（逐图）→ `upload_replace_file` → `link_url`

## 涉及工具

| 工具 | 服务 | 用途 |
|------|------|------|
| `search_files` | drive | 按本地文件名（含 .md）查重；若命中仅含占位内容的文件则复用 file_id，若已是完整正文则走覆盖流程或向用户确认 |
| `list_my_files` | drive | 仅当用户指定目标文件夹时取 drive_id；根目录按名匹配文件夹 |
| `create_folder` | drive | 用户指定文件夹且不存在时创建；文件夹定位见 file-locating-guide |
| `get_file_info` | drive | search 命中同名 .md 时确认后缀与 size，判断是否仅为占位内容 |
| `read_file` | drive | 仅当需判断是否仅为占位内容时读取在线 Markdown 正文（不传 format） |
| `upload_new_file` | drive | 仅当 search 无可复用占位文件时创建占位文档并取得 file_id（本 API 不支持 on_name_conflict，靠查重避免重复） |
| `upload_attachment` | drive | 对正文中每张图 upload 到该 file_id；成功改写为 kdocsmd://object_id |
| `upload_replace_file` | drive | 写入图片已转存后的最终 UTF-8 正文 Base64 |

## 执行流程

> 🎯 **核心路径**：`search_files` 查重 →（必要时）`upload_new_file` 创建占位并取得 `file_id` → `upload_attachment` → `upload_replace_file` 写入最终正文。
> **禁止**未转存图片就把含 `./img` / base64 / 外链的原文当作最终正文上传（云盘无法解析本地路径）。
> 图片规则见 `references/markdown.md`：**一律尝试转为 kdocsmd://**；失败则**保留原 src**。
> **禁止**误走 `create_file_with_content` 默认 otl；用户未要求保留 md 时见「上传 md 默认转为智能文档」`references/workflows/md-upload-to-otl.md`。
> 对话内无本地文件、仅有正文要新建在线 Markdown → `references/workflows/create-cloud-md.md`。

**用户说法示例**：「原样上传 md」「保留 markdown 格式」「上传成 .md，不要转智能文档」。

### 查重与复用（避免产生多个空的 .md 文件）

1. `search_files(keyword="xxx.md")`（或本地 basename）。
2. 若命中同名在线 Markdown：
   - **仅为占位内容**（`size` 很小，如 ≤64B；或 `read_file` 正文去空白后仅 `#` / 空）→ **复用其 `file_id`**，从图片转存步骤继续；**禁止**再 `upload_new_file`。
   - **已写入完整正文**（正文明显长于占位、或用户要覆盖）→ 走 `references/workflows/replace-cloud-md.md`（`upload_attachment` + `upload_replace_file`）。
   - **用户要保留两份** → 换 `name` 后再 `upload_new_file`。
3. 仅当 search **无可复用占位文件**、且需新建时，才 `upload_new_file`。
4. `upload_new_file` **不支持** `on_name_conflict`；禁止未查重就创建占位。若同名已存在且非可复用占位，改走「覆盖已有在线 Markdown」（`references/workflows/replace-cloud-md.md`）或换文件名，**禁止**盲目重试创建（默认 rename 会产生 `(1)(2)` 空文件）。

### 重试规则（单次任务内）

- 一旦已取得 `file_id`（新建占位或复用已有），本次任务后续失败**只能**对该 `file_id` 继续 `upload_attachment` + `upload_replace_file`。
- **禁止**因 Token/参数/图片转存失败而再次 `upload_new_file`。
- 未完成的占位文件由 Agent **复用同一 `file_id` 继续写完**，不要求用户手动删除。

```
步骤 1: 读取本地 .md（UTF-8 文本，勿当二进制上传）

步骤 2: search_files 查重 / 复用（见上）

步骤 3: 目标文件夹（可选，见 file-locating-guide）

步骤 4: upload_new_file（仅无可复用占位时）
        - name: "xxx.md"（与本地 basename 一致，除非用户指定新名）
        - content_base64: 极简占位如 "#\n" 的 Base64（`IyMK`）
        → 记录 file_id、link_url

步骤 5: 图片转存（见 references/markdown.md）
        - ![](...) / <img src>
        - 本地路径 / data: / https: → upload_attachment
        - 成功 → kdocsmd://{object_id}；失败 → 保留原 src

步骤 6: upload_replace_file
        - file_id + 最终正文 content_base64
        - 一般不传 content_format

步骤 7: 展示 link_url；图片 warnings 一并说明
```

覆盖已有在线 Markdown → `references/workflows/replace-cloud-md.md`。
参数见 `references/drive/create_and_upload.md`。
