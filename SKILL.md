---
name: kdocs
description: 操作金山文档（WPS 云文档 / Kdocs）云文档的官方 Skill，支持处理 `https://kdocs.cn/l/...`、`https://www.kdocs.cn/l/...` 或 `https://365.kdocs.cn/l/...` 形式的文档 URL。核心能力覆盖云端新建、读取、编辑、搜索、分享、整理在线文档（智能文档、Word、Excel、PDF、PPT、演示文稿、智能表格、多维表格）。当用户的任务涉及云文档操作时使用，包括但不限于：写周报/日报/工作汇报、处理合同/发票、创建报名表/登记表、网页剪藏、接龙转表格、信息收集、文档总结与内容生成、改写仿写、翻译、AI PPT生成、PDF拆分导出、标签分类归档、收藏管理、碎片笔记整理、表格美化、回收站还原。勿直接执行，先转交子 Skill。
homepage: https://www.kdocs.cn/latest
version: 2.9.0
metadata: {"requires":{"bins":["kdocs-cli"],"cliHelp":"kdocs-cli help"},"openclaw":{"category":"kdocs","tokenUrl":"https://www.kdocs.cn/latest","emoji":"📝","keywords":["金山文档","金山表格","金山收藏","WPS","WPS文档","云文档","在线文档","kdocs","WPS云文档","接龙转表格","接龙","群接龙","报名表","信息收集","收集表","登记表","网页剪藏","剪藏","保存网页","网页保存到文档","保存文章","收藏文章","总结","帮我总结","帮我整理","帮我写","帮我翻译","帮我做PPT","翻译文档 - 做PPT - 生成PPT - 培训课件 - 方案展示 - 项目展示","文档总结","内容生成","改写","仿写","翻译","文档翻译","PPT","演示文稿","幻灯片","PDF","拆分PDF","导出PDF","Word","Excel","表格","Markdown","碎片整理","笔记整理","表格优化","文档处理","文件处理","办公助手","文档助手","周报","日报","工作汇报","合同","发票"]},"file_types":["pdf","doc","docx","xlsx","xls","pptx","ppt","otl","ksheet","dbt","form","jpg","jpeg","png","bmp","gif","webp","url","md","txt","html"],"category":"productivity"}
---

# 金山文档 CLI Skill 使用指南

金山文档 CLI Skill 提供了一套完整的在线文档操作工具，通过 `kdocs-cli` 命令行工具与金山文档 API 交互。支持创建、查询、读取、编辑、分享、移动多种类型的在线文档。

## 严格规则

### 禁止（NEVER）

- 禁止将 Token 明文出现在对话、日志、命令输出、代码注释或任何文件中；Token 仅允许通过 `kdocs-cli auth set-token` 或 `kdocs-cli auth login` 保存到系统密钥链
- 上传写入等接口需传入的 `content_base64` 可能非常大（编码后 >1 MB），禁止在对话中逐 token 生成 Base64 字符串，用脚本完成文件读取、编码和传参
- 工具返回或 stderr 明确表示**当前文件/链接整份不可访问**（如整文件无权限、需向所有者申请访问、无权打开该文档等）时：**停止对该文件的后续读写**，不得换参数对该文件变通重试；告知用户联系文件所有者授权。若用户提供了其他可访问文件或明确换目标，可继续处理其它文件。单次字段/单表/局部操作权限不足且未声明整文件不可用时：说明原因，跳过该步或询问用户是否继续，**不得**仅因子串「权限」就终止本会话内所有云文档操作

### 必须（MUST）

- 不可逆操作（delete/close 类）执行前必须向用户确认
- 创建云文档文件并验证通过后，必须向用户展示可访问链接。若响应包含 `data.link_url` 则直接展示；若响应无链接时，调用 `get_file_link` 获取并展示。
- 除 SKILL.md 已给出最小可用示例的主路径工具外，其他工具调用前须先执行 `kdocs-cli help <tool>` 获取详细参考文档；参数细节（类型、可选值、约束）以工具参考文档为准
- 满足「问题反馈」章节触发条件时，必须运行 feedback 命令并展示完整反馈链接，并遵守该章节的防重复规则；禁止仅口头告知联系客服或仅以「暂不支持」结束对话
- 参考文档按本技能总览或索引中的文件路径读取，不猜文件名、不把目录当文件、不套用其他技能的路径。路径报错时先核对索引。

