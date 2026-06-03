---
name: kdocs
description: "操作金山文档（WPS 云文档 / Kdocs / 365.kdocs.cn / www.kdocs.cn）云文档的官方 Skill。核心能力覆盖云端新建、读取、编辑、搜索、分享、整理在线文档（智能文档、Word、Excel、PDF、PPT、演示文稿、智能表格、多维表格）及个人知识库。当用户的任务涉及云文档操作时使用，包括但不限于：写周报/日报/工作汇报、处理合同/发票、创建报名表/登记表、网页剪藏、接龙转表格、信息收集、文档总结与内容生成、改写仿写、翻译、AI PPT生成、PDF拆分导出、标签分类归档、收藏管理、碎片笔记整理、表格美化、回收站还原、知识库管理。"
homepage: https://www.kdocs.cn/latest
version: 2.5.7
metadata: {"requires":{"bins":["kdocs-cli"],"cliHelp":"kdocs-cli --help"},"openclaw":{"category":"kdocs","tokenUrl":"https://www.kdocs.cn/latest","emoji":"📝","keywords":["金山文档","金山表格","金山收藏","WPS","WPS文档","云文档","在线文档","kdocs","WPS云文档","接龙转表格","接龙","群接龙","报名表","信息收集","收集表","登记表","网页剪藏","剪藏","保存网页","网页保存到文档","保存文章","收藏文章","总结","帮我总结","帮我整理","帮我写","帮我翻译","帮我做PPT","翻译文档 - 做PPT - 生成PPT - 培训课件 - 方案展示 - 项目展示","文档总结","内容生成","改写","仿写","翻译","文档翻译","PPT","演示文稿","幻灯片","PDF","拆分PDF","导出PDF","Word","Excel","表格","Markdown","碎片整理","笔记整理","表格优化","文档处理","文件处理","办公助手","文档助手","周报","日报","工作汇报","合同","发票"]},"file_types":["pdf","doc","docx","xlsx","xls","pptx","ppt","otl","ksheet","dbt","form","jpg","jpeg","png","bmp","gif","webp","url","md","txt","html"],"category":"productivity"}
---

# 金山文档 CLI Skill 使用指南

金山文档 CLI Skill 提供了一套完整的在线文档操作工具，通过 `kdocs-cli` 命令行工具与金山文档 API 交互。支持创建、查询、读取、编辑、分享、移动多种类型的在线文档。


## 严格规则

### 禁止（NEVER）

- 禁止将 Token 明文出现在对话、日志、命令输出、代码注释或任何文件中；Token 仅允许通过 `kdocs-cli auth set-token` 或 `kdocs-cli auth login` 保存到系统密钥链
- 上传写入等接口需传入的 `content_base64` 可能非常大（编码后 >1 MB），禁止在对话中逐 token 生成 Base64 字符串，用脚本完成文件读取、编码和传参

### 必须（MUST）

- 不可逆操作（delete/close 类）执行前必须向用户确认
- 写操作完成后必须用独立读取请求验证实际结果（不信任 `code: 0`）
- 创建云文档文件并验证通过后，必须向用户展示可访问链接。若响应包含 `data.link_url` 则直接展示；若响应无链接时，调用 `get_file_link` 获取并展示。
- 调用工具前必须先阅读对应的 `references/` 详细参考文档，禁止仅凭指南（guide）的概要说明直接拼装调用；参数细节（类型、可选值、约束）以工具参考文档为准

---

## 保持最新版本
何时触发：**首次使用** Skill / **距上次自检 >24h** / **收到 `unknown action` 或 `unknown service` 错误**。

1. **CLI 版本**：`kdocs-cli version` — 若命令不存在则按下方「工具安装与认证」安装；若低于本文件 frontmatter `version`，运行 `kdocs-cli upgrade -y`（自动备份旧版本，失败可 `kdocs-cli upgrade --rollback`）
2. **Skill 版本**：若本文件 `version` 低于 `kdocs-cli version`，运行 `kdocs-cli call check_skill_update version=<本文件version>`，若返回 `update_available: true`，从 `instruction` 中提取 zip 下载链接（格式 `https://...kdocs.zip`），下载解压替换当前 Skill 目录

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


以下工具不可逆，调用前必须向用户确认（详细约束见各工具参考文档的「操作约束」区）：

`otl.block_delete`、`dbsheet.delete_sheet`、`kwiki.close_knowledge_view`、`sheet.delete_sheets`、`sheet.delete_range_data`、`dbsheet.delete_view`、`dbsheet.delete_fields`、`cancel_share`、`kwiki.delete_item`、`sheet.delete_protection_ranges`、`dbsheet.delete_records`、`sheet.delete_data_validations`、`sheet.delete_conditional_format_rules`、`sheet.delete_float_images`、`sheet.delete_filters`、`dbsheet.sheet_batch_delete`、`dbsheet.permission_delete_roles_async`

