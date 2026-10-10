---
name: kdocs
description: "操作金山文档（WPS 云文档 / Kdocs / 365.kdocs.cn / www.kdocs.cn）云文档的官方 Skill。核心能力覆盖云端新建、读取、编辑、搜索、分享、整理在线文档（智能文档、Word、Excel、PDF、PPT、演示文稿、智能表格、多维表格）及个人知识库。当用户的任务涉及云文档操作时使用，包括但不限于：写周报/日报/工作汇报、处理合同/发票、创建报名表/登记表、网页剪藏、接龙转表格、信息收集、文档总结与内容生成、改写仿写、翻译、AI PPT生成、PDF拆分导出、标签分类归档、收藏管理、碎片笔记整理、表格美化、回收站还原、知识库管理。"
homepage: https://www.kdocs.cn/latest
version: 2.7.1
metadata: {"requires":{"bins":["kdocs-cli"],"cliHelp":"kdocs-cli --help"},"openclaw":{"category":"kdocs","tokenUrl":"https://www.kdocs.cn/latest","emoji":"📝","keywords":["金山文档","金山表格","金山收藏","WPS","WPS文档","云文档","在线文档","kdocs","WPS云文档","接龙转表格","接龙","群接龙","报名表","信息收集","收集表","登记表","网页剪藏","剪藏","保存网页","网页保存到文档","保存文章","收藏文章","总结","帮我总结","帮我整理","帮我写","帮我翻译","帮我做PPT","翻译文档 - 做PPT - 生成PPT - 培训课件 - 方案展示 - 项目展示","文档总结","内容生成","改写","仿写","翻译","文档翻译","PPT","演示文稿","幻灯片","PDF","拆分PDF","导出PDF","Word","Excel","表格","Markdown","碎片整理","笔记整理","表格优化","文档处理","文件处理","办公助手","文档助手","周报","日报","工作汇报","合同","发票"]},"file_types":["pdf","doc","docx","xlsx","xls","pptx","ppt","otl","ksheet","dbt","form","jpg","jpeg","png","bmp","gif","webp","url","md","txt","html"],"category":"productivity"}
---

# 金山文档 CLI Skill 使用指南

金山文档 CLI Skill 提供了一套完整的在线文档操作工具，通过 `kdocs-cli` 命令行工具与金山文档 API 交互。支持创建、查询、读取、编辑、分享、移动多种类型的在线文档。


## 严格规则

### 禁止（NEVER）

- 禁止将 Token 明文出现在对话、日志、命令输出、代码注释或任何文件中；Token 仅允许通过 `kdocs-cli auth set-token` 或 `kdocs-cli auth login` 保存到系统密钥链
- 上传写入等接口需传入的 `content_base64` 可能非常大（编码后 >1 MB），禁止在对话中逐 token 生成 Base64 字符串，用脚本完成文件读取、编码和传参
- 权限不足时禁止重试或绕过，立即告知用户无权限

### 必须（MUST）

- 不可逆操作（delete/close 类）执行前必须向用户确认
- 创建云文档文件并验证通过后，必须向用户展示可访问链接。若响应包含 `data.link_url` 则直接展示；若响应无链接时，调用 `get_file_link` 获取并展示。
- 除 SKILL.md 已给出最小可用示例的主路径工具外，其他工具调用前必须先阅读对应的 `references/` 详细参考文档；参数细节（类型、可选值、约束）以工具参考文档为准
- 满足「问题反馈」章节触发条件时，必须运行 feedback 命令并展示完整反馈链接，并遵守该章节的防重复规则；禁止仅口头告知联系客服或仅以「暂不支持」结束对话
- 参考文档按本技能总览或索引中的文件路径读取，不猜文件名、不把目录当文件、不套用其他技能的路径。路径报错时先核对索引。

---

## 保持最新版本
何时触发：**首次使用** Skill / **距上次自检 >24h** / **收到 `unknown action` 或 `unknown service` 错误**。