---

## 保持最新版本
何时触发：**首次使用** Skill / **距上次自检 >24h** / **收到 `unknown action` 或 `unknown service` 错误**。

1. **CLI 版本**：`kdocs-cli version` — 若命令不存在则按下方「工具安装与认证」安装；若低于本文件 frontmatter `version`，运行 `kdocs-cli upgrade -y`（自动备份旧版本，失败可 `kdocs-cli upgrade --rollback`）
2. **Skill 版本**：若本文件 `version` 低于 `kdocs-cli version`，运行 `kdocs-cli call check_skill_update --args '{"version":"2.9.0","skill_name":"kdocs"}'`，若返回 `update_available: true`，从 `instruction` 中提取 zip 下载链接，下载解压替换当前 Skill 目录

若 upgrade 和 rollback 均失败，在本 Skill 目录下重新运行安装脚本（`bash setup.sh` / `powershell setup.ps1` / `node setup.cjs`，脚本位于 `scripts/`）可从 CDN 重新安装。
若仍无法更新，以各能力域 service 参考页内工具选型指引与 `kdocs-cli version` 所对应能力为准，并运行 `kdocs-cli feedback`。

---

## 工具安装与认证

| 操作 | 命令 |
|------|------|
| 安装 | `bash scripts/setup.sh` / `powershell scripts/setup.ps1` / `node scripts/setup.cjs` |
| 认证 | 用户已提供 Token: `kdocs-cli auth set-token "<token>"` · 无 Token: `kdocs-cli auth login` |

login 失败时的手动获取流程、`auth status` 诊断、`auth logout` 退出等详见 `references/auth.md`。

---

## 调用格式

```
kdocs-cli <service> <action> [参数]
```

### 参数传递

| 场景 | 写法 | 示例 |
|------|------|------|
| 标量参数（单个或多个） | 连续 `--key value` | `kdocs-cli drive search-files --keyword test --type all` |
| 数组/对象、中文或超长 JSON | `--file` 指向**完整** JSON 参数文件 | `kdocs-cli otl insert-content --file payload.json` |

- `--file` 必须是该工具的**完整 JSON 参数对象**；使用 `--file` 时**只读 JSON 文件**，命令行里 `<service> <action>` 之后不要再写 `--file-id` 等工具选项（多余选项会被忽略，容易误以为生效）
- `--file` = 本地 JSON 参数文件；`--file-id` = 云文档 ID（reference 里 JSON 字段 `file_id` 写成 `--file-id`）。勿把云文档 ID 或 `.otl` 路径传给 `--file`
- PowerShell 传 JSON 字符串须反斜杠转义：`'{\"key\":[\"val\"]}'`

> **`--file` 示例**：写入大段内容时，用脚本生成 JSON 文件再传入，操作完成后删除临时文件：
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

**全局选项**（Agent 子集；可与 `<service> <action>` 同条命令；勿用 `--compact` / `--silent` / `--endpoint` 等替代）：

| 选项 | 说明 | 示例 |
|------|------|------|
| `--file <path>` | 从 JSON 文件读取完整参数对象（与 `--args` 互斥） | `kdocs-cli otl insert-content --file payload.json` |
| `--args <json>` | 显式 JSON 参数（与 `--file` 互斥） | `kdocs-cli drive search-files --args '{"keyword":"test","type":"all"}'` |
| `--timeout <ms>` | HTTP 请求超时（毫秒，默认 30000） | `kdocs-cli drive get-file-info --file-id <id> --timeout 120000` |
| `--token <token>` | 一次性 Token（优先级最高，不持久化） | `kdocs-cli drive list-my-files --page-size 10 --token <token>` |

成功时 stdout 为单行 JSON（工具返回的 `data`）；失败时 stderr 为结构化错误。

**帮助**：`kdocs-cli help`、`kdocs-cli help <service>`、`kdocs-cli help <tool>`

