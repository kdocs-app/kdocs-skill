# 回收站查看与恢复

> 查看回收站文件并恢复误删文件到原位置

**适用场景**：用户需要查看回收站或恢复误删的文件

**触发词**：回收站、恢复、误删、找回、还原

- 用户要求查看回收站内容或恢复误删文件

**工具链**：`list_deleted_files` → `restore_deleted_file`

## 涉及工具

| 工具 | 服务 | 用途 |
|------|------|------|
| `list_deleted_files` | drive | 列出回收站文件列表（支持分页） |
| `restore_deleted_file` | drive | 将选定文件还原到原位置 |

## 执行流程

```
步骤 1: list_deleted_files(page_size=20)
        → 返回回收站文件列表（含 file_id、name、type）
        → 有 next_page_token 时翻页继续

步骤 2: 向用户展示回收站文件列表，确认需要恢复的文件

步骤 3: restore_deleted_file(file_id=步骤1返回的 items[].id)
        → 勿使用 url、link_id 或 get_file_info 的 id
        → 文件还原到原位置
        → 批量恢复时逐个调用
```

调用接口前先执行以下命令获取接口参数及约束：
`kdocs-cli help drive.list_deleted_files drive.restore_deleted_file`
