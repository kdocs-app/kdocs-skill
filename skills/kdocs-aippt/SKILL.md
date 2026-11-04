# AI PPT（aippt）（工具路由索引）

本文件包含金山文档 Skill 中 **AI PPT** 相关工具的使用指南。

**适用范围**：支持主题、单文档与多文档生成演示文稿、生成前的本地材料上传、交互式意图确认、同会话续接及生成产物获取。

**不适用范围**：已有幻灯片编辑、与生成材料无关的文件上传、移动及分享。遇到上述域外需求、需要借助其他组件定位文件或准备外部资源，或无法判断请求应由哪个组件处理时，[返回主 Skill](../../SKILL.md)安排，不自行切换子 Skill。

---

## 通用说明

### 场景路由（先定路由，再按对应编排执行）

执行原则：**自上而下依次匹配，命中即执行**。

```
用户请求到达
 │
 ├─ 用户有现成大纲要修改？
 │    └─ 是 → 路由④：修改大纲（同 session 发 user_message，见下文）
 │
 ├─ 用户提供了文档附件？
 │    └─ 先做「附件格式过滤」，只统计可传入文件
 │         ├─ 可传入 0 个 → 路由①：主题生成（text 写入识别到的内容与意图）
 │         ├─ 可传入 1 个 → 路由②：单文档生成
 │         ├─ 可传入 2～10 个 → 路由③：多文档生成（先 import_aidocs）
 │         └─ 可传入 >10 个 → 停止，用 ask_user 请用户精简到 ≤10 份
 │
 └─ 用户仅说了主题，无附件
      └─ → 路由①：主题生成
```

| 路由 | 首轮素材 | 详细编排 |
|---|---|---|
| ① 主题生成 | `text`（主题与要求） | `references/workflows/aippt-topic.md` |
| ② 单文档生成 | `files`（`file_type=v7_file_id` 直传） | `references/workflows/aippt-single-document.md` |
| ③ 多文档生成 | 先 `action=import_aidocs` 取 `aidocs_groups`，再作为首轮素材 | `references/workflows/aippt-multi-document.md` |
| ④ 修改大纲 | 同 `session_id` 下 `action=user_message` + 修改说明 | 本节「路由④」；跳过意图确认，大纲更新后直接生成 |

用户意图模糊（如只说「帮我做个 PPT」）时先问清楚再走路由，不要自行假设。

**路由④ — 修改大纲**：用户已有大纲并要求改章节 / 结构时，在同一 `session_id` 发 `action=user_message`（`text` 写修改要求，`wait_mode=until_round_end`）；**不要**再 `ask_user` 做意图确认。系统返回新版大纲后告知用户要点，再 `wait_mode=until_generate_done` 发起生成（与路由①②③ 大纲就绪后一致）。

### 关键行为

- 一次调用只推进一轮，同一份 PPT 全程复用同一个 `session_id`（首轮不传，之后每轮都带上）
- 意图确认必须用 `ask_user`；大纲就绪（`phase_hint=outline_ready`）后**直接生成**，不设大纲人工闸门
- 成片完成后用 `generation.merged_url` 下载 PPTX，不要为了拿链接再开一轮对话
- 附件单次最多 **10** 份、单文件 < 1024MB；可传入格式见下表

### 附件格式过滤（chat 接口限制）

| 类别 | 可传入后缀 / 形态 |
|---|---|
| 文档 | `doc` `docx` `pdf` `pptx` `html` `htm` `txt` `md` |
| 表格 | `xls` `xlsx` `csv` `et` |
| 在线智能文档 | `otl`；金山文档云文档 id / `*.kdocs.cn/l/{id}` 链接 |

其它后缀（如 `png` `jpg` `webp` `zip` `ppt` `py` `json` `mp4`）**不可传入**：先识别其内容与用户意图，写成简短要点并入 `text`，不要上云、不要放进 `files` / `aidocs_groups`。混传时只把可传入文件交给 `aippt.execute_round`，并在 `text` 里说明「已根据 xx 等不可传入文件识别到：…」。

### 文档来源