1. **CLI 版本**：`kdocs-cli version` — 若命令不存在则按下方「工具安装与认证」安装；若低于本文件 frontmatter `version`，运行 `kdocs-cli upgrade -y`（自动备份旧版本，失败可 `kdocs-cli upgrade --rollback`）
2. **Skill 版本**：若本文件 `version` 低于 `kdocs-cli version`，运行 `kdocs-cli call check_skill_update version=2.7.1 skill_name=kdocs`，若返回 `update_available: true`，从 `instruction` 中提取 zip 下载链接，下载解压替换当前 Skill 目录

若 upgrade 和 rollback 均失败，在本 Skill 目录下重新运行安装脚本（`bash setup.sh` / `powershell setup.ps1` / `node setup.cjs`，脚本位于 `scripts/`）可从 CDN 重新安装。若仍无法更新，以 `kdocs-cli --help` 实际支持的工具集为准。

---

## 工具安装与认证

| 操作 | 命令 |
|------|------|
| 安装 | `bash scripts/setup.sh` / `powershell scripts/setup.ps1` / `node scripts/setup.cjs` |
| 认证 | 用户已提供 Token: `kdocs-cli auth set-token "<token>"` · 无 Token: `kdocs-cli auth login` |

login 失败时的手动获取流程、`auth status` 诊断、`auth logout` 退出等详见 `references/auth.md`。

---

## 调用格式

kdocs-cli <service> <action> [参数]

### 参数传递

| 参数特征 | 推荐方式 | 示例 |
|----------|----------|------|
| 简单值（无中文） | key=value | `kdocs-cli drive search-files keyword=test type=all` |
| 数组/对象，短 JSON | JSON 字符串 | `kdocs-cli sheet query-records '{"file_id":"xxx","filter":{}}'` |
| 数组/对象，或含中文/换行/>200 字符 | --file | `kdocs-cli otl insert-content --file payload.json` |
| 脚本流水线集成 | stdin | `node gen.js \| kdocs-cli otl insert-content -` |

- `--file` / stdin 输入必须是该工具的**完整 JSON 参数对象**
- 中文/多行参数**禁止** key=value（Windows/PowerShell 破坏 UTF-8 编码）
- 生成 JSON 文件用 Node.js/Python；**禁止** ConvertTo-Json（输出带 BOM）
- PowerShell 传 JSON 字符串须反斜杠转义：`'{\"key\":[\"val\"]}'`

> **--file 示例**：写入大段内容时，用脚本生成 JSON 文件再 `--file` 传入，操作完成后删除临时文件：
>
> ```javascript
> const fs = require('fs');
> fs.writeFileSync('payload.json', JSON.stringify({
>   file_id: "<file_id>",
>   content: fs.readFileSync('article.md', 'utf8'),
>   format: "markdown",
>   mode: "append"
> }), 'utf8');
> ```
> ```
> kdocs-cli otl insert-content --file payload.json --silent
> ```
> ```javascript
> // 操作完成后清理临时文件
> fs.unlinkSync('payload.json');
> ```

**全局选项**：

| 选项 | 说明 |
|------|------|
| `--token <token>` | 一次性 Token（优先级最高，不持久化） |
| `--endpoint <url>` | 覆盖默认 endpoint |
| `--compact` | 输出紧凑 JSON |
| `--silent` | 仅输出 `data` 字段 |
| `--verbose` | 输出请求详情到 stderr |
| `--timeout <ms>` |  HTTP 请求超时（毫秒，默认 30000） |

**帮助**：`kdocs-cli --help`、`kdocs-cli <service> --help`、`kdocs-cli <service> <action> --help`

> **找不到命令？** 浏览 `--help` 时若发现预期的 service 或 action 不存在，先运行 `kdocs-cli upgrade -y` 升级到最新版本再重试。CLI 能力随版本持续扩展，未升级是命令缺失的首要原因。详见上方「保持最新版本」章节。

---

## 读写主路径

读用 `read_file`，写用 `create_file_with_content`。其他工具⚠️ **禁止跳过 reference 直接调用**（违反必导致参数错误），工具总览仅供路由选择，调用前必须读对应 `references/` 文档。

### 读：`read_file`

`read_file` 是正文读取主入口，定位参数 `url` / `link_id` / `file_id` 三选一：

