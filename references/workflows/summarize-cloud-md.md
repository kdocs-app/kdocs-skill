# 在线 Markdown 总结与问答

> 读取在线 Markdown 正文后，总结要点或按用户问题作答

**适用场景**：用户要对一份在线 .md 做总结、提炼或问答

**触发词**：总结 md、md 总结、在线 md 总结、根据 md 回答、md 里有什么、这份 md 结论

- 对在线 Markdown 做总结、提炼或问答

**工具链**：`search_files`（可选）→ `read_file` → Agent 总结/问答

## 涉及工具

| 工具 | 服务 | 用途 |
|------|------|------|
| `search_files` | drive | 用户未给链接/file_id 时按文件名搜索 |
| `read_file` | drive | 取 Markdown 源码（不传 format） |

## 执行流程

> 🎯 **核心路径**：先 `read_file` 取在线 Markdown 正文，再由 Agent 总结或问答。**没有**单独的「总结 md」API。

**用户说法示例**：「总结一下这份 md」「这份在线 md 里有什么结论」「根据这个 md 回答…」。

```
步骤 1: 定位文档（定位方式同「仅读取正文」`references/workflows/read-cloud-md.md`）
        - 用户给链接 → read_file(url=...) 或 read_file(link_id=...)
        - 已有 file_id → read_file(file_id=...)
        - 仅有文件名/关键词 → search_files(keyword=...) 取 file_id

步骤 2: read_file（不传 format；`.md` 默认 markdown 源码）
        → status=ok 时取 data.content
        → status=pending 时原参数 + task_id 再次调用

步骤 3: Agent 基于正文作答
        - 用户要「总结/摘要/要点」→ 结构化输出核心结论（3–7 条为宜），可附简短导语
        - 用户提具体问题 → 仅依据正文回答；正文未提及处明确说明「文档中未找到」
        - 默认**不**新建云文档；用户明确要求「写成新文档/保存总结」时再走 create_file_with_content
        - 回答中可注明来源文件名（data.name）或链接
```

定位与读正文细节见「仅读取正文」`references/workflows/read-cloud-md.md`；参数见 `references/drive/read_and_download.md` 中 `read_file`。
