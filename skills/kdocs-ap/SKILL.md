# 智能文档（otl）（工具路由索引）

金山文档智能文档（otl）提供了专属的内容写入接口，支持以 Markdown 格式向文档插入内容（标题、文本、列表等），系统自动转换为富文本格式。

**适用范围**：支持 Markdown/HTML 内容插入、追加、前置与替换，HTML/Markdown 转文档块，以及文档块的结构查询、插入、更新和删除；支持查询内嵌图片、封面和附件的资源标识。

**不适用范围**：通用创建上传、搜索、分享及多维表格内嵌文档操作。遇到上述域外需求、需要借助其他组件定位文件或准备外部资源，或无法判断请求应由哪个组件处理时，[返回主 Skill](../../SKILL.md)安排，不自行切换子 Skill。

以下工具不可逆，调用前必须向用户确认；参数与边界以 `kdocs-cli help <tool>` 的工具卡 为准：

`otl.block_delete`

---

## 前置说明（重要）
PowerShell 下复杂 JSON（含中文、数组、大对象）优先用 `--file`；完整规则见[主 Skill「调用格式」](../../SKILL.md#调用格式)。otl 示例：`kdocs-cli otl block-query --file params.json`。

## 通用说明

### 智能文档特点

- **推荐度**：⭐⭐⭐ **首选文档格式**
- 排版美观，支持标题、列表、待办、表格、分割线等丰富块组件
- 适合图文混排、报告撰写、知识文档、会议纪要等场景
- 是网页剪藏（`scrape_url`）的默认输出格式

### 读取智能文档

读取内嵌图片（非独立 `.png`/`.jpg` 文件）→ 见本文 **常用工作流 → 读取智能文档内嵌图片**。

#### 首选方式：`otl.block_query`（结构化读取）

使用 `otl.block_query` 查询文档块结构与内容，能完整获取文档的层级信息和全部块类型。传入 `params: { blockIds: ["doc"] }` 可获取全文：

```json
{
  "file_id": "file_otl_001",
  "params": { "blockIds": ["doc"] }
}
```

#### 备选方式：`read_file`（Markdown 导出）

> ⚠️ `read_file` 对智能文档存在**内容遗漏风险**——部分组件类型（如嵌入表格、附件、特殊块）可能在转换过程中丢失。**仅在需要将文档导出为 Markdown 格式时使用**，日常读取和编辑前的内容确认应优先使用 `otl.block_query`。

**图片导出**：`read_file` 默认返回图片临时可下载 URL（有效期约 10 分钟，须立即告知用户时效并询问是否下载）；显式传 `enable_upload_medias=false` 或图片本身为空/不可转时图片为 `![image]()` 空占位。


### 写入/更新已有智能文档

已有文档追加/前置/替换正文 → `otl.insert_content`（填参 `kdocs-cli help otl.insert_content`）。块级编辑 → `otl.block_query` → `otl.block_delete` / `otl.block_insert` / `otl.block_update`。

## 入参形态

长正文先写工作区 MD，再 `--content-file`（如 `otl.insert_content`）。块 ops 用 `--ops-file`：

```shell
kdocs-cli otl insert-content --file-id FID --content-file ./draft.md --format markdown --mode append
kdocs-cli otl block-update --file-id FID --ops-file ./ops.md
```

标量 ID 用 `--file-id` 等（见主 Skill「调用格式」）。

> **工具填参**：下表每行 `<tool>` 选定后须 `kdocs-cli help <tool>`（可批量：`kdocs-cli help otl.insert_content otl.block_insert`）。

## 工具列表

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `otl.insert_content` | 向智能文档插入 Markdown/HTML 内容 | `url`, `link_id`, `file_id`, `content` |
| `otl.block_insert` | 向智能文档插入一个或多个块 | `url`, `link_id`, `file_id`, `params` |
| `otl.block_delete` | 删除智能文档中一个或多个块区间 | `url`, `link_id`, `file_id`, `params` |
| `otl.block_query` | 查询智能文档指定块的结构与内容 | `url`, `link_id`, `file_id` |
| `otl.convert` | 将 HTML/Markdown 转换为智能文档块结构 | `url`, `link_id`, `file_id`, `params` |
| `otl.block_update` | 更新智能文档指定块的内容或属性 | `url`, `link_id`, `file_id`, `params` |

## 常用工作流

#### 读取智能文档内嵌图片

**触发识别**：用户要求读取/查看/下载 **智能文档（.otl）内的图片、封面或附件** 时走此流程（含表格单元格内 picture、文档封面等）。

调用接口前先执行以下命令获取接口参数及约束：
`kdocs-cli help drive.read_file otl.block_query drive.download_attachment`

**路径 B（按块下载原图）**：`otl.block_query`（`params: { blockIds: ["doc"] }`）→ 找目标 type: "picture" 节点的 `attrs.sourceKey` → `download_attachment`（`attachment_id` 填该值，文档标识沿用块查询时的值）

**注意**：若 `read_file` 返回**空图片占位**（显式 `enable_upload_medias=false` 或图片本身为空/不可转），该占位不含可下载资源，改走路径 B，**不要凭空占位调 `download_attachment`**。

## 工具组合速查

| 用户需求 | 推荐工具组合 |
|----------|-------------|
| 向已有文档追加/前置正文 | `otl.insert_content`（`mode=prepend` / `append`；全文替换用 `mode=replace`，填参 `kdocs-cli help otl.insert_content`） |
| 向已有文档追加/插入表格 | 末尾追加 → `otl.insert_content`（`format=markdown` + 管道表，**禁止 html**）；指定块位置 → `otl.block_query` → `otl.convert`（markdown）→ `otl.block_insert`（块约束见 `references/otl/node.md`，填参 `kdocs-cli help otl.insert_content otl.convert`） |
| 读取现有文档内容 | `otl.block_query`（可省略 params，默认全文；也可显式传 `params: { blockIds: ["doc"] }`） |
| **读取/下载文档内图片** | 见本文「常用工作流 → 读取智能文档内嵌图片」 |
| 导出文档为 Markdown / 落盘到工作区 | `read_file` 取正文 → Agent 写入本地 `.md`（无一体 download-otl；可能遗漏部分组件） |
| 精确修改文档块 | `otl.block_query` → `otl.block_delete` / `otl.block_insert` |
| 修改文档标题 | `otl.block_query`（`params: { blockIds: ["doc"] }`）获取 title 块 ID → `otl.block_update`（`update_content`，`content` 传 text 节点） |
| 向文档插入图片 | `upload_attachment`（获取 `object_id`）→ `otl.block_insert`（`type: "picture"`，`sourceKey` 设为 `object_id`，其余属性见 otl/node.md） |
| 下载文档中的图片/附件 | `otl.block_query` → 找到目标块的 `sourceKey` → `download_attachment`（`attachment_id` 为 `sourceKey`） |
| 获取文档封面图 | `otl.block_query`（`params: { blockIds: ["doc"] }`）→ 查看返回的 `cover.sourceKey`；可通过 `download_attachment` 下载封面图资源 |
| 设置文档封面图 | `upload_attachment`（获取 `object_id`）→ `otl.block_update`（`update_attrs`，`blockId: "doc"`，`attrs.cover.sourceKey` 设为 `object_id`） |
| 清除文档封面图 | `otl.block_update`（`update_attrs`，`blockId: "doc"`，`attrs: { cover: {} }`） |
| 外部富文本段落转块后插入（非表格） | `otl.convert`（html/markdown）→ `otl.block_insert` |