```bash
kdocs-cli drive read-file url="https://www.kdocs.cn/l/xxx"
```

按已有定位信息选择一个参数即可：用户给完整链接传 `url`；已解析分享链接传 `link_id`；已拿到文件 ID 传 `file_id`。不要同时传多个定位参数。

首次请求返回 `status=pending` 时，携带该响应中的 `task_id` 和原读取参数再次请求，将返回文档全量内容：

```bash
kdocs-cli drive read-file file_id=file_xxx task_id=task_xxx
```

`task_id` 必须来自本次响应，禁止猜测或跨请求复用。

必须检查返回里的 `data.warnings`：表格可能只读默认工作表或首屏区域，warnings 会提示实际读取范围。

**智能文档（.otl）内嵌图片**：`read_file` 默认返回图片临时可下载 URL（约 10 分钟有效）；显式传 `enable_upload_medias=false` 或图片本身为空/不可转时，图片为 `![image]()` 空占位，不含可下载资源，不要据此调 `download_attachment`。

**在线 Markdown**：`read_file` 默认返回 Markdown 源码（含 `kdocsmd://` 图片协议）；`enable_upload_medias` **不会**把 `kdocsmd://` 转为 https。用户给链接 / file_id 直接读，仅有文件名时先 `search_files`。仅读取正文、不保存本地文件 → 见 `references/workflows/read-cloud-md.md`。

**在线 Markdown 图片**：

| 意图 | 怎么做 |
|------|--------|
| 本地文件上传为在线 Markdown，图片转为 `kdocsmd://` | `references/workflows/upload-md-raw.md` |
| 覆盖已有在线 Markdown，并转存图片 | `references/workflows/replace-cloud-md.md` |
| 对话/粘贴正文新建在线 Markdown，并转存图片 | `references/workflows/create-cloud-md.md` |
| 导出到本地（默认 https 临时链，可选 assets） | `references/workflows/download-cloud-md.md` |

规则详见 `references/markdown.md`。

**otl/docx/pdf → Markdown 文本**：`read_file(format=markdown)` 做格式转换提取；在线 Markdown 读源码不传 format（默认 markdown）。

### 写：`create_file_with_content`

`create_file_with_content` 是“新建并写入内容”的主路径。目标目录明确时只传 `parent_id`（文件夹 id）；未指定目录时盘和目录都可省略（默认个人云盘根）。

```bash
kdocs-cli drive create-file-with-content --file payload.json
```

`payload.json`：

```json
{
  "name": "周报",
  "file_extension": "otl",
  "content": "# 周报\n\n这里是正文"
}
```

| 后缀 | 关键参数 |
|------|----------|
| `.otl` `.docx` `.md` `.pdf` | `name` + `file_extension` + `content` |
| `.xlsx` `.ksheet` | `name` + `file_extension` + `sheet_name` + `rangeData` |
| `.dbt` | `name` + `file_extension` + `sheet_name` + `fields` + `records` |

创建表格时，`rangeData` 为对象数组，`formula` 为二维数组。`sheet.update_range_data` 的 `rangeData` 每项需包含 `opType` 和四个 camelCase 坐标，写值或公式时 `formula` 为字符串；两种结构不要混用。

`payload.json`：

```json
{
  "name": "台账",
  "file_extension": "xlsx",
  "sheet_name": "Sheet1",
  "rangeData": [{
    "row_from": 0,
    "row_to": 0,
    "col_from": 0,
    "col_to": 1,
    "formula": [["姓名", "部门"]]
  }]
}
```

表格、多维表结构不确定时，先读 `create_file_with_content` reference。写入成功后优先展示返回里的 `data.link_url`。

**本地 Markdown 上传**：用户仅说「上传 md」且未要求保留 `.md` 时，走 `create_file_with_content`（`name` 不带后缀，默认 `file_extension=otl`；步骤见 `references/workflows/md-upload-to-otl.md`）；明确要求保留 `.md` 时走「本地文件上传为在线 Markdown」（`references/workflows/upload-md-raw.md`，含图片转 `kdocsmd://`）。对话内正文新建在线 Markdown → `references/workflows/create-cloud-md.md`。总规则见 `references/markdown.md`。

