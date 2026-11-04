# 个人知识库（kwiki）（工具路由索引）

本文件说明金山文档 Skill 中个人知识库相关的 `kwiki.*` 工具如何使用。它们面向"知识库空间"和"知识库内资料管理"场景，适合创建知识库、浏览库内目录、导入已有云文档，以及整理知识库中的文件和文件夹。

**适用范围**：知识库空间、库内文件夹与资料的查询、创建、导入、删除及管理。

**不适用范围**：通用云盘文件管理、内容编辑、上传下载及跨组件步骤，返回主 Skill 安排。遇到上述域外需求、需要借助其他组件定位文件或准备外部资源，或无法判断请求应由哪个组件处理时，[返回主 Skill](../../SKILL.md)安排，不自行切换子 Skill。

以下工具不可逆，调用前必须向用户确认；参数与边界以 `kdocs-cli help <tool>` 的工具卡 为准：

`kwiki.close_knowledge_view`、`kwiki.delete_item`

## 通用说明

### 何时使用 `kwiki.*`

- 需要新建一个个人知识库或资料库空间
- 需要查询已有知识库列表，或按名称/ID 获取某个知识库详情
- 需要浏览知识库根目录或某个知识库文件夹下的资料
- 需要把**已有云文档**导入知识库
- 需要在知识库里新建文件夹或在线文件，并对库内资料做删除、下载

### 特别说明

> - 仔细阅读接口参数说明，不猜测，不胡编乱造
> - 本地上传不走 `kwiki.*`

### 链接输出规范

接口返回的数据中，`url` 字段为**相对路径**（如 `/l/xxx?source=kmwiki` 或 `/wiki/l/xxx`），`kuid`字段为**知识库/文件夹/文件id**。**Agent 在拼接完整链接时，必须遵循以下规则，不猜测：**

1. **拼接规则**：`https://www.kdocs.cn` + `data.url 原值`。
2. **手动构造**：若接口未返回 `url` 但返回了 `kuid`，格式为 `https://www.kdocs.cn/wiki/l/${kuid}`。

### 标识说明

在 `kwiki.*` 场景里，常见会用到以下标识：

- `drive_id`: 知识库对应的云盘 ID
- `group_id`: 知识库所属组 ID
- `kuid`: 知识库或知识库内文件/文件夹的标识

经验上：

- 知识库本身的 `kuid` 常见为 `0s...`
- 知识库内文件夹/文件的 `kuid` 常见为 `0l...`

如果用户只给了知识库名称，通常先用 `kwiki.list_knowledge_views` 搜，再把返回的 `drive_id` / `group_id` / `kuid` 传给后续工具。

> **注意**： `kuid` 仅用于 kwiki 专属操作（`delete_item`/`import_cloud_doc` 等）。

> **工具填参**：下表每行 `<tool>` 选定后须 `kdocs-cli help <tool>`（可批量：`kdocs-cli help kwiki.create_knowledge_view kwiki.list_knowledge_views`）。

## 工具列表

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `kwiki.create_knowledge_view` | 创建个人知识库 | `space_name`, `status` |
| `kwiki.list_knowledge_views` | 查询知识库列表 |  |
| `kwiki.get_knowledge_view` | 获取单个知识库详情 | `drive_id`, `name` |
| `kwiki.update_knowledge_view` | 修改知识库基础配置 | `drive_id`, `cover_img`, `status` |
| `kwiki.close_knowledge_view` | 关闭（删除）知识库 | `drive_id` |
| `kwiki.list_items` | 列出知识库目录下的内容 | `kuid` |
| `kwiki.import_cloud_doc` | 将已有云文档导入知识库 | `kuid`, `file_infos` |
| `kwiki.create_item` | 在知识库中创建文件或文件夹 | `doc_type`, `kuid`, `title` |
| `kwiki.delete_item` | 删除知识库中的文件或文件夹 | `kuid` |

---

## 高频操作流程

| 流程 | 说明 | 详细参考 |
|------|------|----------|
| 上传本地文件到知识库 | 上传本地文件到知识库，支持常规文件和 Markdown 转智能文档 | `references/workflows/upload-to-kb.md` |
| 重命名知识库内的文件或文件夹 | 通过 kwiki.list_items 获取 file_id 后调用 rename_file 重命名知识库内的文件或文件夹 | `references/workflows/rename-in-kb.md` |
| 下载知识库文件到本地 | 从知识库下载文件到本地 | `references/workflows/download-from-kb.md` |
| 把文件放到知识库 | 将本地文件、网页或云盘已有文件归档到知识库中 | `references/workflows/put-files-to-kb.md` |
| 查找知识库内的文件 | 在指定知识库或全部知识库中搜索文件 | `references/workflows/search-in-kb.md` |
| 整理分类知识库 | 遍历知识库内容，按类型或主题创建分类文件夹并移动文件 | `references/workflows/organize-kb.md` |
| 清理知识库无用文件 | 按条件筛选并清理知识库中的过期或无用文件 | `references/workflows/cleanup-kb.md` |
| 网页内容存入知识库 | 将网页内容剪藏后存入指定知识库 | `references/workflows/web-to-kb.md` |
| 知识智能整理 | 对知识库中的零散内容进行智能化整理和结构化重组 | `references/workflows/knowledge-format.md` |
| 知识一键存入 | 将各类内容（网页、文件、文本）一键保存到知识库 | `references/workflows/knowledge-save.md` |

## 错误速查表

> ⛔ **强制规则**：命中下方任一错误条目时，**必须立即按「处理方式」向用户提示，禁止尝试其他接口绕过或反复重试。**

| 错误特征 | 原因 | 处理方式 |
|----------|------|----------|
| `code: 403000006`，`msg: "当前版本仅支持个人用户"` | 当前登录的是企业/团队账号，该知识库接口仅对个人账号开放 | 提示用户切换至个人账号后重试 |
| `conflict` / `lock` / 写入冲突 | 并发操作同一知识库节点（如同时创建/移动/删除兄弟节点）导致锁竞争 | 指数退避重试（2s → 4s → 8s，最多 3 次）；批量操作兄弟节点时改为串行逐条执行 |
