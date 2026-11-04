# AI 多文档生成演示文稿

> aippt.execute_round 多轮会话协议：多文档素材（2～10 个可传入附件）→ import_aidocs 取 aidocs_groups → 意图确认 → 生成 → merged_url 交付 PPTX

**适用场景**：用户提供了 2～10 份文档素材，需要合并生成一份演示文稿。

**触发词**：多文档生成PPT、多份文档生成PPT、合并文档生成PPT、综合文档生成PPT、多个文件生成PPT、几份报告生成PPT

## 涉及工具

| 工具 | 服务 | 用途 |
|------|------|------|
| `drive.upload_new_file` | drive | 本地文件先逐个上云，取返回 data.link_url 末尾的短链 id；已有云文档 id 则跳过 |
| `aippt.execute_round` | aippt | action=import_aidocs 导入云文档，用返回的 aidocs_groups 作为首轮素材 |

## 执行流程

> **前置阅读**：`SKILL.md`，调用接口前先执行命令`kdocs-cli help drive.upload_new_file aippt.execute_round`获取接口参数及约束
> 适用条件：格式过滤后 **2～10 个可传入文件**；可传入超过 10 个时先停止，用 `ask_user` 请用户精简到 ≤10 份，不要擅自截取前 10 个；不可传入的附件先识别内容与意图写进 `text`，不要导入。

**步骤 1 — 拿到每份文档的云文档 id**

| 用户给的是 | 做法 |
|---|---|
| 云文档 id 或 `*.kdocs.cn/l/{id}` 链接 | 逐个从链接路径末尾提取 id，直接用，不要再上传 |
| 本地文件路径 | 逐个 `drive.upload_new_file`（`name` 带后缀 + `content_base64`）上云，取返回 `data.link_url` 末尾的短链 id |

收集全部 `file_id` + `file_name`；上云失败时停止，向用户说明原因。

**步骤 2 — 导入 aidocs（`action=import_aidocs`）**

多文档**不能**把云文档 id 直接当作首轮的 `files`。先把上一步的云文档 id 与文件名放进 `files` 提交导入，从返回结果取 `aidocs_groups`：

```json
{
  "action": "import_aidocs",
  "files": [
    { "file_id": "cc5B9QRQul0U", "file_name": "报告一.docx" },
    { "file_id": "EEf8mN2pQr1T", "file_name": "报告二.docx" }
  ],
  "timeout_ms": 1800000
}
```

`file_id` 传实际拿到的云文档 id（v7 短链或已是 v5 数字 id 均可，服务端会解析成 v5 数字 id），不需要传 `file_type`。导入失败时停止操作并向用户说明原因，**不要**跳过导入直接发对话请求。

**步骤 3 — 首轮提交（`aidocs_groups`）**

把上一步返回的 `aidocs_groups` **原样**填入首轮；不传 `session_id`；带 `text`（合并意图与要求）、`generate_config`、`wait_mode=until_round_end`、`timeout_ms=1800000`。记下返回的 `session_id`。

```json
{
  "action": "user_message",
  "text": "综合两份报告生成 PPT",
  "aidocs_groups": [
    { "group_id": "从导入返回的 folder_id", "file_ids": ["从导入返回的 file_id"], "filenames": ["报告一.docx"] },
    { "group_id": "从导入返回的 folder_id", "file_ids": ["从导入返回的 file_id"], "filenames": ["报告二.docx"] }
  ],
  "wait_mode": "until_round_end",
  "timeout_ms": 1800000
}
```

```bash
kdocs-cli aippt execute-round --file request.json --timeout 1800000
```

**每份文档独立一组**：导入返回的 `aidocs_groups` 与首轮传入的都遵循「一份文档 = 一个 group」——`group_id` 取该文档的 `folder_id`，`file_ids` / `filenames` 均为**单元素数组**。2 份文档就是 2 个 group，不要把多份文档合并进同一个 group。

**步骤 4 — 按本轮返回决定下一步**

| 本轮返回 | 下一步 |
|---|---|
| `round_status=interrupted`（`interrupt` 有值） | 用 `ask_user` 一次问完本轮 `interrupt.data` 的题目（`options` 传 `[{label, value}]` 对象数组），再以 `action=resume_interrupt` + 上一轮 `session_id` / `interrupt_id` / `interrupt_type` + `interrupt_payload.items` 提交；提交后重新看本表 |
| `phase_hint=outline_ready` 且无待确认项 | 告知大纲（`outline.markdown_excerpt`）后**直接生成**：`action=user_message` + `text=生成PPT` + `wait_mode=until_generate_done`，带 `session_id` |
| `phase_hint=round_end` | 本轮无新大纲，**不要**生成；需要补充信息时再发一轮 |
| `round_status=error` / `timeout` | 见 `SKILL.md` 的权益与超时规则：保留 `session_id` 同会话续聊，不要新建会话、不要杀进程重开 |

**步骤 5 — 交付**

用 `generation.merged_url` 下载 PPTX 并告知本地路径；不要为了拿链接再开一轮对话，不要邀请后续编辑。续轮不必重复传 `aidocs_groups`。