**参考文档（按需读取）**

需要补充参数说明或排查调用错误时，可读取：

- `read_file`：`references/drive/read_and_download.md`
- `create_file_with_content`：`references/drive/create_and_upload.md`

按上述完整路径读取参考文档，不要猜测文件名。


以下工具不可逆，调用前必须向用户确认（详细约束见各工具参考文档的「操作约束」区）：

`otl.block_delete`、`dbsheet.delete_sheet`、`kwiki.close_knowledge_view`、`sheet.delete_sheets`、`sheet.delete_range_data`、`dbsheet.delete_view`、`dbsheet.delete_fields`、`cancel_share`、`kwiki.delete_item`、`sheet.delete_protection_ranges`、`dbsheet.delete_records`、`sheet.delete_data_validations`、`cancel_collaborator_permissions`、`sheet.delete_conditional_format_rules`、`sheet.delete_float_images`、`sheet.delete_filters`、`dbsheet.sheet_batch_delete`、`dbsheet.dashboard_delete`、`sheet.delete_pivot_table`、`dbsheet.dashboard_clear`、`dbsheet.permission_delete_roles_async`、`dbsheet.innerdoc_block_delete`、`dbsheet.innerdoc_block_batch_delete`

---

## 能力范围

### 操作域路由

Agent 首先判定用户请求的操作域：

> 进入「局部更新」「类型专属能力」且后缀/类型未知时：先按 `references/file-locating-guide.md` 确认类型，再打开对应类型 reference；勿猜测调用。读取/创建/文件管理不必先调。

| 操作域 | 触发场景 | 路由 |
|--------|---------|------|
| 创建/写入 | 新建并写入、上传本地文件、新建空白文档 | 主路径见上方「读写主路径」；上传本地文件、新建空白文档见 `references/drive.md` |
| 局部更新 | 改块/改段/改单元格，已有目标文档上的修改 | 按「支持的文档类型」→ 对应 reference 中的写入/更新类工具 |
| 类型专属能力 | 条件格式、导出转换、翻译、PDF 拆分、幻灯片主题、数据校验 | 按「支持的文档类型」→ 对应 reference 中的专属功能章节 |
| 读取 | 读取/提取/导出文档内容 | 正文：`read_file`（`url`/`link_id`/`file_id`，见 `references/drive/read_and_download.md`）；**.otl 内嵌图片** → `references/otl.md`「常用工作流 → 读取智能文档内嵌图片」；否则按「定位文件」 |
| 定位文件 | 搜索/按链接查文件标识或属性/浏览目录 | 详见 `references/file-locating-guide.md`；看我的云盘 → `references/drive/read_and_download.md`（`drive.list_my_files`） |
| 文件管理 | 移动/重命名/分享/标签/收藏/回收站/评论/文档库/历史版本/另存为 | → `references/drive.md` |
| AI 生成 | AI 做PPT/生成演示文稿 | → `references/aippt.md` |
| 知识库 | 知识库空间/导入/整理 | → `references/kwiki.md` |

### 支持的文档类型

| 类型 | 别名 | 文件后缀 | 说明 | 详细参考 |
|------|------|----------|------|----------|
| **智能文档** 首选 | ap | .otl | 排版美观，支持丰富组件 | `references/otl.md` — 页面、文本、标题、待办等元素操作 |
| 在线 Markdown | md | .md | 源码 Markdown，图片为 kdocsmd:// 协议 | `references/markdown.md` — 上传时图片转存、导出到本地（临时 https 或本地 assets） |
| 表格 | et / Excel | .xlsx | 数据表格专用 | `references/sheet.md` — 工作表管理、范围数据获取、批量更新 |
| PDF文档 | pdf | .pdf | PDF 文档专用 | `references/pdf.md` — PDF 创建与内容读取 |
| 文字文档 | wps / Word | .docx | 传统格式 | `references/wps.md` — Word 文档创建与内容操作 |
| 演示文稿 | wpp | .pptx | PPT 文档专用 | `references/wpp.md` — 幻灯片主题字体和配色设置、下载和导出 |
| 智能表格 | as | .ksheet | 结构化表格，支持多视图、字段管理 | `references/sheet.md` — 工作表管理、范围数据获取、批量更新 |
| 多维表格 | db / dbsheet | .dbt | 多数据表、丰富字段类型与视图（表格/看板/甘特等） | `references/dbsheet.md` — 支持数据表/视图/字段/记录的完整增删改查，含表单视图、父子记录、分享协作、高级权限与 Webhook |
| 智能表单 | form | .form | 轻量表单草稿创建、题目配置、发布与查询 | `references/form.md` — 草稿创建/更新/发布与表单信息查询 |

