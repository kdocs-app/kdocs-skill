# 智能文档（otl）工具完整参考文档

金山文档智能文档（otl）提供了专属的内容写入接口，支持以 Markdown 格式向文档插入内容（标题、文本、列表等），系统自动转换为富文本格式。

---

## 前置说明（重要）
当终端为 PowerShell 时，为避免转义问题，推荐使用 `@file` 方式传入 JSON 参数：
### JSON 参数传递方式
#### 方式一：`@file`（推荐）
先将参数写入 JSON 文件，再用 `@file` 传入：

示例
```powershell
kdocs-cli otl block-query @params.json
```
#### 方式二：JSON 字符串
PowerShell 中双引号用 `\"` 转义：

示例
```powershell
kdocs-cli otl block-query '{\"file_id\":\"cqTNWO4EMAn9\",\"params\":{\"blockIds\":[\"doc\"]}}'
```

## 通用说明

### 智能文档特点

- **推荐度**：⭐⭐⭐ **首选文档格式**
- 排版美观，支持标题、列表、待办、表格、分割线等丰富块组件
- 适合图文混排、报告撰写、知识文档、会议纪要等场景
- 是网页剪藏（`scrape_url`）的默认输出格式

### 创建智能文档

通过 `create_file` 创建，`name` 须带 `.otl` 后缀，`file_type` 设为 `file`：

```json
{
  "name": "项目周报.otl",
  "file_type": "file",
  "parent_id": "folder_abc123"
}
```

创建完成后用下文 **`otl.insert_content`** 写入 Markdown/文本。**勿**对 `.otl` 使用 `upload_file`：该工具面向本地文字/表格/演示/PDF 文件上传，不支持 `.otl` 智能文档。

### 读取智能文档

#### 首选方式：`otl.block_query`（结构化读取）

使用 `otl.block_query` 查询文档块结构与内容，能完整获取文档的层级信息和全部块类型。传入 `blockIds: ["doc"]` 可获取全文：

```json
{
  "file_id": "file_otl_001",
  "params": { "blockIds": ["doc"] }
}
```

#### 备选方式：`read_file_content`（Markdown 导出）

> ⚠️ `read_file_content` 对智能文档存在**内容遗漏风险**——部分组件类型（如嵌入表格、附件、特殊块）可能在转换过程中丢失。**仅在需要将文档导出为 Markdown 格式时使用**，日常读取和编辑前的内容确认应优先使用 `otl.block_query`。

##### 步骤 1：提交读取任务

调用参数：

```json
{
  "drive_id": "drive_abc123",
  "file_id": "file_otl_001",
  "format": "markdown",
  "include_elements": ["all"]
}
```

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `drive_id` | string | 是 | 文件所在云盘 ID |
| `file_id` | string | 是 | 文件 ID |
| `format` | string | 是 | 固定传 `"markdown"` |
| `include_elements` | array | 是 | 固定传 `["all"]` |

> **⚠️ CLI 调用注意**：`include_elements` 是**数组**，`key=value` 语法无法可靠传递数组。请用 JSON 字符串或 `@file` 传递：
>
> ```shell
> kdocs-cli drive read-file-content '{"drive_id":"<DRIVE_ID>","file_id":"<FILE_ID>","format":"markdown","include_elements":["all"]}'
> ```
>
> 或使用 `@file` 方式：`kdocs-cli drive read-file-content @params.json`

##### 步骤 2：轮询获取内容

将步骤 1 返回的 `task_id` 加入参数再次调用：

```json
{
  "drive_id": "drive_abc123",
  "file_id": "file_otl_001",
  "format": "markdown",
  "include_elements": ["all"],
  "task_id": "步骤1返回的task_id"
}
```

> **CLI 轮询命令**（替换 `<TASK_ID>` 为步骤 1 返回值）：
>
> ```shell
> kdocs-cli drive read-file-content '{"drive_id":"<DRIVE_ID>","file_id":"<FILE_ID>","format":"markdown","include_elements":["all"],"task_id":"<TASK_ID>"}'
> ```
>
> 或使用 `@file` 方式（将含 `task_id` 的完整参数写入 JSON 文件）：`kdocs-cli drive read-file-content @params.json`

> ⚠️ **`include_elements` 必须是数组** `["all"]`，不是字符串 `"all"`。传错类型会导致服务端仅返回段落文本。
>
> ⚠️ 轮询间隔按文件大小分档：`size` 字段统一使用 `FileInfo.size`（bytes）。优先从 `search_files.items[].file.size` 获取；若当前入口为 `get_share_info`（仅返回 `file_id/drive_id`），先调用 `get_file_info(file_id)` 补齐 `data.size`。推荐策略：
> - `small`（`<1MB`）：`1s,1s,2s`
> - `medium`（`1MB~10MB`）：`2s,2s,3s`
> - `large`（`>=10MB`）：`3s,3s,5s`
> - `size` 缺失/异常：回退 `2s,2s,3s,5s`

---

## 一、内容写入与转换

> 整篇 Markdown/纯文本写入与 HTML/Markdown 转块数据

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| [`otl.insert_content`](otl/insert_content.md) | 向智能文档插入 Markdown/文本内容 | `file_id`, `content` |
| [`otl.convert`](otl/convert.md) | 将 HTML/Markdown 转换为智能文档块结构 | `file_id`, `params` |

## 二、块级操作

> 按 block id 定位进行查询、插入、更新、删除

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| [`otl.block_insert`](otl/block_insert.md) | 向智能文档插入一个或多个块 | `file_id`, `params` |
| [`otl.block_delete`](otl/block_delete.md) | 删除智能文档中一个或多个块区间 | `file_id`, `params` |
| [`otl.block_query`](otl/block_query.md) | 查询智能文档指定块的结构与内容 | `file_id`, `params` |
| [`otl.block_update`](otl/block_update.md) | 更新智能文档指定块的内容或属性 | `file_id`, `params` |

## 工具组合速查

| 用户需求 | 推荐工具组合 |
|----------|-------------|
| 新建文档并写入内容 | `create_file` → `otl.insert_content` |
| 读取现有文档内容 | `otl.block_query`（`blockIds: ["doc"]` 获取全文） |
| 导出文档为 Markdown | `read_file_content`（可能遗漏部分组件内容） |
| 精确修改文档块 | `otl.block_query` → `otl.block_delete` / `otl.block_insert` |
| 外部内容转块后插入 | `otl.convert` → `otl.block_insert` |
