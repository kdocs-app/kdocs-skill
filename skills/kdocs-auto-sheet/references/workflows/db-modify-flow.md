# 修改已有自动化流程

> 安全地修改一条已有自动化流程：先读现状 → 增量改 → 带 revision 保存 → 回读确认（含 opaque 节点硬规则）

**适用场景**：把我那条「订单超时提醒」的时间改成每天 10 点；或者给流程再加一个通知节点

**触发词**：改一下自动化、修改流程、调整时间、再加一个节点、把那个流程改成

- 用户指的是**已存在**的流程（不是新建）

**工具链**：`auto_sheet.list_flows` → `auto_sheet.get_flow` → `auto_sheet.save_flow`(update) → `auto_sheet.validate_flow`

## 前置条件

- 必须先 auto_sheet.list_flows 定位到唯一的那条流程

## 涉及工具

| 工具 | 服务 | 用途 |
|------|------|------|
| `auto_sheet.list_flows` | auto_sheet | 列出流程，确认要改的是哪一条（把流程名回显给用户核对） |
| `auto_sheet.get_flow` | auto_sheet | 读当前完整配置（DSL）与 revision、opaque_steps |
| `auto_sheet.save_flow` | auto_sheet | 基于读到的 DSL 做增量修改后保存 |
| `auto_sheet.validate_flow` | auto_sheet | 保存后校验 |
| `auto_sheet.set_trigger_status` | auto_sheet | 按需开启/关闭（改了配置不代表已生效） |

## 执行流程

**典型说法**：「把我那条订单超时提醒改成每天 10 点」「给那个流程再加个通知节点」
「那条自动化不提醒了，帮我看看」「把接收人换成我」

**为什么必须"先读后写"**

流程的底层配置里混着服务端生成的表达式包装、节点 ID 与展示结构，其中一部分（变量引用、
字段值映射、分支条件等）无法在 DSL 里无损展开，`auto_sheet.get_flow` 会把它们原样放在
`{"$raw": …}` 里并把节点标进 `opaque_steps`。保存时服务端会把这些节点与当前配置**逐字节比对**：

| 你的操作 | 服务端行为 |
|---|---|
| 原样带回 `$raw` 节点 | 通过 |
| 改一个字的 `$raw` 内容（连 label 也不行） | 拒绝，`kind=unsafe_operation` |
| 删掉 `$raw` 节点 | 拒绝，`kind=unsafe_operation` |

所以**正确做法**是：`get_flow` → 只改需要改的部分 → 整份 `steps` 原样回传。

**标准动作**

```
1. auto_sheet.list_flows                       → 定位流程 ID（回显名称给用户确认）
2. auto_sheet.get_flow(flow_id)                → flow.steps / revision / opaque_steps
3. 在 flow 上做最小修改（例如只改 trigger.cron.time_formats）
4. auto_sheet.save_flow(action="update", flow_id=…, revision=…, flow=<改后的>)
5. auto_sheet.validate_flow(flow_id)           → ok=true
6. 如需要：auto_sheet.set_trigger_status       → 开启/关闭
```

**只改触发器时间的最小示例**（其余字段全部沿用 `get_flow` 的返回）：

```jsonc
{
  "action": "update",
  "flow_id": "42",
  "revision": "<get_flow 返回的 revision>",
  "flow": {
    "name":       "<沿用原值>",
    "func_desc":  "<沿用原值>",
    "trigger":    { "type": "cron", "cron": { "cron_type": "every_day", "time_formats": ["10:00"] } },
    "steps":      [ /* 原样复制 get_flow 的 steps，含 $raw 节点 */ ]
  }
}
```

**节点 key 与引用**

- `steps[].key` 是节点在流程内的唯一标识（`s1`/`s2`…），`{"$var": "s1.字段名"}` 就靠它指向。
- 改流程时**保持原有 key 不变**；插入新节点用新的、不与现有重复的 key。
- 如果改动导致引用失效（比如删了被引用的节点），`validate_flow` 会报 `unknown_reference`。

**revision 是乐观锁**

保存时带上 `get_flow` 返回的 `revision`。若期间有别人改过，服务端会拒绝并告诉你当前值——
这时要**重新 `get_flow`**再改，不要为了保存成功而丢掉 `revision`（那会覆盖别人的改动）。

**改了要不要重新开启？**

- 流程原本未启用：保存后仍是草稿，需要时再 `set_trigger_status` 开启。
- 流程原本已启用：`save_flow` 的 `enabled` 决定保存后的状态；未显式传 `enabled` 会**沿用当前状态**，
  所以改完要按用户意图决定是否再次开启，并明确告知当前状态。

**失败处置**

| 现象 | 原因 | 处置 |
|---|---|---|
| `unsafe_operation` | 动了 `opaque_steps` 里的节点 | 重新 `get_flow`，把该节点原样带回 |
| `unsafe_operation`（删节点） | 删了 `$raw` 节点 | 保留该节点；确需删除请让用户在界面里操作 |
| revision 冲突 | 流程被他人改过 | 重新 `get_flow` 后再改 |
| `unknown_reference` | `$var` 的 key 或字段名写错 | 核对 `get_flow` 的 `steps[].key` 与 `list_actions` 的 `returns` |
| `verified=false` | 服务端回读与期望不一致 | 如实告知用户未确认成功，并用 `get_flow` 核对 |

## 注意事项

- **auto_sheet.list_flows**：候选为空或有名称相近的多条时，列给用户挑选，不要擅自选一条
- **auto_sheet.get_flow**：这一步是**硬性前置**；不读就改会被服务端拒绝，且可能覆盖别人的改动
