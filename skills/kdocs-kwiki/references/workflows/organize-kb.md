# 整理分类知识库

> 遍历知识库内容，按类型或主题创建分类文件夹并移动文件

**适用场景**：帮我整理一下知识库，按类型分类

**触发词**：整理、分类、归类

- 用户明确提到知识库/库/资料库等关键词

**工具链**：`kwiki.list_knowledge_views` → `kwiki.list_items`（递归）→ AI 分类 → `kwiki.create_item(folder)` → `move_file`

## 涉及工具

| 工具 | 服务 | 用途 |
|------|------|------|
| `kwiki.list_knowledge_views` | kwiki | 搜索目标知识库 |
| `kwiki.list_items` | kwiki | 遍历根目录及子文件夹内容 |
| `kwiki.create_item` | kwiki | 批量创建分类文件夹 |
| `move_file` | drive | 按分类方案移动文件（file_id 来自 kwiki.list_items） |

## 执行流程

**触发示例**：「帮我整理一下 XX 知识库」「把 XX 库里的文件按类型分类」

> ⚠️ **场景识别**：当用户明确提到「知识库」「库」「资料库」等关键词时，优先使用 `kwiki.*` 系列接口完成整理/分类，确保知识库内部元数据（索引、搜索等）一致。

**流程**：

1. `kwiki.list_knowledge_views(keyword="库名")` 搜索目标知识库，获取 `drive_id`、`group_id`、`kuid` 等关键标识
2. `kwiki.list_items(kuid=知识库kuid)` 遍历根目录内容；如需递归遍历子文件夹，继续用文件夹的 `kuid` 调用 `kwiki.list_items`。收集每个文件的 `kuid`、`title`、`doc_type` 信息
3. 列出需新建的分类文件夹、文件移动目标、建议删除的内容，明确标注操作影响范围，**提交用户确认后再执行**
4. 批量创建文件夹（`kwiki.create_item`）→ 批量移动文件（`move_file`）→ 删除确认的冗余内容（`kwiki.delete_item`），提示回收站恢复路径（7 天内可恢复）

调用接口前先执行以下命令获取接口参数及约束：
`kdocs-cli help kwiki.list_knowledge_views kwiki.list_items kwiki.create_item drive.move_file`