---

## 能力范围

### 操作域路由

Agent 首先判定用户请求的操作域：

| 操作域 | 触发场景 | 路由 |
|--------|---------|------|
| 创建/写入 | 新建并写入、上传本地文件、新建空白文档 | **见下方「创建/写入」** |
| 局部更新 | 改块/改段/改单元格，已有目标文档上的修改 | 按「支持的文档类型」→ 对应 reference 中的写入/更新类工具 |
| 类型专属能力 | 条件格式、导出转换、翻译、PDF 拆分、幻灯片主题、数据校验 | 按「支持的文档类型」→ 对应 reference 中的专属功能章节 | 读取 | 读取/提取/导出文档内容 | `read_file`（传 url 或 file_id，详见 `references/drive/read_and_download.md`）；没有则先「定位文件」 |
| 读取 | 读取/提取/导出文档内容 | `read_file`（传 url 或 file_id，详见 `references/drive/read_and_download.md`）；没有则先「定位文件」 |
| 定位文件 | 搜索/按链接找文件/浏览目录 | **必读** `references/file-locating-guide.md` |
| 文件管理 | 移动/重命名/分享/标签/收藏/回收站 | → `references/drive.md` |
| AI 生成 | AI 做PPT/生成演示文稿 | → `references/aippt.md` |
| 知识库 | 知识库空间/导入/整理 | → `references/kwiki.md` |

### 支持的文档类型

| 类型 | 别名 | 文件后缀 | 说明 | 详细参考 |
|------|------|----------|------|----------|
| **智能文档** 首选 | ap | .otl | 排版美观，支持丰富组件 | `references/otl.md` — 页面、文本、标题、待办等元素操作 |
| 表格 | et / Excel | .xlsx | 数据表格专用 | `references/sheet.md` — 工作表管理、范围数据获取、批量更新 |
| PDF文档 | pdf | .pdf | PDF 文档专用 | `references/pdf.md` — PDF 创建与内容读取 |
| 文字文档 | wps / Word | .docx | 传统格式 | `references/wps.md` — Word 文档创建与内容操作 |
| 演示文稿 | wpp | .pptx | PPT 文档专用 | `references/wpp.md` — 幻灯片主题字体和配色设置、下载和导出 |
| 智能表格 | as | .ksheet | 结构化表格，支持多视图、字段管理 | `references/sheet.md` — 工作表管理、范围数据获取、批量更新 |
| 多维表格 | db / dbsheet | .dbt | 多数据表、丰富字段类型与视图（表格/看板/甘特等） | `references/dbsheet.md` — 支持数据表/视图/字段/记录的完整增删改查，含表单视图、父子记录、分享协作、高级权限与 Webhook |
| 智能表单 | form | .form | 轻量表单草稿创建、题目配置、发布与查询 | `references/form.md` — 草稿创建/更新/发布与表单信息查询 |

### 高频流程指引

#### 创建/写入

| 用户意图 | 工具 | 适用后缀 |
|----------|------|----------|
| 对话中已有要写的内容 | `create_file_with_content` | .otl .docx .pdf .xlsx .ksheet .dbt |
| 上传本地已有文件 | `upload_file` | .doc .docx .xls .xlsx .ppt .pptx .pdf .md .txt |
| 新建空白文档（不写内容） | `create_file` | .doc .docx .otl .dbt .xlsx .xls .ksheet .pptx .ppt |
| AI 生成 PPT | `aippt.execute` | .pptx |

后缀不确定时默认 `.otl`。指定文件夹时先按 `references/file-locating-guide.md` 取 `drive_id`、`parent_id`。

选定工具后，阅读 `references/drive/create_and_upload.md` 对应章节获取参数约束（`aippt.execute` 见 `references/aippt.md`）。

#### 搜索定位文档

工具说明：`search_files(keyword="关键词", type="all", page_size=20)`，获取 `file_id`、`drive_id` 供后续链路使用。
详细参数与返回结构见 `references/drive/search.md`。

### 更多操作流程