### 高频流程指引

#### 创建/写入

| 用户意图 | 工具 | 适用后缀 / 说明 |
|----------|------|----------|
| 仅需空白文档 | `create_empty_file` | .doc .docx .otl .dbt（默认列） .xlsx .xls .ksheet .pptx .ppt |
| 已有正文或表格数据要写入 | `create_file_with_content` | .otl .docx .pdf .xlsx .ksheet .dbt |
| 新建多维表且须自定义列 | `create_file_with_content` | .dbt（无业务数据也须 fields+records） |
| 本地 Markdown 新建为办公文档 | `create_file_with_content` | .otl .docx .pdf；原样 .md 才用 `upload_new_file` |
| 通过上传本地文件新建云文档 | `upload_new_file` | .doc .docx .xls .xlsx .ppt .pptx .pdf .md .txt .html .zip .png .jpg .jpeg .csv .json .dps .et .wps .gif |
| AI 生成 PPT | `aippt.execute` | .pptx |

后缀不确定时默认 `.otl`。指定文件夹时先按 `references/file-locating-guide.md` 取 `drive_id`、`parent_id`。

选定工具后，阅读 `references/drive/create_and_upload.md` 对应章节获取参数约束（`aippt.execute` 见 `references/aippt.md`）。

#### 搜索定位文档

工具说明：`search_files(keyword="关键词", page_size=10)` 即可搜索（`type` 可省略，默认 `all`）；获取 `file_id`、`drive_id` 供后续链路使用。
`type` 为搜索维度（file_name/content/all），筛选文件夹/文件请用 `file_type`。
详细参数与返回结构见 `references/drive/search.md`。

### 更多操作流程

