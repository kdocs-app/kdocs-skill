# 标签列表、打标与按标检索

> 列出标签、批量给文档打标，或按标签检索文件

**适用场景**：自定义分类标签、批量给文档打星标/项目标签，或列出「星标」「待办」等系统标签下的文件

**工具链**：`list_labels`（或已知系统标签 ID）→ `search_files` / `list_files` 收集 `file_id` → `batch_add_label_objects`；查看某标签下文件：`get_label_objects(label_id, object_type="file")`；需确认标签定义时 `get_label_meta`。

## 涉及工具

| 工具 | 服务 | 用途 |
|------|------|------|
| `list_labels` | drive | 获取标签列表 |
| `batch_add_label_objects` | drive | 批量打标签 |
| `get_label_objects` | drive | 按标签检索文件 |

## 执行流程

1. **list_labels**
   获取标签列表
2. **batch_add_label_objects**
   批量打标签
3. **get_label_objects**
   按标签检索文件