> **找不到命令？** 浏览 `kdocs-cli help` / `kdocs-cli help <service>` 时若发现预期的 service 或 action 不存在，先运行 `kdocs-cli upgrade -y` 升级到最新版本再重试。CLI 能力随版本持续扩展，未升级是命令缺失的首要原因。详见上方「保持最新版本」章节。

---

## 能力范围

### 操作域路由

Agent 首先判定用户请求的操作域，**将后续步骤转交到对应子 Skill**，在子 Skill 内按索引选用工具。子 Skill 遇到域外需求时返回主 Skill；主 Skill 保留已完成操作、实际资源和任务 ID，再安排后续转交。

| 操作域 | 触发场景 | 路由 |
|--------|---------|------|
| 创建/写入 | 新建并写入、上传本地文件、新建空白文档 | 转交到 `kdocs-drive` 技能「创建/写入」（后缀不确定时默认 `.otl`；AI 生成 PPT 见「AI 生成」操作域） |
| 局部更新 | 改块/改段/改单元格，已有目标文档上的修改 | 按「支持的文档类型」→ 对应子 Skill 中的写入/更新类工具 |
| 类型专属能力 | 条件格式、导出转换、翻译、PDF 拆分、幻灯片主题、数据校验 | 按「支持的文档类型」→ 对应子 Skill 中的专属功能章节 |
| 读取 | 读取/提取/导出文档内容 | 正文：转交到 `kdocs-drive` 技能（工具列表 → `read_file`，`kdocs-cli help drive.read_file`）；**.otl 内嵌图片** 转交到 `kdocs-ap` 技能「常用工作流 → 读取智能文档内嵌图片」；否则按「定位文件」 |
| 定位文件 | 搜索/按链接查文件标识或属性/浏览目录 | 转交到 `kdocs-drive` 技能 |
| 文件管理 | 移动/重命名/分享/标签/收藏/回收站 | 转交到 `kdocs-drive` 技能 |
| AI 生成 | AI 做PPT/生成演示文稿 | 转交到 [`kdocs-aippt`](skills/kdocs-aippt/SKILL.md) 技能 |
| 知识库 | 知识库空间、资料导入、库内目录与资料管理 | 转交到 [`kdocs-kwiki`](skills/kdocs-kwiki/SKILL.md) 技能 |
| 自动化 | 多维表格与表格：查询、读取、校验、保存、启停流程及核对运行记录 | 转交到 [`kdocs-auto-sheet`](skills/kdocs-auto-sheet/SKILL.md) 技能 |

### 云文档管理

| 能力 | 说明 | 子 Skill |
|------|------|----------|
| 云文档管理 | 搜索定位、创建上传、读取下载、分享移动、标签收藏、回收站等 | [`kdocs-drive`](skills/kdocs-drive/SKILL.md)「云文档管理」 |

### 支持的文档类型

| 类型 | 别名 | 后缀 | 说明 | 子 Skill |
|------|------|------|------|----------|
| **智能文档** 首选 | ap | .otl | 排版美观，支持丰富组件 | [`kdocs-ap`](skills/kdocs-ap/SKILL.md) |
| 在线 Markdown | md | .md | 源码 Markdown，图片为 kdocsmd:// 协议 | [`kdocs-drive`](skills/kdocs-drive/SKILL.md) |
| 表格 | et / Excel | .xlsx | 数据表格专用 | [`kdocs-et`](skills/kdocs-et/SKILL.md) |
| PDF文档 | pdf | .pdf | PDF 文档专用 | [`kdocs-pdf`](skills/kdocs-pdf/SKILL.md) |
| 文字文档 | wps / Word | .docx | 传统格式 | [`kdocs-wps`](skills/kdocs-wps/SKILL.md) |
| 演示文稿 | wpp | .pptx | PPT 文档专用 | [`kdocs-wpp`](skills/kdocs-wpp/SKILL.md) |
| 智能表格 | as | .ksheet | 结构化表格，支持多视图、字段管理 | [`kdocs-et`](skills/kdocs-et/SKILL.md) |
| 多维表格 | db / dbsheet | .dbt | 多数据表、丰富字段类型与视图（表格/看板/甘特等） | [`kdocs-db`](skills/kdocs-db/SKILL.md) |
| 智能表单 | form | .form | 轻量表单草稿创建、题目配置、发布与查询 | [`kdocs-form`](skills/kdocs-form/SKILL.md) |

