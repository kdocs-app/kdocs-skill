---
name: kdocs
description: "操作金山文档（WPS 云文档 / Kdocs / 365.kdocs.cn / www.kdocs.cn）云文档的官方 Skill。核心能力覆盖云端新建、读取、编辑、搜索、分享、整理在线文档（智能文档、Word、Excel、PDF、PPT、演示文稿、智能表格、多维表格）及个人知识库。当用户的任务涉及云文档操作时使用，包括但不限于：写周报/日报/工作汇报、处理合同/发票、创建报名表/登记表、网页剪藏、接龙转表格、信息收集、文档总结与内容生成、改写仿写、翻译、AI PPT生成、PDF拆分导出、标签分类归档、收藏管理、碎片笔记整理、表格美化、回收站还原、知识库管理。"
homepage: https://www.kdocs.cn/latest
version: 2.4.6
metadata: {"requires":{"bins":["kdocs-cli"],"cliHelp":"kdocs-cli --help"},"openclaw":{"category":"kdocs","tokenUrl":"https://www.kdocs.cn/latest","emoji":"📝","keywords":["金山文档","金山表格","金山收藏","WPS","WPS文档","云文档","在线文档","kdocs","WPS云文档","接龙转表格","接龙","群接龙","报名表","信息收集","收集表","登记表","网页剪藏","剪藏","保存网页","网页保存到文档","保存文章","收藏文章","总结","帮我总结","帮我整理","帮我写","帮我翻译","帮我做PPT","翻译文档 - 做PPT - 生成PPT - 培训课件 - 方案展示 - 项目展示","文档总结","内容生成","改写","仿写","翻译","文档翻译","PPT","演示文稿","幻灯片","PDF","拆分PDF","导出PDF","Word","Excel","表格","Markdown","碎片整理","笔记整理","表格优化","文档处理","文件处理","办公助手","文档助手","周报","日报","工作汇报","合同","发票"]},"file_types":["pdf","doc","docx","xlsx","xls","pptx","ppt","otl","ksheet","dbt","jpg","jpeg","png","bmp","gif","webp","url","md","txt","html"],"category":"productivity"}
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
- 创建文档并验证通过后，必须调用 `get_file_link` 获取链接并展示给用户

---

## 版本自检
何时触发：**首次使用** Skill / **距上次自检 >24h** / **收到 `unknown command` 或不兼容错误**。其他时刻无需重复执行。版本自检流程见 `references/version-check.md`。

---

## 工具安装

运行安装脚本，自动检测平台并下载对应二进制到全局 PATH 位置：

```bash
bash scripts/setup.sh          # Linux/macOS → ~/.local/bin/kdocs-cli
powershell scripts/setup.ps1    # Windows → %LOCALAPPDATA%\kdocs-cli\
node scripts/setup.cjs          # 任意平台（需 Node.js >= 18）
```

验证安装：

```bash
kdocs-cli version
```

**升级**

```bash
kdocs-cli upgrade              # 内置自升级到最新版本
kdocs-cli upgrade --check      # 仅检查新版本
kdocs-cli upgrade --rollback   # 回滚到上一版本
```

## 认证配置

### Token 设置（推荐）

当用户已提供 Token 或通过其他途径获取到 Token 时，使用 `auth set-token` 直接保存到系统密钥链：

```bash
kdocs-cli auth set-token <token>
```

Token 包含 `/`、`+`、`=` 等特殊字符时，使用 stdin 模式避免 shell 转义问题：

```bash
echo "<token>" | kdocs-cli auth set-token -
```

`set-token` 保存后会自动验证 Token 有效性，返回验证结果。

### Token 登录

无现成 Token 时，通过浏览器 OAuth 登录获取：

```bash
kdocs-cli auth login
```

登录成功后 Token 自动保存到系统密钥链，后续命令自动使用。

| 操作 | 说明 |
|------|------|
| 设置 Token | `kdocs-cli auth set-token <token>` — 直接保存到系统密钥链（推荐） |
| 浏览器登录 | `kdocs-cli auth login` — OAuth 登录，Token 自动保存到密钥链 |
| 查看状态 / 诊断 | `kdocs-cli auth status` — Token 来源、密钥链一致性、环境变量状态 |
| 退出登录 | `kdocs-cli auth logout` — 从密钥链移除 Token |
| 验证 | `kdocs-cli drive search-files keyword=test page_size=1` — 返回 `code: 0` 即认证成功 |
| 过期 | 收到错误码 `400006` 时，CLI 自动输出诊断信息，根据提示修复或使用 `auth set-token` 重新设置 |

#### 手动获取 Token（登录命令失败时的兜底方案）

当 `kdocs-cli auth login` 或 `get-token` 脚本因环境问题执行失败时，引导用户手动获取：