| 用户给的是 | 做法 |
|---|---|
| 云文档 id 或 `*.kdocs.cn/l/{id}` 链接 | 从链接路径末尾提取 id，直接用，不要再上传 |
| 本地文件 | `drive.upload_new_file` 上云（见 `references/aippt-upload-local-file.md`），取返回 `data.link_url` 末尾的短链 id |

**可传入格式的附件禁止本地读取 / 摘要正文**：只需拿到云文档 id 后交给 `aippt.execute_round`；多文档还要先 `action=import_aidocs` 取 `aidocs_groups`。

> **工具填参**：下表每行 `<tool>` 选定后须 `kdocs-cli help <tool>`（可批量：`kdocs-cli help aippt.execute_round`）。

## 工具列表

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `aippt.execute_round` | AI PPT 多轮会话生成（主题/文档） |  |

---

## 高频操作流程

| 流程 | 说明 | 详细参考 |
|------|------|----------|
| AI 主题生成演示文稿 | aippt.execute_round 多轮会话协议：主题首轮 → 意图确认 → 大纲就绪直接生成 → merged_url 交付 PPTX | `references/workflows/aippt-topic.md` |
| AI 单文档生成演示文稿 | aippt.execute_round 多轮会话协议：单文档素材（恰好 1 个可传入附件）→ 意图确认 → 大纲就绪直接生成 → merged_url 交付 PPTX | `references/workflows/aippt-single-document.md` |
| AI 多文档生成演示文稿 | aippt.execute_round 多轮会话协议：多文档素材（2～10 个可传入附件）→ import_aidocs 取 aidocs_groups → 意图确认 → 生成 → merged_url 交付 PPTX | `references/workflows/aippt-multi-document.md` |

## 附录

### 错误处理

| 情况 | 说明 |
|------|------|
| `round_status=interrupted` | 意图确认闸门，**不是失败**：用 `ask_user` 问本轮 `interrupt.data` 的题目，再以 `action=resume_interrupt` 提交 |
| `round_status=timeout` / 连接断开 | 保留 `session_id` 同会话续聊；不要新建会话，不要杀进程重开 |
| 没有 AIPPT 生成权益 | 响应顶层 `code=403407100` / `error.code=no_aippt_entitlement`（`msg` 常为 `unAuthGenerate`）时，停止并向用户告知「没有AIPPT生成权益」，**不要重试**；401 先检查认证，其它 403 按 `error.message` 告知，不要冒充权益问题 |
| 多文档 aidocs 导入失败 | 停止操作，向用户说明原因；不要跳过导入直接发对话 |
| 上云失败 | `drive.upload_new_file` 失败时停止，向用户说明原因；没有云文档 id 不要调用 `aippt.execute_round` |
| 云空间已满 | 已生成但云空间不足无法保存到云端时，响应 `data.pptx_url` 含临时下载链接。根据 `message` 判断是个人还是企业空间，按下方对应模板回复用户 |

#### 云空间已满 — 回复模板

根据 `message` 中包含「个人」还是「企业」选择对应模板。将模板中的 `{主题名}` 替换为实际生成的 PPT 主题，`{下载链接}` 替换为 `data.pptx_url`。

**个人云空间已满：**

```
✅ {主题名}幻灯片已生成！

💾 暂存提示：您的云空间已满，文件暂时保存在本地
🔗 临时下载链接（24小时内有效）：[点击下载]({下载链接})

📌 云空间已满，按以下步骤释放空间：
1. WPS客户端 → 我的 → 云空间管理
2. 删除不需要的文件
3. 重新生成PPT，文件将自动同步
```

**企业云空间已满：**

```
✅ {主题名}幻灯片已生成！

💾 暂存提示：您的企业云空间已满，文件暂时保存在本地
🔗 临时下载链接（24小时内有效）：[点击下载]({下载链接})

💡 同步至云端（任选一种）：
1. 自行清理：前往「WPS客户端 → 我的 → 云空间管理」删除不需要的文件
2. 申请扩容：联系企业管理员释放空间
清理/扩容后返回此页面，重新生成PPT，文件将自动同步
```