---

## 错误速查

| 错误特征 | 原因 | 处理方式 |
|----------|------|----------|
| `400006` / 鉴权失败 | Token 过期或未配置 | 运行 `kdocs-cli auth login` 重新登录，或 `kdocs-cli auth set-token <token>` 重新设置 |
| `429001` / 限频 | 请求过于频繁，响应含**限频恢复时间** | 立即停止命令调用，直到达到恢复时间；禁止立即重试、换参、换子命令连续请求 |
| `429002` / 熔断 | 多因短时间内连续触发 `429001` ，响应含**熔断持续时间** | 熔断时长内零请求，期满再试；重新规划任务避免请求过频 |
| `403` / 权限不足 / `无权访问` / `forbidden` | 当前凭据对目标文档、目录或资源无操作权限 | 停止操作，禁止重试或尝试其他接口绕过；告知用户当前账号无权限，并建议联系文档所有者开通权限、确认分享链接权限，或切换到有权限的账号 |
| 文件不存在类：`文件(夹)不存在`、`链接不存在` 等 | 文件已删除或链接失效，重试不会成功 | 直接告知用户；需要继续找同一份文件时用 `search_files` 重新定位 |
| 无权限类：`无权限`、`用户不在企业内`、`文档协作已关闭` 等 | 当前账号无权访问该文件，重试不会成功 | 直接告知用户，请其确认权限或换一份有权限的文件 |
| `unknown action` / `unknown service` | CLI 版本过旧或名称拼写错误 | 先运行 `kdocs-cli upgrade` 升级到最新版本；仍报错再以 `kdocs-cli help` / `kdocs-cli help <service>` 浏览索引，用表内点号名执行 `kdocs-cli help <tool>` 取参，再以 `kdocs-cli <service> <action>` 执行 |
| 第三方服务错误：任务不存在 | 使用了无效、跨请求或已完成的 `task_id` | 前一次已返回完整内容时直接使用已有结果；否则丢弃旧 `task_id`，按原文件定位参数重新读取 |
| `Client.Timeout exceeded while awaiting headers` | 服务端处理或排队时间超过 CLI HTTP 超时，常见于上传、导出、AI 生成、格式转换、大文件读取等慢操作 | 确认工具幂等性后重试 1 次，并显式设置 `--timeout` 值（如 `--timeout=120000`）；写入/创建类工具重试前先查询结果，避免重复创建 |
| `conflict` / `lock` / 并发写入冲突 | 多个写操作同时修改同一资源（知识库节点、多维表记录等）导致锁竞争 | 指数退避重试（2s → 4s → 8s，最多 3 次）；批量写入场景改为串行逐条执行；详见 kwiki / dbsheet 各 reference「错误速查表」 |
| HTTP 5xx | 服务端故障 | 等 3 秒重试 1 次 |
| 验证不通过（回读值与预期不符） | 写入未生效或延迟 | 等 2 秒重新验证，仍不通过则报告用户 |
| `setup.sh` 执行失败 / 安装报错 | 当前版本可能已不兼容 | 执行上方「保持最新版本」流程 |
| CLI 接口返回未知错误码（非 5xx、非 400006、非 429001/429002、非工具不存在） | Skill 版本过旧导致接口不兼容 | 执行上方「保持最新版本」流程；仍失败到 `kdocs-cli feedback` |
| 错误信息含 `version`、`incompatible`、`not_supported`、`deprecated` 等版本关键词 | Skill 或 API 版本不兼容 | 执行上方「保持最新版本」流程；仍失败到 `kdocs-cli feedback` |
| 工具调用失败且原因不明 | 可能是 Skill 版本过旧 | 执行上方「保持最新版本」流程；仍失败到 `kdocs-cli feedback` |
| 上述处理方式均已尝试仍无法解决 | 未知问题 | 仍失败则 `kdocs-cli feedback` |
| 工具调用失败需判断是否可重试 | 不同工具幂等性不同 | 执行 `kdocs-cli help <tool>`，按工具卡「调用约束」中的幂等性说明判断；幂等工具可安全重试，非幂等工具须先确认状态 |

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
