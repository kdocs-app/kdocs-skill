# 清理知识库无用文件

> 按条件筛选并清理知识库中的过期或无用文件

**适用场景**：清理知识库里1个月未修改的文件

**触发词**：清理、删除、清空

- 用户要求清理知识库中的文件

**工具链**：`kwiki.list_items`（递归）→ 筛选 → 用户确认 → `kwiki.delete_item`

## 涉及工具

| 工具 | 服务 | 用途 |
|------|------|------|
| `kwiki.list_items` | kwiki | 递归遍历全库获取文件列表 |
| `list_files` | drive | 如需按修改时间筛选，获取 mtime |
| `kwiki.delete_item` | kwiki | 逐个删除确认的文件 |

## 执行流程

**触发示例**：「清理 XX 库里 1 个月未修改的文件」「删掉 XX 库里的空文件夹」「把 XX 库里过期的资料清理一下」

**流程**：

1. `kwiki.list_items(kuid=空间kuid)` 递归遍历全库，获取每个文件/文件夹的 `kuid`、`file_id`、`title`、`doc_type`、`ctime`
2. **如需按修改时间筛选**：用 `list_files(drive_id=知识库drive_id, parent_id=父目录file_id, page_size=500, order_by="mtime")` 获取 `mtime`，通过 `file_id` 匹配到 `kuid`
3. **向用户展示待删除清单并确认**
4. `kwiki.delete_item(kuid=xxx)` 逐个删除（进入回收站，7 天内可 `restore_deleted_file` 恢复）
5. 空文件夹可同样通过 `kwiki.delete_item` 删除

调用接口前先执行以下命令获取接口参数及约束：
`kdocs-cli help kwiki.list_items drive.list_files kwiki.delete_item`
