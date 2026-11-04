# 查找知识库内的文件

> 在指定知识库或全部知识库中搜索文件

**适用场景**：在知识库里找某个文件或相关资料

**触发词**：找、搜索、查找

- 用户要求在知识库中查找文件

**工具链**：`kwiki.get_knowledge_view` → `search_files(drive_ids=[知识库drive_id])` → 展示结果

## 涉及工具

| 工具 | 服务 | 用途 |
|------|------|------|
| `kwiki.get_knowledge_view` | kwiki | 确认目标知识库存在并获取 drive_id |
| `search_files` | drive | 在知识库范围内搜索文件 |

## 执行流程

**触发示例**：「帮我找一下 XX 文件」「在 XX 库里找 XX 相关的资料」「我要找关于 XX 的文档」

**流程**：

1. **提取条件**：从用户指令中识别关键词、指定库名、文件类型等筛选条件
2. **定位搜索范围**：
   - 指定库 → `kwiki.get_knowledge_view(name=...)` 确认库存在，获取 `drive_id`
   - 未指定库 → `kwiki.list_knowledge_views` 获取全量知识库列表
3. **执行搜索**：`search_files(keyword="关键词", type="all", drive_ids=[目标drive_id列表])` 跨库或指定库检索
4. **返回结果**：按匹配度排序，展示文件名、所在库/路径、修改时间、直达链接；结果过多时提示用户按文件类型或时间范围二次筛选
5. **展示结果并询问用户** → 展示文件信息 + **主动询问是否下载到本地或打开查看（提供在线链接）用户选择下载时的后续操作**：

- `search_files` 返回的 `file_id` 可直接用于 `read_file` 等通用接口
- 根据文件类型选择下载方式，详见「下载知识库文件到本地」流程

调用接口前先执行以下命令获取接口参数及约束：
`kdocs-cli help kwiki.get_knowledge_view drive.search_files`
