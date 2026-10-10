# 文档内容转 Markdown

> 将云文档内容提取为 Markdown 格式文本

**适用场景**：将在线文档（智能文档、Word、PDF）内容转换为 Markdown 输出

**触发词**：转 Markdown、转成 Markdown、导出 MD、导出 Markdown、转换为 Markdown、转为 md、提取为 markdown

- 用户要求将云文档内容转换、导出或提取为 Markdown 格式

**工具链**：`search_files` → `read_file(format=markdown)` → 返回 Markdown 文本

## 涉及工具

| 工具 | 服务 | 用途 |
|------|------|------|
| `search_files` | drive | 定位目标文档（用户提供链接时跳过此步，直接将 url/link_id 传给 read_file） |
| `read_file` | drive | 以 Markdown 格式提取文档内容（format=markdown） |

## 执行流程

> 🎯 **核心工具**：`read_file(format="markdown")` 可将 .otl / .docx / .pdf 内容提取为 Markdown 文本。
> **在线 Markdown 不要走本流程**：读源码用 `read_file` 不传 format（`.md` 默认 markdown），见 `references/workflows/read-cloud-md.md`。

**适用类型**：智能文档（.otl）、Word（.docx）、PDF（.pdf）
**不适用**：表格（.xlsx / .ksheet）、多维表格（.dbt）、演示文稿（.pptx）

```
步骤 1: 定位文档
        - 用户给文件名/关键词 → search_files(keyword="文档名") 获取 file_id
        - 用户给链接 → 直接提取 url 或 link_id

步骤 2: read_file(file_id=..., format="markdown") 或 read_file(url=...) / read_file(link_id=...)
        → status=ok 时取 data.content
        → status=pending 时用本次 task_id 和原参数获取文档全量内容

步骤 3: 将 Markdown 文本直接返回给用户
```

> ⚠️ **"导出为 MD 文件"** 分两种意图：
> - **仅要 Markdown 文本**（otl/docx/pdf 转换结果）：将 `read_file(format=markdown)` 的 `data.content` 直接返回给用户，或由其自行保存为本地 `.md`。
> - **要在云盘新建 `.md` 文件**：本地文件走 `references/workflows/upload-md-raw.md`；对话内正文走 `references/workflows/create-cloud-md.md`（均含图片 kdocsmd 转存）。
> - **已有在线 Markdown 导出到本地文件**：`references/workflows/download-cloud-md.md`。
