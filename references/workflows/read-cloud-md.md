# 读取在线 Markdown

> 读取在线 Markdown 文件正文（Markdown 源码）

**适用场景**：用户提供 md 链接、file_id，或按文件名搜索云盘 .md 后读取（不保存本地文件）

**触发词**：读 md、打开 md、在线 md、markdown 链接、读一下 md

- 读取在线 Markdown 正文，不要求保存到本地磁盘

**工具链**：`search_files`（可选）→ `read_file` → 返回正文

## 涉及工具

| 工具 | 服务 | 用途 |
|------|------|------|
| `search_files` | drive | 用户未给链接/file_id 时，按文件名或关键词搜索 .md |
| `read_file` | drive | url / link_id / file_id 三选一；`.md` 默认返回源码正文 |

## 执行流程

> 🎯 **核心工具**：`read_file`。在线 Markdown 返回的正文即 Markdown 源码，**不传** `format=markdown`。
> 默认含 `kdocsmd://` 协议串；**不**自动变成可下载 https。
> **导出到本地文件**（含图片临时链或 assets 目录）→ 「导出到本地」`references/workflows/download-cloud-md.md`。
> 在线 Markdown 图片协议说明 → `references/markdown.md`。

**用户说法示例**：「打开这个 md 链接」「读一下这份在线 md」。

```
步骤 1: 定位文档（url / link_id / file_id / search_files）

步骤 2: read_file（不传 format）
        → status=ok 时取 data.content
        → status=pending 时原参数 + task_id 再次调用

步骤 3: 将正文返回给用户（默认不新建、不覆盖云盘文件）
```

参数见 `references/drive/read_and_download.md` 中 `read_file`。
