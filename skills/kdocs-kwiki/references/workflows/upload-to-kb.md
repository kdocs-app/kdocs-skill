# 上传本地文件到知识库

> 上传本地文件到知识库，支持常规文件和 Markdown 转智能文档

**适用场景**：把本地文件归档/上传/同步/放到知识库的文件夹

**触发词**：上传、归档、同步、放到知识库

- 用户要求将本地文件上传或归档到知识库

**工具链**：`kwiki.get_knowledge_view` → `kwiki.list_items` → `upload_new_file` / `kwiki.create_item` + `otl.insert_content`

## 涉及工具

| 工具 | 服务 | 用途 |
|------|------|------|
| `kwiki.get_knowledge_view` | kwiki | 获取目标知识库的 drive_id |
| `kwiki.list_items` | kwiki | 定位目标文件夹，获取 kuid |
| `upload_new_file` | drive | 上传常规文件（docx/pdf/pptx/xlsx 等） |
| `kwiki.create_item` | kwiki | Markdown 文件转智能文档时，先创建智能文档 |
| `otl.insert_content` | otl | 将 Markdown 内容写入智能文档 |

## 执行流程

**触发示例**：「把本地 XX 文件归档/上传/同步/放到 XX 库的 XX 文件夹」「把这些资料归档/上传/同步/放到 XX 库」「定时归档/上传/同步/放 XX 到 XX 库」

**流程**：
1. `kwiki.get_knowledge_view` 或 `kwiki.list_knowledge_views` 获取目标知识库的 `drive_id`
2. 如需放入子文件夹：
   - `kwiki.list_items` 定位目标文件夹，获取其 `kuid`（用于 kwiki 操作）和 `file_id`（用于通用接口的 `parent_id`）
   - 放入根目录则 `parent_id="0"`
3. 按文件类型选择上传方式：

**常规文件（docx/pdf/pptx/xlsx 等）**：
`upload_new_file(drive_id=知识库drive_id, parent_id=目标文件夹id, name="文件名.docx", content_base64=...)`

**Markdown 文件（.md）**：
> 默认转为在线智能文档，保留格式和结构化内容。仅当用户明确要求"上传并保持 md 格式"时，才使用 `upload_new_file` 直接上传原始 `.md` 文件。

- `kwiki.create_item(doc_type="o", kuid=目标文件夹kuid, title="文件名（不含后缀）")` 创建智能文档
- 读取本地 `.md` 文件内容
- `otl.insert_content(file_id=新文档的file_id, content=markdown原文, format="markdown", mode="prepend")` 将 markdown 写入智能文档
  - 如果内容过长（>3000 字符），分段写入：首段用 `mode="prepend"`，后续段用 `mode="append"` 追加
- 从 `kwiki.list_items` 返回中获取 `link_id`，拼接在线链接

调用接口前先执行以下命令获取接口参数及约束：
`kdocs-cli help kwiki.get_knowledge_view kwiki.list_items drive.upload_new_file kwiki.create_item otl.insert_content`