1. 用户在浏览器访问 https://www.kdocs.cn/latest （需已登录 WPS 账号）
2. 点击页面右上角个人头像旁的主菜单 → 选择「龙虾专属入口」→ 复制 Token
3. 用户将 Token 提供给 Agent
4. Agent 保存到密钥链：

```bash
kdocs-cli auth set-token <TOKEN>
```

> 收到用户 Token 后直接通过 `auth set-token` 保存。保存后自动验证（`code: 0` 即成功）。

---

## 调用格式

kdocs-cli <service> <action> [参数]

### 参数传递

| 参数特征 | 推荐方式 | 示例 |
|----------|----------|------|
| 简单值（无中文） | key=value | `kdocs-cli drive search-files keyword=test type=all` |
| 数组/对象，短 JSON | JSON 字符串 | `kdocs-cli sheet query-records '{"file_id":"xxx","filter":{}}'` |
| 数组/对象，或含中文/换行/>200 字符 | @file | `kdocs-cli otl insert-content @payload.json` |
| 脚本流水线集成 | stdin | `node gen.js \| kdocs-cli otl insert-content -` |

- @file / stdin 输入必须是该工具的**完整 JSON 参数对象**
- 中文/多行参数**禁止** key=value（Windows/PowerShell 破坏 UTF-8 编码）
- 生成 JSON 文件用 Node.js/Python；**禁止** ConvertTo-Json（输出带 BOM）
- PowerShell 传 JSON 字符串须反斜杠转义：`'{\"key\":[\"val\"]}'`

> **@file 示例**：写入大段内容时，用脚本生成 JSON 文件再 `@file.json` 传入：
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
> kdocs-cli otl insert-content @payload.json --silent
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


以下工具不可逆，调用前必须向用户确认（详细约束见各工具参考文档的「操作约束」区）：

`otl.block_delete`、`dbsheet.delete_sheet`、`kwiki.close_knowledge_view`、`sheet.delete_sheets`、`sheet.delete_range`、`dbsheet.delete_view`、`dbsheet.delete_fields`、`cancel_share`、`kwiki.delete_item`、`sheet.delete_protection_ranges`、`dbsheet.delete_records`、`sheet.delete_data_validations`、`sheet.delete_conditional_format_rules`、`sheet.delete_float_images`、`sheet.delete_filters`、`dbsheet.sheet_batch_delete`、`dbsheet.permission_delete_roles_async`

---

## 能力范围

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

### 通用工具总览

#### 文档创建与上传
| 工具 | 用途 |
|------|------|
| `create_file` | 在云盘下新建文件或文件夹 |
| `scrape_url` | 网页剪藏，抓取网页内容并自动保存为智能文档 |
| `scrape_progress` | 查询网页剪藏任务进度 |
| `upload_file` | 全量上传写入文件（更新已有 docx/pdf 或新建并上传本地文件） |

#### 文档读取与下载
| 工具 | 用途 |
|------|------|
| `list_files` | 获取指定文件夹下的子文件列表 |
| `download_file` | 获取文件下载信息 |
| `read_file_content` | 文档内容抽取为 Markdown/纯文本 |

#### 文件组织
| 工具 | 用途 |
|------|------|
| `move_file` | 批量移动文件(夹) |
| `rename_file` | 重命名文件（夹） |

#### 分享与访问
| 工具 | 用途 |
|------|------|
| `share_file` | 开启文件分享 |
| `set_share_permission` | 修改分享链接属性 |
| `cancel_share` | 取消文件分享 |
| `get_share_info` | 获取分享链接信息 |
| `get_file_link` | 获取文件的云文档在线访问链接 |

#### 搜索
| 工具 | 用途 |
|------|------|
| `search_files` | 文件（夹）搜索 |

完整参数、示例与返回值见 `references/drive.md`。

### 不支持的操作

- 无批量删除文件工具（仅支持移动）
- 云盘 drive 侧暂无逐文件 ACL 成员矩阵（以分享链接为主）；多维表格（.dbt）见 dbsheet.permission_* 与 dbsheet.share_*（详阅 references/dbsheet.md）
- 在线 Excel / 智能表格工作表区域保护见 sheet.*_protection_ranges 相关工具（详阅 references/sheet.md）
- 无文件版本回滚
- 无实时协同编辑控制

---

## 操作指南

### 执行指南

> 执行以下操作前，**必须**先阅读对应指南文件：

| 操作类型 | 指南文件 | 何时阅读 |
|----------|----------|----------|
| 获取文件标识指南 | `references/file-locating-guide.md` | 需要搜索或浏览文件时 |
| 文件读取指南 | `references/file-reading-guide.md` | 需要获取文档内容时 |
| 文件创建与写入指南 | `references/file-writing-guide.md` | 需要创建或编辑文档时 |

⚠️ 不阅读指南直接操作可能导致：参数错误、内容丢失、格式异常。

### 高频流程指引

#### 创建并写入文档