| 流程 | 说明 | 详细参考 |
|------|------|---------|
| AI 生成演示文稿（全文） | aippt.execute 单接口全文生成链路：支持 html（两次调用 + follow_up）和 basic（一次调用，经典简约模式）两种模式，覆盖主题/文档场景 | `references/workflows/aippt-full-text.md` |
| AI 单页生成幻灯片 | aippt.execute 单接口单页生成幻灯片：HTML 布局模式，一次调用完成，可通过 wpp.import_slides 插入到已有演示文稿 | `references/workflows/aippt-single-page.md` |
| 网页剪藏 | 抓取网页内容并自动保存为智能文档 | `references/workflows/web-scrape.md` |
| 搜索-读取-汇报撰写 | 搜索多份文档、提取信息、汇总撰写新报告 | `references/workflows/search-read-report.md` |
| 定期读取与播报 | 定期读取指定文档，提取关键信息生成摘要 | `references/workflows/periodic-read-summary.md` |
| 智能分类整理 | 列出目录，按内容或指定维度分类创建文件夹并归档 | `references/workflows/smart-classify.md` |
| 精准搜索与风险排查 | 在特定目录批量搜索文档，逐一读取分析，汇总到新文档 | `references/workflows/precise-search-analysis.md` |
| 云文档导入幻灯片 | 将外部 PPTX 文件中的指定幻灯片导入到已有演示文稿中 | `references/workflows/import-slides.md` |
| 接龙转表格 | 识别接龙文本内容，自动提取并转为在线表格 | `references/workflows/jielong-to-table.md` |
| 信息收集表单生成 | 根据用户需求自动设计并创建信息收集表格 | `references/workflows/form-generator.md` |
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
| `unknown action` / `unknown service` | CLI 版本过旧或名称拼写错误 | 先运行 `kdocs-cli upgrade` 升级到最新版本；仍报错再运行 `kdocs-cli <service> --help` 确认可用命令 |
| 搜索无结果 | 关键词过精确 / 索引延迟 | 缩短关键词 / 等待 3-5 秒重试 |
| 读取内容为空 | 文件无内容或格式不支持 | 确认文件非空且后缀正确 |
| 创建文件失败 | 文件名后缀不正确 | 检查后缀：`.otl` / `.docx` / `.xlsx` / `.ksheet` / `.dbt` / `.pdf` / `.pptx` |
| 移动文件失败 | 目标文件夹不存在 | 先搜索确认或创建文件夹 |
| `Client.Timeout exceeded while awaiting headers` | 服务端处理或排队时间超过 CLI HTTP 超时，常见于上传、导出、AI 生成、格式转换、大文件读取等慢操作 | 确认工具幂等性后重试 1 次，并显式设置 `--timeout` 值（如 `--timeout=120000`）；写入/创建类工具重试前先查询结果，避免重复创建 |
| `conflict` / `lock` / 并发写入冲突 | 多个写操作同时修改同一资源（知识库节点、多维表记录等）导致锁竞争 | 指数退避重试（2s → 4s → 8s，最多 3 次）；批量写入场景改为串行逐条执行；详见 kwiki / dbsheet 各 reference「错误速查表」 |
| HTTP 5xx | 服务端故障 | 等 3 秒重试 1 次 |
| 验证不通过（回读值与预期不符） | 写入未生效或延迟 | 等 2 秒重新验证，仍不通过则报告用户 |
| `setup.sh` 执行失败 / 安装报错 | 当前版本可能已不兼容 | 执行上方「保持最新版本」流程 |
| CLI 接口返回未知错误码（非 5xx、非 400006、非 429001/429002、非工具不存在） | Skill 版本过旧导致接口不兼容 | 执行上方「保持最新版本」流程 |
| 错误信息含 `version`、`incompatible`、`not_supported`、`deprecated` 等版本关键词 | Skill 或 API 版本不兼容 | 执行上方「保持最新版本」流程 |
| 工具调用失败且原因不明 | 可能是 Skill 版本过旧 | 执行上方「保持最新版本」流程 |
| 上述处理方式均已尝试仍无法解决 | 未知问题 | 运行 `kdocs-cli feedback` 获取反馈链接，引导用户提交反馈 |
| 工具调用失败需判断是否可重试 | 不同工具幂等性不同 | 查看该工具参考文档「操作约束」区的幂等性说明，幂等工具可安全重试，非幂等工具须先确认状态 |

---

## 安全约束

- 凭据由 `kdocs-cli` 系统密钥链管理，Skill 自身不存储、不记录
- 无状态代理，不缓存任何文档内容或业务数据
- 仅在用户主动发起操作时调用对应 API


## 金山文档 Skill（kdocs skill）问题反馈

当满足以下任一条件时，生成反馈链接并提供给用户：

1. 错误速查表中的处理方式（重试、保持最新版本等）均已尝试但问题仍未解决
2. 用户主动要求反馈或投诉

运行 `kdocs-cli feedback` 获取反馈链接，将完整链接展示给用户并告知"点击即可打开反馈页面"，由用户决定是否打开。