| 流程 | 说明 | 详细参考 |
|------|------|---------|
| 读取多维表云文档元信息 | 从多维表记录中提取云文档字段的 file_id，再调用 get_file_info 获取元信息（文件名、大小、类型、修改时间等） | `references/workflows/read-cloud-doc-meta.md` |
| AI 生成演示文稿（全文） | aippt.execute 单接口全文生成链路：支持 html（两次调用 + follow_up）和 basic（一次调用，经典简约模式）两种模式，覆盖主题/文档场景 | `references/workflows/aippt-full-text.md` |
| AI 单页生成幻灯片 | aippt.execute 单接口单页生成幻灯片：HTML 布局模式，一次调用完成，可通过 wpp.import_slides 插入到已有演示文稿 | `references/workflows/aippt-single-page.md` |
| 网页剪藏 | 抓取网页内容并自动保存为智能文档 | `references/workflows/web-scrape.md` |
| 搜索-读取-汇报撰写 | 搜索多份文档、提取信息、汇总撰写新报告 | `references/workflows/search-read-report.md` |
| 定期读取与播报 | 定期读取指定文档，提取关键信息生成摘要 | `references/workflows/periodic-read-summary.md` |
| 文档内容转 Markdown | 将云文档内容提取为 Markdown 格式文本 | `references/workflows/read-to-markdown.md` |
| 智能分类整理 | 列出目录，按内容或指定维度分类创建文件夹并归档 | `references/workflows/smart-classify.md` |
| 读取在线 Markdown | 读取在线 Markdown 文件正文（Markdown 源码） | `references/workflows/read-cloud-md.md` |
| 精准搜索与风险排查 | 在特定目录批量搜索文档，逐一读取分析，汇总到新文档 | `references/workflows/precise-search-analysis.md` |
| 在线 Markdown 导出到本地 | 将在线 Markdown 导出为本地文件；图片默认内联 HTTPS 临时链，也可保存到本地 assets 目录 | `references/workflows/download-cloud-md.md` |
| 本地 Markdown 上传为在线文档（保留 .md） | 将本地 .md 上传为在线 Markdown，并把图片转存为 kdocsmd://（不转为 otl/docx） | `references/workflows/upload-md-raw.md` |
| 对话内正文新建在线 Markdown | 将对话中的 Markdown 正文新建为在线 Markdown，并把图片转存为 kdocsmd:// | `references/workflows/create-cloud-md.md` |
| 本地 Markdown 上传为办公文档 | 本地 .md 转为云盘办公文档（默认 otl；亦可 docx/pdf） | `references/workflows/md-upload-to-otl.md` |
| 本地 Markdown 覆盖在线 Markdown | 用本地 .md 全量覆盖已有在线 Markdown，并把图片转存为 kdocsmd:// | `references/workflows/replace-cloud-md.md` |
| 在线 Markdown 总结与问答 | 读取在线 Markdown 正文后，总结要点或按用户问题作答 | `references/workflows/summarize-cloud-md.md` |
| 云文档导入幻灯片 | 将外部 PPTX 文件中的指定幻灯片导入到已有演示文稿中 | `references/workflows/import-slides.md` |
| 接龙转表格 | 识别接龙文本内容，自动提取并转为在线表格 | `references/workflows/jielong-to-table.md` |
| 跨表字段回填（按主键匹配） | 两张表格按主键列（如订单号、企业ID、SKU）匹配，将源表指定列写入目标表对应列；须先读表头按列名定位，写后按列名回读验证 | `references/workflows/sheet-cross-fill.md` |
| 表单收集 | 根据用户需求设计并创建智能表单，发布后生成填写链接 | `references/workflows/form-collection.md` |
| 知识智能整理 | 对知识库中的零散内容进行智能化整理和结构化重组 | `references/workflows/knowledge-format.md` |
| 知识一键存入 | 将各类内容（网页、文件、文本）一键保存到知识库 | `references/workflows/knowledge-save.md` |
| 表格美化与数据规范 | 读取表格数据，进行格式美化、数据规范化和样式调整，并通过条件格式、数据校验、区域权限固化规则 | `references/workflows/table-beautify.md` |

---

## 错误速查

