# 消息类节点：接收人写法与"发出去了几条"必须核对

**接收人**：写**扁平**的用户 id 列表即可（编译期会自动套一层，chatflow 侧该字段是"列表的列表"）：

```jsonc
{ "receiver_type": "person", "person_id_list": ["用户id"] }
```

⚠️ 接收人必须是**发送方所在公司（团队）的用户 id**。个人账号 id、或不同公司的用户会被**静默过滤**，
结果是节点执行成功但**一条都没发**。

**必须核对发送条数**：消息节点返回的是「发送 [N] 条消息」。

- `N > 0` 才能对用户说「已发送」；
- `N = 0` 说明收件人被全部过滤（或没有命中接收人）——要**如实告知未送达**，不要报成功。

**正文里引用数据：必须带 `|each`**

| 想引用什么 | 写法 |
|---|---|
| **触发**本次运行的那条记录（事件/表格触发） | `${trigger_dataframe.文本\|each,plain}` |
| 前面节点**查到的**记录的列 | `${s1.文本\|each,plain}` |

```jsonc
// 事件触发：引用"触发记录"的列
{ "message_content": { "$template": "「${trigger_dataframe.文本|each,plain}」刚刚更新" } }

// 引用"查找内容"步骤查到的列：按记录**逐行展开**，3 条记录 → 发 3 条消息
{ "message_content": { "$template": "超时订单：${s1.订单号|each,plain}" } }
```

- `|each` 是必须的：不加会**静默渲染成空**（消息照发、节点成功、回读正常，但那段是空的）；
- **不带 `each` 时不会报错**，所以判断"引用生效"的唯一办法是核对消息正文/发送条数，不能只看节点状态；
- 引用**前序步骤**的列时，被引用步骤的 `fields` 底层要的是**列 id**。你可以直接写字段名——
  编译期会自动翻成列 id（`{"fields": ["文本"]}` → `["B"]`）；若某个取值翻不出来，
  保存结果里会有 warning，请按提示核对（写错的字段名会导致下游引用失败）；
- `plain` 是格式，可省略（`|each` 即可）；
- `trigger_dataframe` **只在事件/表格触发下存在**：手动、定时触发的流程里引用它会运行时报
  「选择步骤已删除」——这类流程要引用数据就引用前序步骤的列（`${s1.…}`）；
- 我方会把编辑器默认参数（`group_id_list`/`at_type`/`with_source` 等）自动补齐，
  所以写消息节点时给出**接收人 + 标题 + 内容**就够。

### 正文有 **3w 字符**上限（`|each` 展开富记录极易踩到）

单条消息正文最多 **30000 字符**。用 `|each` 展开**富记录**（长文本 / 富文本 / 图片附件 / 多选等）时，
N 条记录的内容会拼进**同一条**消息里，很容易超限 —— 此时节点**执行失败**并报
`WOA_MSG_TEXT_OVERSIZE`（「消息内容超出30000字符长度限制，请检查配置后再尝试」），
**整条消息都发不出去**。

规避办法（按代价从低到高）：

1. **缩小查找范围**：用筛选条件把命中行数压到少量（例如只筛本次触发的那条记录），别对整表 `each`；
2. **少引用列**：`fields` 只返回真正要用的列，正文里也别引用多余的列（富文本/附件列尤其占字符）；
3. **拆条发送**：把不同列拆到不同消息里分别发。

⚠️ 与"收件人被静默过滤"区分开：**超限是节点失败**（`exec_status != 1`，运行日志里有 `error_type`），
而收件人被过滤是**节点成功但 `发送 [0] 条消息`**。两种都必须如实告知用户，都不能报"已发送"。
# 群机器人渠道（企微 / 钉钉 / 飞书）

要把消息发到**外部群机器人**时用下面三个，全部**已真机投递验证**（prod，节点返回「成功发送消息」）：

| 渠道 | 动作名 | 必填参数 | 备注 |
|---|---|---|---|
| 企业微信群机器人 | `send_wecom_robot_message_v6` | `webhook`、`message_type`、`message_content` | `message_type` 用 `text` |
| 钉钉群机器人 | `send_dingtalk_robot_message_v6` | `webhook`、`sign_token`、`message_type`、`message_content` | 见下方约束 2 |
| 飞书群机器人 | `send_feishu_robot_message_with_webhook_v3` | `webhook`、`message_title`、`message_content` | v3 起 `message_title` 必填 |

⚠️ **WPS 协作没有群机器人渠道**：旧动作 `send_woa_robot_message_v6` 已被平台下架（`online=0`，无后继），
改用 `send_woa_message_v9` 的 `group_id_list`（群发已并入 v9，见上一节）。

约束（每条都踩过或验过，不是推测）：

1. **`webhook` 只能由用户提供**：这是机器人的地址+密钥，**不要编造、不要猜、不要硬编码**。
   用户没给就先问；给不出就说清楚做不了。
