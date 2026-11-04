# 整夹批量移动

> 将一个文件夹下的所有文件批量移动到另一个文件夹

**适用场景**：用户要求将某个文件夹下的文件全部移动到另一个位置

**触发词**：移动文件、批量移动、移到、搬到、归档

- 用户要求将文件夹内文件批量移动到另一个位置

**工具链**：`search_files`（定位源和目标文件夹）→ `list_files`（分页列出全部文件）→ `move_file`（批量移动）

## 涉及工具

| 工具 | 服务 | 用途 |
|------|------|------|
| `search_files` | drive | 定位源文件夹和目标文件夹 |
| `list_files` | drive | 分页列出源文件夹内所有文件 |
| `move_file` | drive | 批量移动到目标文件夹 |

## 执行流程

```
步骤 1: search_files(keyword="源文件夹名", file_type="folder")
        → 获取源文件夹 file_id 和 drive_id
        search_files(keyword="目标文件夹名", file_type="folder")
        → 获取目标文件夹 file_id（作为 dst_parent_id）和 dst_drive_id
        → 目标不存在时 create_folder(name="目标文件夹") 先创建

步骤 2: list_files(drive_id, parent_id=源文件夹ID, page_size=500)
        → 收集所有 file_id（有 next_page_token 时翻页继续）

步骤 3: move_file(drive_id, file_ids=[...], dst_drive_id, dst_parent_id=目标文件夹ID)
        → ⚠️ 批量移动前需向用户确认文件列表和目标位置
```

调用接口前先执行以下命令获取接口参数及约束：
`kdocs-cli help drive.search_files drive.list_files drive.move_file`
