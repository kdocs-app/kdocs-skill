# 云 Word 转智能文档

> 云盘 .docx/.doc 正文转为 .otl，清洗 Word 域代码后再写入

**适用场景**：用户要把 Word/文字文档转成智能文档、otl，或「word 转 otl」

**触发词**：word 转 otl、docx 转 otl、转成智能文档、word转智能文档、文字文档转 otl、doc 转 otl

- 源文件为云盘 .docx 或 .doc，目标为新建 .otl（非本地 md 上传）

**工具链**：`read_file`(docx) → 清洗 Markdown → `create_file_with_content`(otl) → `read_file` 验证（禁控制符）

## 涉及工具

| 工具 | 服务 | 用途 |
|------|------|------|
| `read_file` | drive | 读取源 Word（format=markdown 或默认）；pending 时带 task_id 轮询至 status=ok |
| `create_file_with_content` | drive | 仅传入**清洗后**的 Markdown；name 不带后缀，file_extension=otl |
| `otl.insert_content` | otl | content 超过单次上限时：先建空 otl，再分段写入 |

## 执行流程

> 🎯 **禁止**将 `read_file` 对 `.docx` 返回的 `content` **原样**写入 `create_file_with_content`（`file_extension=otl`）。含自动目录、TOC、HYPERLINK 等域的 Word 抽取结果常带控制符（如 `\u0003` `\u0013` `\u0014` `\u0015`），直接灌入 OTL 会在界面显示乱码。

**与「本地 md 上传 otl」的区别**：本地 `.md` 走 `references/workflows/md-upload-to-otl.md`；本流程源为**云 Word**。

```
步骤 1: read_file
        - url / link_id / file_id 指向源 .docx 或 .doc
        - 取 data.content（字符串）；status=pending 时按 task_id 轮询

步骤 2: 清洗（必须，脚本或等价逻辑）
        - 删除 Word 域块：从 \u0013 到配对 \u0015 的整段（含 TOC、HYPERLINK 等）
        - 删除残留 \u0003 \u0013 \u0014 \u0015
        - 保留 \t \n \r；合并连续空行（≥4 行压为 3 行）
        - 将清洗结果写入工作区 UTF-8 文件（如 body.md），勿在对话里贴全文

步骤 3: create_file_with_content
        - name: 用户文档名（无后缀）
        - file_extension: otl
        - content: 清洗后的 Markdown
        - 或：`kdocs-cli drive create-file-with-content --name … --file-extension otl --content-file ./body.md`

步骤 4: 正文超过单次上限
        - 同 md-upload-to-otl：空 otl + otl.insert_content 分段

步骤 5: 写后验证（必须，不可仅用章节标题子串）
        - read_file 新建 otl 的 link_url 或 file_id
        - 若 content 仍匹配 /[\u0000-\u0008\u000B\u000C\u000E-\u001F]/ 或含字面量「\u0013TOC」「HYPERLINK \\l」→ 视为失败，回到步骤 2 加强清洗或告知用户目录域需手工整理
        - 通过后再回复用户 link_url
```

**清洗参考实现**（stdin 读入、stdout 写出，≤15 行可内联执行）：

```javascript
let s = require('fs').readFileSync(0, 'utf8');
s = s.replace(/\u0013[\s\S]*?\u0015/g, '');
s = s.replace(/[\u0003\u0013\u0014\u0015]/g, '');
s = s.replace(/\n{4,}/g, '\n\n\n').trim();
process.stdout.write(s);
```

参数以 `kdocs-cli help drive.read_file` / `drive.create_file_with_content` 为准。