2. **钉钉的 `sign_token` 即使走「关键词」模式也必须非空**：模板里它是必填项，传空串会让流程
   运行时判 `10001 流程存在配置错误`（线上实测：空=失败、任意非空占位=成功）。
   关键词模式通常要求正文含该关键词（例如机器人配的是 `wps`，正文就得出现 `wps`）。
3. **`with_source` 不必自己写**：三个渠道都有这个必填参数，默认值是 `1`（消息尾部附上文档名称/链接）。
   不写时服务端会按默认值补上；写 `0` 则不附。
4. **用 online 版本**：上面这些是平台当前提供的版本。老流程里可能出现 `..._v2`、无版本后缀的号
   机器人节点，那些版本平台已下线 —— 读出来会是**只读节点**（见「只读节点」一节），原样带回即可，
   **不要**照抄它们去新建。

示例（企微）：

```jsonc
{ "key": "s1", "action": "send_wecom_robot_message_v6", "label": "通知企微群",
  "params": {
    "webhook": "<用户提供的企微机器人 webhook>",
    "message_type": "text",
    "message_content": { "$template": "订单 ${s2.订单号|each} 已超时" }
  } }
```

> **只验证过**「文本/卡片正文 + 标题 + `with_source`」这条路径。图片（`image_content`）、
> @手机号（`at_mobile_list`）、markdown 模式、卡片按钮**未验证** —— 用户要这些时如实说明，
> 不要凭参数名存在就承诺可用。
# 发送邮件（`send_email`）

```jsonc
{ "key": "s2", "action": "send_email", "params": {
    // ⚠️ 必须先查 auto_sheet.get_context 的 accounts["send_email"] 拿真实账号 id
    "account_id": 10001,          // 从 get_context.accounts 取到的真实账号 id（占位示例）
    "to":      [ "someone@example.com" ],            // 数组；工具会按底层 [][]string 包裹
    "subject": { "$template": "订单 ${s1.订单号} 已通过" },
    "content": "正文（纯文本）" } }
```

- **`account_id` 是硬前置**：底层校验是 `required,min=1` —— 必须是**真实存在的邮箱账号**
  （没有其它消息动作那种 `-1` 默认账号）。可用账号在 `auto_sheet.get_context` 的
  `accounts["send_email"]`（含 id、名称、地址）。
  - 那里如果**没有**账号，说明用户还没授权过邮箱 —— **先让用户在编辑器里添加邮箱账号**
    （拖一个「发送邮件」节点 → 账号下拉 → 添加账号：邮箱地址 / 邮件类型 / 名称 / 密码或授权码），
    **不要编造 id**，工具也会在保存前拦住。
- `subject` / `content` 用 `$template`（可嵌 `${s1.字段}`），工具会落成编辑器要求的写法。
- **没有抄送/密送，也没有附件** —— 底层结构里就没有这些字段。用户要这些时如实说明做不到，
  不要因为"邮件一般都能抄送"就答应。

> **已验证**：真机发信成功（收件人收到 1 封）。
> 发件账号是由工具**现场从账号连接器取**的，所以"怎么拿到 account_id"这条链路也验证过。
# 创建日程（`create_wps_schedule_v1`）

按记录批量建 WPS 日程（提醒）。两个**必须照做**的写法：

```jsonc
{ "key": "s2", "action": "create_wps_schedule_v1", "label": "创建日程",
  "params": {
    "select_schedule_creator": "last_update_user",   // 只能 last_update_user 或 trigger
    "schedule_title": { "$template": "订单 ${s1.订单号|each} 评审" },
    "schedule_notes": "由自动化创建",
    // ⚠️ 起止时间**必须写变量引用**，不能写字面量日期
    "schedule_start_time": { "$var": "var_time_current" },
    "schedule_end_time":   { "$var": "s1.日期列" },
    "schedule_participants": [ "1000000001" ]        // 用户 id 列表（占位示例；服务端会自动套层）
  } }
```

1. **时间只能是变量引用**：可以是系统变量（`${var_time_current}` / `${var_time_recent_day.N}`），
   也可以是**上游记录里的日期列**（`s1.日期列`）。写**字面量日期**（`"2026-09-17 10:00"`，
   无论直接写、包 `$template`、还是改 `biz_type`）都会让运行时判
   「引用的字段存在空值」并报 `PARAMS_ERROR` —— 服务端已会拦住并给出这条提示。
2. **`select_schedule_creator` 只有两个合法值**：`last_update_user`（最后编辑人）与
   `trigger`（触发人）。界面上只列了前者，但源码硬编码认这两个；写别的直接报参数错误。

> **已验证**：真机创建日程成功。**未验证**：`schedule_start_before`（提前提醒）、
> `add_video_meeting`（视频会议）等可选开关的实际效果 —— 不要替用户承诺这些。
