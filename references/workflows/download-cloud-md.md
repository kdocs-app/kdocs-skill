# 在线 Markdown 导出到本地

> 将在线 Markdown 导出为本地文件；图片默认内联 HTTPS 临时链，也可保存到本地 assets 目录

**适用场景**：用户要把在线 Markdown 下载/保存到本地，且需处理 kdocsmd:// 或内嵌图片

**触发词**：下载 md、导出 md、在线 md 保存到本地、拉取 markdown、带图下载、图片保存到本地、assets

- 目标为在线 Markdown，需保存到用户本地或工作区

**工具链**：`read_file` → `download_attachment`（逐张 `kdocsmd://`）→ 写本地 .md`

## 涉及工具

| 工具 | 服务 | 用途 |
|------|------|------|
| `search_files` | drive | 用户未给链接/file_id 时按文件名搜索 |
| `read_file` | drive | 取 Markdown 源码（不传 format）；在线 Markdown 勿指望 enable_upload_medias 解析 kdocsmd:// |
| `download_attachment` | drive | 对每个 kdocsmd://{attachment_id} 取 download_url；inline_https 替换正文，local_assets 再 GET 保存到本地 |

## 执行流程

> 🎯 **与「仅读取正文、不保存本地文件」的区别**：本流程产出**本地文件**（及可选 `*.assets/` 目录），不只返回正文字符串（仅读正文见 `references/workflows/read-cloud-md.md`）。
> 图片导出两种模式见 `references/markdown.md`。

**用户说法示例**：「把这份在线 md 下载到本地」「导出 md 带图片」「离线能看的 md」。

### 导出模式

| 模式 | 何时用 | 本地产物 |
|------|--------|----------|
| `inline_https`（**默认**） | 「下载/导出 md」 | 单个 `.md`，图为 `![](https://临时链...)` |
| `local_assets` | 「图片也下载」「离线」「assets」 | `.md` + `{主文件名}.assets/*.png` |

意图不明时询问：「临时 https 链接（约 10 分钟）还是图片保存到本地文件夹？」

### 流程 A：`inline_https`（默认）

```
步骤 1: 定位在线 Markdown（定位方式同「仅读取正文」流程）

步骤 2: read_file(file_id)
        - 不传 format
        - **不要**指望 enable_upload_medias 把 kdocsmd:// 变成 https（在线 Markdown 仍返回 kdocsmd 协议）
        - pending 时带 task_id 轮询

步骤 3: 对 content 中每个 kdocsmd://{attachment_id}（占位 SK 可跳过）：
        download_attachment(file_id, attachment_id=attachment_id)
        → 将 kdocsmd://… 替换为响应 **download_url**（不是 download_file 的 data.url）

步骤 4: 写入本地 xxx.md

步骤 5: 告知用户图片 https 约 10 分钟有效
```

### 流程 B：`local_assets`

```
步骤 1–2: read_file 取源码（不传 format）

步骤 3: 对每个 kdocsmd://{attachment_id}（及需保存到本地的图）：
        download_attachment → GET **download_url**（沙箱须 wps_sid，见 download_file 保存说明）
        → 保存到与 md 同目录的 {主文件名}.assets/

步骤 4: 正文改为 ![]( ./{主文件名}.assets/文件名 )
        单张失败保留原 src + warnings

步骤 5: 写入本地 xxx.md，说明需保留 md 与 assets 目录一起移动
```

仅读取正文、不保存文件 → `references/workflows/read-cloud-md.md`。
