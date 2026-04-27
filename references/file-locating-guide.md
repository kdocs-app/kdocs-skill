# 获取文件标识指南

大多数工具需要 `file_id` 和 `drive_id` 参数。按用户提供的信息选择定位方式：

`create_file` 与 `upload_file`（新建）中，`drive_id`、`parent_id` 在接口上**可为非必填**，但**凡能先定位到的，均应显式传入**，以减少落点与用户意图不符的情况。

| 用户提供 | 定位方式 |
|---------|---------|
| 文件名/关键词 | `search_files` → 返回结果中包含 `file_id` 和 `drive_id` |
| 文档链接 | 从 URL 提取 `link_id`（见下方链接解析）→ `get_share_info(link_id)` → 取 `file_id` 和 `drive_id` |
| 已知 `file_id` | `get_file_info(file_id)` → 补充获取 `drive_id` |
| 创建文件（指定目录） | `search_files` 搜索目标目录 → 取 `drive_id` 和 `file_id`（作为 `parent_id`） |
| 创建文件（未指定目录） | `search_files(file_type="folder", type="all", scope="personal_drive", page_size=1)` → 取命中目录的 `drive_id`，`parent_id` 常用 `"0"`，再调用 `create_file` / `upload_file`（新建） |

> 根目录的 `parent_id` 固定为 `"0"`。

#### 文档链接解析

当链接域名为 `365.kdocs.cn` 或 `www.kdocs.cn` 时，按路径格式提取末尾的 `link_id`：

| 路径格式 | 提取规则 |
|---------|---------|
| `/l/<link_id>` | 文件分享链接 |
| `/folder/<link_id>` | 文件夹分享链接 |
| `/view/l/<link_id>` | 文件预览链接 |

提取后调用 `get_share_info(link_id)` 获取 `file_id` 和 `drive_id`。

> **AIPPT 文档转 PPT 快捷方式**：当用户提供金山文档链接并要求生成 PPT 时，从 URL 提取的 `link_id` 可直接以 `type: "v7_file_id"` 传入 `aippt.doc_outline_options` 和 `aippt.doc_outline`，无需先调用 `get_share_info` 获取 `file_id`。
