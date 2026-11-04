# AI 单文档生成演示文稿

> aippt.execute_round 多轮会话协议：单文档素材（恰好 1 个可传入附件）→ 意图确认 → 大纲就绪直接生成 → merged_url 交付 PPTX

**适用场景**：用户提供了文档（云文档链接 / 本地文件），格式过滤后可传入附件恰好 1 个。

**触发词**：根据文档生成PPT、文档转PPT、文件转PPT、文档生成PPT、把文档做成PPT、根据文档生成演示文稿

## 涉及工具

| 工具 | 服务 | 用途 |
|------|------|------|
| `drive.upload_new_file` | drive | 本地文件先上云，取返回 data.link_url 末尾的短链 id；已有云文档 id 则跳过 |
| `aippt.execute_round` | aippt | 首轮提交文档素材（action=user_message + files，file_type=v7_file_id，不传 session_id） |

## 执行流程

> **前置阅读**：`SKILL.md`，调用接口前先执行命令`kdocs-cli help drive.upload_new_file aippt.execute_round`获取接口参数及约束
> 适用条件：格式过滤后**恰好 1 个可传入文件**（`doc` `docx` `pdf` `pptx` `html` `htm` `txt` `md` `xls` `xlsx` `csv` `et` `otl`，或金山文档云文档）。不可传入的附件先识别内容与用户意图写进 `text`，不要放进 `files`；可传入超过 10 个时用 `ask_user` 请用户精简。

**步骤 1 — 拿到云文档 id**

| 用户给的是 | 做法 |
|---|---|
| 云文档 id 或 `*.kdocs.cn/l/{id}` 链接 | 从链接路径末尾提取 id，直接用，不要再上传 |
| 本地文件路径 | 见 `references/aippt-upload-local-file.md`：`drive.upload_new_file` 上云，取返回 `data.link_url` 末尾的短链 id |

上云失败或拿不到云文档 id 时停止，向用户说明原因，不要发对话请求。

**步骤 2 — 首轮提交（`files` + `file_type=v7_file_id`）**

第一次拿到的云文档 id 一般是 v7 短链，**按 `file_type=v7_file_id` 直传，不要转 v5**；`v7_file_id` 传法下 `file_name` / `file_ext` / `file_size` 可省略。仅当拿到的已是 v5 数字 id 时才改用 `file_type=file_id`，且必须带 `file_name`。

不传 `session_id`；带 `text`（生成意图与要求）、`files`、`generate_config`、`wait_mode=until_round_end`、`timeout_ms=1800000`。记下返回的 `session_id`。

```json
{
  "action": "user_message",
  "text": "基于文档生成 PPT",
  "files": [
    { "file_type": "v7_file_id", "file_id": "ccF3kQxRm9Tz", "file_name": "未命名文档(104).otl" }
  ],
  "generate_config": { "merge_slides": true },
  "wait_mode": "until_round_end",
  "timeout_ms": 1800000
}
```

```bash
kdocs-cli aippt execute-round --file request.json --timeout 1800000
```

**步骤 3 — 按本轮返回决定下一步**

| 本轮返回 | 下一步 |
|---|---|
| `round_status=interrupted`（`interrupt` 有值） | 用 `ask_user` 一次问完本轮 `interrupt.data` 的题目（`options` 传 `[{label, value}]` 对象数组），再以 `action=resume_interrupt` + 上一轮 `session_id` / `interrupt_id` / `interrupt_type` + `interrupt_payload.items` 提交；提交后重新看本表 |
| `phase_hint=outline_ready` 且无待确认项 | 告知大纲（`outline.markdown_excerpt`）后**直接生成**：`action=user_message` + `text=生成PPT` + `wait_mode=until_generate_done`，带 `session_id` |
| `phase_hint=round_end` | 本轮无新大纲，**不要**生成；需要补充信息时再发一轮 |
| `round_status=error` / `timeout` | 见 `SKILL.md` 的权益与超时规则：保留 `session_id` 同会话续聊，不要新建会话、不要杀进程重开 |

**步骤 4 — 交付**

用 `generation.merged_url` 下载 PPTX 并告知本地路径；不要为了拿链接再开一轮对话，不要邀请后续编辑。续轮不必重复提供文件或配置，服务端会继承上一轮上下文。