执行顺序：
1) 先按 `references/file-locating-guide.md` 获取目标目录 `drive_id`(可选)、`parent_id`(可选)。
2) 再按 `references/file-writing-guide.md` 选择文档类型与写入路径。
字段传递：步骤 1 获取 `drive_id`(可选)、`parent_id`(可选)，作为步骤 2 的输入，执行“新建写入”流程。

#### 上传本地文件到云盘

执行顺序：
1) 先按 `references/file-locating-guide.md` 获取目标目录 `drive_id`(可选)、`parent_id`(可选)、`file_id`(可选)。
2) 再按 `references/file-writing-guide.md` 的“本地文件上传（upload_file）”路径调用上传能力（新建上传或覆盖更新）。
字段传递：新建上传使用步骤 1 的 `drive_id`(可选)、`parent_id`(可选) + `name`；覆盖更新使用步骤 1 的 `file_id` 。

#### 搜索定位文档

工具说明：`search_files(keyword="关键词", type="all", page_size=20)`，获取 `file_id`、`drive_id` 供后续链路使用。
详细参数与返回结构见 `references/drive/search.md`。

### 更多操作流程

| 流程 | 说明 | 详细参考 |
|------|------|---------|
| AI 主题生成演示文稿 | 主题生成 PPT 标准链路：澄清需求、研究资料、大纲与生成上传 | `references/workflows/topic-ppt.md` |
| AI 文档生成演示文稿 | 文档生成 PPT 标准链路：创建会话、解析文档、生成大纲、美化风格与生成上传 | `references/workflows/doc-ppt.md` |
| 网页剪藏 | 抓取网页内容并自动保存为智能文档 | `references/workflows/web-scrape.md` |
| 搜索-读取-汇报撰写 | 搜索多份文档、提取信息、汇总撰写新报告 | `references/workflows/search-read-report.md` |
| 定期读取与播报 | 定期读取指定文档，提取关键信息生成摘要 | `references/workflows/periodic-read-summary.md` |
| 智能分类整理 | 列出目录，按内容或指定维度分类创建文件夹并归档 | `references/workflows/smart-classify.md` |
| 精准搜索与风险排查 | 在特定目录批量搜索文档，逐一读取分析，汇总到新文档 | `references/workflows/precise-search-analysis.md` |
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
| 工具找不到 | service 或 action 名称错误 | 运行 `kdocs-cli --help` 或 `kdocs-cli <service> --help` 确认可用命令 |
| 搜索无结果 | 关键词过精确 / 索引延迟 | 缩短关键词 / 等待 3-5 秒重试 |
| 读取内容为空 | 文件无内容或格式不支持 | 确认文件非空且后缀正确 |
| 创建文件失败 | 文件名后缀不正确 | 检查后缀：`.otl` / `.docx` / `.xlsx` / `.ksheet` / `.dbt` / `.pdf` / `.pptx` |
| 移动文件失败 | 目标文件夹不存在 | 先搜索确认或创建文件夹 |
| HTTP 5xx / 超时 | 服务端故障 | 等 3 秒重试 1 次 |
| 验证不通过（回读值与预期不符） | 写入未生效或延迟 | 等 2 秒重新验证，仍不通过则报告用户 |
| `setup.sh` 执行失败 / 安装报错 | 当前版本可能已不兼容 | 执行上方「版本自检」流程 |
| CLI 接口返回未知错误码（非 5xx、非 400006、非 429001/429002、非工具不存在） | Skill 版本过旧导致接口不兼容 | 执行上方「版本自检」流程 |
| 错误信息含 `version`、`incompatible`、`not_supported`、`deprecated` 等版本关键词 | Skill 或 API 版本不兼容 | 执行上方「版本自检」流程 |
| 工具调用失败且原因不明 | 可能是 Skill 版本过旧 | 执行上方「版本自检」流程 |
| 上述处理方式均已尝试仍无法解决 | 未知问题 | 运行 `kdocs-cli feedback` 获取反馈链接，引导用户提交反馈 |
| 工具调用失败需判断是否可重试 | 不同工具幂等性不同 | 查看该工具参考文档「操作约束」区的幂等性说明，幂等工具可安全重试，非幂等工具须先确认状态 |

---

## 安全约束

- 凭据由 `kdocs-cli` 系统密钥链管理，Skill 自身不存储、不记录
- 无状态代理，不缓存任何文档内容或业务数据
- 仅在用户主动发起操作时调用对应 API


## 金山文档 Skill（kdocs skill）问题反馈

当满足以下任一条件时，生成反馈链接并提供给用户：

1. 错误速查表中的处理方式（重试、版本自检等）均已尝试但问题仍未解决
2. 用户主动要求反馈或投诉

运行 `kdocs-cli feedback` 获取反馈链接，将完整链接展示给用户并告知"点击即可打开反馈页面"，由用户决定是否打开。
