# 在线 Markdown（.md）与图片

在线 Markdown 正文中的图片使用 **`kdocsmd://{object_id}`** 协议（`object_id` 来自 `upload_attachment`）。与智能文档（otl）的 `picture.sourceKey` 同源 API，但**写入文档的格式不同**。

完整工作流见 `references/workflows/`：

| 意图 | 详细步骤 |
|------|----------|
| 本地 `.md` 上传为在线 Markdown（含图片转存） | `references/workflows/upload-md-raw.md` |
| 本地 `.md` 覆盖已有在线 Markdown | `references/workflows/replace-cloud-md.md` |
| 对话内正文新建在线 Markdown（含图片转存） | `references/workflows/create-cloud-md.md` |
| 在线 Markdown 导出到本地 | `references/workflows/download-cloud-md.md` |
| 仅读取正文、不保存本地文件 | `references/workflows/read-cloud-md.md` |

---

## 图片转存（上传到云盘，三入口共用）

### 须解析的图片 src

| 类型 | 示例 | 处理 |
|------|------|------|
| 本地相对/绝对路径 | `./img/a.png`、`/path/a.png` | 读盘 → `upload_attachment(content_base64)` |
| Base64 | `data:image/png;base64,...` | 解码 → `upload_attachment(content_base64)` |
| 外链 | `https://...` | `upload_attachment(url=...)` |
| 已是 kdocsmd | `kdocsmd://SK` | 跳过（或同文档内复用） |

同时处理 Markdown 图片语法 `![]()` 与 HTML `img` 标签（在线 Markdown 持久化常见 HTML）。

### 成功与失败

- **成功**：将 src 替换为 `kdocsmd://{object_id}`（保留 alt、width/height 等其它属性）。
- **失败**（读不到原图、`upload_attachment` 失败）：**保留原文 src**，在汇总里用 `warnings` 列出，不阻断整篇。

### 为何必须先有 `file_id`

`upload_attachment` 要求目标文档已存在。新建流程：

```
search_files 查重；若有可复用的占位文件则复用（见下）
  → 占位创建（upload_new_file 或 create_file_with_content，极简正文）
  → 取得 file_id
  → 逐张 upload_attachment + 改写正文
  → upload_replace_file 写入最终 Markdown
```

覆盖流程：已有 `file_id` → 转存图片 → `upload_replace_file`。

### 占位、复用与重试（禁止产生多个空的 `.md` 文件）

因必须先占位才有 `file_id`，**占位成功但图片转存或写入最终正文失败**时，云盘会出现仅含 `#\n` 的小文件。规则：

| 情况 | 处理 |
|------|------|
| 新建前 `search_files` 命中同名 `.md`，`size` 很小（如 ≤64B）或正文去空白后仅 `#` | **复用该 `file_id`**，从 `upload_attachment` 继续；**禁止**再 `upload_new_file` / `create_file_with_content` |
| 命中同名且已写入完整正文 | 走「覆盖已有在线 Markdown」（`references/workflows/replace-cloud-md.md`），或向用户确认是否覆盖 |
| 首次占位 | 必须先 `search_files`；`upload_new_file` 不支持 `on_name_conflict`；`create_file_with_content` 亦须先查重再创建 |
| 本次任务已取得 `file_id` 后任意步骤失败 | 只对该 `file_id` 重试 `upload_attachment` + `upload_replace_file`；**禁止**第二次占位创建 |

未完成的占位文件由 Agent **复用同一 `file_id` 继续写完**，不要求用户手动删除。详见 `references/workflows/upload-md-raw.md`。

### 与 otl 的差异

| | 在线 Markdown | 智能文档 `.otl` |
|--|----------|-----------------|
| 引用形式 | Markdown 内 `kdocsmd://` 协议 | `picture` 块 `sourceKey` |
| 批量写入 | 改写 Markdown 后 `upload_replace_file` | `otl.insert_content`（图须 base64/https）或 `block_insert` |

---

## 图片导出（导出到本地）

### 模式 `inline_https`（默认）

用户要「下载 md」「导出到本地」且未要求把图片另存为本地文件时：

1. `read_file(file_id)`（不传 `format`）。**勿依赖** `enable_upload_medias`：对在线 Markdown 它**不会**把正文内 `kdocsmd://` 转为 https（该参数主要作用于 docx/pdf/otl 等抽取路径）。
2. 对 `content` 中每个 `kdocsmd://{attachment_id}` 调 `download_attachment(file_id, attachment_id)` → 将正文替换为返回的 **`download_url`**（HTTPS 临时链；**不是** `download_file` 的 `data.url`）。
3. 保存单个本地 `.md` 文件。
4. **必须告知用户**：图片链接约 **10 分钟**有效，过期需重新导出。

### 模式 `local_assets`（用户选择）

触发词示例：带图下载、图片保存到本地、离线能看、保存到 assets、连同图片。

1. `read_file` 取源码（不传 `format`）。
2. 提取 `kdocsmd://{attachment_id}`（及需保存到本地的 https）。
3. 对每个附件：`download_attachment` → 用登录态 GET 返回的 **`download_url`** 写入 `{md主文件名}.assets/`（与 md 同目录）。
4. 正文改为相对路径形式：`![]` + `./文档名.assets/图片文件名`。
5. 单张失败：保留原 src + warnings。

沙箱环境保存文件须带 `wps_sid`（见 `drive/read_and_download.md` 中 `download_file` 的沙箱保存说明）。

### 模式选择

| 用户说法 | 模式 |
|----------|------|
| 下载 / 导出 md（默认） | `inline_https` |
| 图片也下载 / 离线 / assets | `local_assets` |
| 意图不明 | 问一句：「临时链接（约 10 分钟）还是图片保存到本地文件夹？」 |

---

## 工具速查

| 操作 | 工具 |
|------|------|
| 上传附件 | `upload_attachment`（`url` 或 `content_base64`）→ `object_id` |
| 写最终正文 | `upload_replace_file`（UTF-8 Base64） |
| 新建占位 | `upload_new_file`（`name=*.md`）或 `create_file_with_content`（`file_extension=md`） |
| 读源码 | `read_file`（在线 Markdown 不传 `format`） |
| 取附件临时链 | `download_attachment`（`attachment_id` = `kdocsmd://` 后的 object_id）→ **`download_url`** |