| 错误特征 | 原因 | 处理方式 |
|----------|------|----------|
| `400006` / 鉴权失败 | Token 过期或未配置 | 运行 `kdocs-cli auth login` 重新登录，或 `kdocs-cli auth set-token <token>` 重新设置 |
| `429001` / 限频 | 请求过于频繁，响应含**限频恢复时间** | 立即停止命令调用，直到达到恢复时间；禁止立即重试、换参、换子命令连续请求 |
| `429002` / 熔断 | 多因短时间内连续触发 `429001` ，响应含**熔断持续时间** | 熔断时长内零请求，期满再试；重新规划任务避免请求过频 |
| `403` / 权限不足 / `无权访问` / `forbidden` | 当前凭据对目标文档、目录或资源无操作权限 | 停止操作，禁止重试或尝试其他接口绕过；告知用户当前账号无权限，并建议联系文档所有者开通权限、确认分享链接权限，或切换到有权限的账号 |
| `unknown action` / `unknown service` | CLI 版本过旧或名称拼写错误 | 先运行 `kdocs-cli upgrade` 升级到最新版本；仍报错再运行 `kdocs-cli <service> --help` 确认可用命令 |
| 搜索无结果 | 关键词过精确 / 索引延迟 | 缩短关键词 / 等待 3-5 秒重试 |
| 读取内容为空 | 文件无内容或格式不支持 | 确认文件非空且后缀正确 |
| 第三方服务错误：任务不存在 | 使用了无效、跨请求或已完成的 `task_id` | 前一次已返回完整内容时直接使用已有结果；否则丢弃旧 `task_id`，按原文件定位参数重新读取 |
| 创建文件失败 | 文件名后缀不正确 | 检查后缀：`.otl` / `.docx` / `.xlsx` / `.ksheet` / `.dbt` / `.pdf` / `.pptx` |
| 移动文件失败 | 目标文件夹不存在 | 先搜索确认或创建文件夹 |
| `Client.Timeout exceeded while awaiting headers` | 服务端处理或排队时间超过 CLI HTTP 超时，常见于上传、导出、AI 生成、格式转换、大文件读取等慢操作 | 确认工具幂等性后重试 1 次，并显式设置 `--timeout` 值（如 `--timeout=120000`）；写入/创建类工具重试前先查询结果，避免重复创建 |
| `conflict` / `lock` / 并发写入冲突 | 多个写操作同时修改同一资源（知识库节点、多维表记录等）导致锁竞争 | 指数退避重试（2s → 4s → 8s，最多 3 次）；批量写入场景改为串行逐条执行；详见 kwiki / dbsheet 各 reference「错误速查表」 |
| HTTP 5xx | 服务端故障 | 等 3 秒重试 1 次 |
| 验证不通过（回读值与预期不符） | 写入未生效或延迟 | 等 2 秒重新验证，仍不通过则报告用户 |
| `setup.sh` 执行失败 / 安装报错 | 当前版本可能已不兼容 | 执行上方「保持最新版本」流程 |
| CLI 接口返回未知错误码（非 5xx、非 400006、非 429001/429002、非工具不存在） | Skill 版本过旧导致接口不兼容 | 执行上方「保持最新版本」流程；仍失败到 `kdocs-cli feedback` |
| 错误信息含 `version`、`incompatible`、`not_supported`、`deprecated` 等版本关键词 | Skill 或 API 版本不兼容 | 执行上方「保持最新版本」流程；仍失败到 `kdocs-cli feedback` |
| 工具调用失败且原因不明 | 可能是 Skill 版本过旧 | 执行上方「保持最新版本」流程；仍失败到 `kdocs-cli feedback` |
| 上述处理方式均已尝试仍无法解决 | 未知问题 | 仍失败则 `kdocs-cli feedback` |
| 工具调用失败需判断是否可重试 | 不同工具幂等性不同 | 查看该工具参考文档「操作约束」区的幂等性说明，幂等工具可安全重试，非幂等工具须先确认状态 |

---

## 问题反馈

满足以下**任一**条件时 MUST 提供反馈链接。**可界定异常**无需先穷尽错误速查表。

**可界定异常（必出链接）：** 功能缺少、工具不存在、格式/类型不支持、接口/参数不支持、结果不达标（回读验证失败且无法修正）、权限/账户限制（告知原因后）、搜索/查询无结果（已调整检索条件至少 1 次且任务仍无法继续）。

**错误与兜底：** 当前错误处理步骤（含有限重试）已完成仍失败。

**用户主动：** 要求反馈、投诉、报告问题或提交功能需求。

**引导语：** Bug/异常用「欢迎反馈具体问题」；**功能需求**用「欢迎通过反馈链接告诉我们你希望实现什么」。功能缺少或不支持时禁止仅以「暂不支持」结束，必须附带反馈链接。

运行 `kdocs-cli feedback`（可选 `--reason <trigger_reason>` 用于统计），将 stdout 完整 URL 展示给用户，告知「点击即可打开反馈页面」。

复杂任务部分成功且用户可能有改进建议时，可在回复末尾附带反馈链接；日常简单成功不必每次展示。

**防重复（MUST 遵守）：**
- 每条回复最多展示 1 个反馈链接；已展示则同条回复内不再重复获取链接。
- 同一会话内，同一根因（同一类阻断，如相同的不支持格式、相同的缺失能力）只须出链接 1 次；后续同类情况用文字说明，可提示用户沿用已给出的链接。
- 用户已表示不需要反馈或已知悉后，本会话不再主动贴链接；除非用户再次要求或出现不同根因的新阻断。
- 「搜索/查询无结果」须在已调整检索条件至少 1 次且仍无法继续时，才计入 MUST。

---

## 安全约束

- 凭据由 `kdocs-cli` 系统密钥链管理，Skill 自身不存储、不记录
- 无状态代理，不缓存任何文档内容或业务数据
- 仅在用户主动发起操作时调用对应 API

