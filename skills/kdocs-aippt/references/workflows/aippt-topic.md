# AI 主题生成演示文稿

> aippt.execute_round 多轮会话协议：主题首轮 → 意图确认 → 大纲就绪直接生成 → merged_url 交付 PPTX

**适用场景**：用户只给出主题描述（如「做一个关于 AI 趋势的 PPT」），或附件经格式过滤后可传入数量为 0。

**触发词**：生成PPT、生成演示文稿、做PPT、做个PPT、生成幻灯片、主题生成PPT、做一份PPT、按主题生成

## 涉及工具

| 工具 | 服务 | 用途 |
|------|------|------|
| `aippt.execute_round` | aippt | 首轮提交主题与要求（action=user_message，不传 session_id，wait_mode=until_round_end） |

## 执行流程

> **前置阅读**：`SKILL.md`，调用接口前先执行命令`kdocs-cli help aippt.execute_round`获取接口参数及约束
> 一次调用只推进一轮，同一份 PPT 全程复用同一个 `session_id`。

**步骤 1 — 首轮提交需求**

不传 `session_id`（新建会话）；带 `text`（主题 + 受众 / 页数 / 风格等要求）、`generate_config`、`wait_mode=until_round_end`、`timeout_ms=1800000`。记下返回的 `session_id`。

请求 JSON：

```json
{
  "action": "user_message",
  "text": "做一个关于长颈鹿的儿童科普 PPT，面向小学生，约 10 页",
  "generate_config": { "merge_slides": true },
  "wait_mode": "until_round_end",
  "timeout_ms": 1800000
}
```

```bash
kdocs-cli aippt execute-round --file request.json --timeout 1800000
```

**步骤 2 — 按本轮返回决定下一步**

| 本轮返回 | 下一步 |
|---|---|
| `round_status=interrupted`（`interrupt` 有值） | 走步骤 3：用 `ask_user` 问用户 |
| `phase_hint=outline_ready` 且无待确认项 | 走步骤 4：告知大纲并直接生成 |
| `phase_hint=round_end` | 本轮无新大纲，**不要**生成；需要补充信息时再发一轮 `action=user_message` / `continue_text` |
| `round_status=error` / `timeout` | 停止或续聊，见 `SKILL.md` 的权益与超时规则：保留 `session_id` 同会话续聊，不要新建会话、不要杀进程重开 |

**步骤 3 — 意图确认（人工闸门，必须用 `ask_user`）**

1. 题目只取本轮返回的 `interrupt.data`（`questions` / `items`），不得自造用途、受众、页数
2. **一次问完**：所有题放进同一次 `ask_user`；`options` 必须是 `[{label, value}]` 对象数组，服务端返回字符串数组时先转换
3. 拿到答复后组 `interrupt_payload.items`（`choice` 传选中项字符串数组，`text` 传 `text_input`），以 `action=resume_interrupt` 提交，并回传本轮的 `session_id` / `interrupt_id` / `interrupt_type`
4. 提交后回到步骤 2：再次返回 `interrupt` 就继续问；无待确认项且 `phase_hint=outline_ready` 就进入步骤 4

> 用户说「走默认」时仍要用 `ask_user` 列出拟提交项让用户确认，不能直接代填提交。

**步骤 4 — 告知大纲并立即生成（无大纲闸门）**

1. 依据本轮 `outline.markdown_excerpt` 转述章及主要页题（不全文粘贴、不只报章名），并说明「开始生成，可能需要几分钟」
2. **不要**再问「是否确认大纲」；直接发起新一轮：`action=user_message` + `text=生成PPT` + `wait_mode=until_generate_done` + 上一轮的 `session_id`
3. 等到 `generation.state=success`（耗时较长，`timeout_ms=1800000`）

**步骤 5 — 交付**

用 `generation.merged_url` 下载 PPTX，把本地路径告知用户；不要为了拿链接再开一轮对话，不要邀请继续编辑配色 / 增删页。
