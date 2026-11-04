# 重命名知识库内的文件或文件夹

> 通过 kwiki.list_items 获取 file_id 后调用 rename_file 重命名知识库内的文件或文件夹

**适用场景**：帮我把知识库里某个文件改个名字

**触发词**：重命名、改名、改文件名

- 用户要求重命名知识库中的文件或文件夹

**工具链**：`kwiki.get_knowledge_view` → `kwiki.list_items` → `rename_file`

## 涉及工具

| 工具 | 服务 | 用途 |
|------|------|------|
| `kwiki.get_knowledge_view` | kwiki | 获取知识库 drive_id |
| `kwiki.list_items` | kwiki | 定位目标文件，获取 file_id |
| `rename_file` | drive | 执行重命名 |

## 执行流程

使用通用接口 `rename_file` 重命名知识库内的文件或文件夹：

1. `kwiki.get_knowledge_view(name="知识库名")` 获取知识库的 `drive_id` 和 `kuid`
2. `kwiki.list_items(kuid=知识库kuid)` 定位目标文件，获取 `file_id` 和 `drive_id`
3. `rename_file(drive_id=drive_id, file_id=file_id, dst_name="新名称")`
   - 文件须带后缀（如 `"新报告.docx"`）
   - 文件夹不带后缀（如 `"项目资料"`）

调用接口前先执行以下命令获取接口参数及约束：
`kdocs-cli help kwiki.get_knowledge_view kwiki.list_items drive.rename_file`
