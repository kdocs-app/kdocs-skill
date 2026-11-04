# 下载知识库文件到本地

> 从知识库下载文件到本地

**适用场景**：帮我下载知识库里的某个文件

**触发词**：下载、导出

- 用户要求从知识库下载文件

**工具链**：`kwiki.list_items` → 按文件类型选择下载方式

## 涉及工具

| 工具 | 服务 | 用途 |
|------|------|------|
| `kwiki.list_items` | kwiki | 定位目标文件，获取 file_id 和 link_id |

## 执行流程

**流程**：

1. `kwiki.list_items(kuid=目标目录kuid)` 定位目标文件，获取 `file_id`、`link_id`、`drive_id`
2. 根据文件类型选择下载方式：

**普通文件（docx/pptx/pdf/图片等）**：
1. 使用 `wps.export` 等导出工具获取带签名的下载 URL（`link_id` 来自 `kwiki.list_items`）
2. `curl.exe -L -o "文件名" "签名URL"` 下载

**智能文档（doc_type="o"）**：`wps.export` 不支持直接导出，无特殊情况，默认转换成Markdown格式：
- **Markdown** → `read_file(file_id=...)`；`status=pending` 时用本次响应的 `task_id` 和原参数获取结果，返回完整内容后，将 `data.content` 保存为 `.md` 文件

**快捷方式文件（type="shortcut"）**：通过 `search_files` 搜索原始文件名找到源文件，再用源文件的 `link_id` 走上述通用流程。

> 注意：`download_file` 返回的 URL 需登录态，无法直接 curl。始终优先使用 `wps.export` 获取带签名的下载 URL。受保护文件（SecureDocumentError / forbidProtectedFile）所有导出接口均无法操作，需提示用户。

调用接口前先执行以下命令获取接口参数及约束：
`kdocs-cli help kwiki.list_items`
