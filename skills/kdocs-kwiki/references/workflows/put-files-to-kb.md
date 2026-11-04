# 把文件放到知识库

> 将本地文件、网页或云盘已有文件归档到知识库中

**适用场景**：帮我把文件放到知识库的某个文件夹

**触发词**：放到知识库、归档到知识库、存到知识库

- 用户要求将文件放入知识库

**工具链**：`kwiki.get_knowledge_view` → `kwiki.list_items` → `upload_new_file` / `scrape_url` / `kwiki.import_cloud_doc`

## 涉及工具

| 工具 | 服务 | 用途 |
|------|------|------|
| `kwiki.get_knowledge_view` | kwiki | 定位目标知识库 |
| `kwiki.list_items` | kwiki | 定位目标文件夹 |
| `kwiki.create_item` | kwiki | 按需创建文件夹结构 |

## 执行流程

**触发示例**：「帮我把 XX 放到 XX 知识库」「把这些文件归档到 XX 库的 XX 文件夹」「帮我把本地 XX 文件夹的文件放到 XX 库里面」「帮我把 XX 网页的文章放到知识库」

**流程**：

1. **定位知识库**：指定库名 → `kwiki.get_knowledge_view(name=...)`；未指定 → `kwiki.list_knowledge_views` 推荐或引导创建
2. **定位目标路径**：指定文件夹 → `kwiki.list_items` 逐层查找，不存在则 `kwiki.create_item(doc_type="folder")` 按层级创建；未指定 → 根目录
3. **归档**：
   - 本地文件/文件夹 → 按「上传本地文件到知识库」流程逐个上传，保持子目录结构时递归创建文件夹
   - 网页 → `scrape_url` + `scrape_progress` 轮询完成后，`move_file` 移入目标库
   - 云盘已有文件 → `kwiki.import_cloud_doc(action="copy"/"shortcut")`
4. **确认结果**：`kwiki.list_items` 返回存放路径与直达链接；批量时展示成功/失败明细

调用接口前先执行以下命令获取接口参数及约束：
`kdocs-cli help kwiki.get_knowledge_view kwiki.list_items kwiki.create_item`
