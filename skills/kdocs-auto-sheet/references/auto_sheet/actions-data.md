# 筛选条件与写入内容的写法

这两处**形状很挑，写错会静默失效**（保存成功、`dry_run` 也 ok，但流程跑不起来或写不进去）。

### 筛选条件（`search_range.filter`）

```jsonc
"search_range": {
  "filter_type": "filter",          // 有条件时必须是 filter；all=全表、ever_result=用上游结果
  "filter": { "mode": "AND",        // AND / OR
              "criteria": [ { "field": "状态", "op": "eq", "value": "待处理" } ] }
}
```

- `field` 写**字段名**即可（编译期翻成列 id；也可直接写列 id）；
- `op` 用**小写词**：`eq` / `ne` / `contains` / `not_contains` / `starts_with` / `ends_with` /
  `gt` / `gte` / `lt` / `lte` / `between` / `in` / `empty` / `not_empty`；
- 同一字段**只能写一条**条件：多值用 `op: "in"` + `values`，区间用 `op: "between"` + 两个值；
- 写了 `filter_type: "filter"` 却没有条件会被**直接拒绝**（底层会当成全表，界面上又会显示
  「满足条件的记录」却没有筛选行）；
- **别**写 `{"field": …, "op": "==", "value": …}`：`==`/`=` 这类符号不是 DSL 词，底层要的是
  枚举操作符，会被拒（`选择的字段不存在` / `未知枚举值`）——DSL 只写 `eq` 等小写词，由工具翻译成
  平台枚举即可。

### 写入内容（`key_values`）

```jsonc
"key_values": { "备注": "已处理" }                        // 字段名 → 值
"key_values": [ { "field": "备注", "value": { "$var": "s1.负责人" } } ]  // 值支持 $var/$template
```

- 值可以是字面量，也可以是 `{"$var": …}` / `{"$template": …}`（引用前序步骤或触发记录）；
- 字段名会自动翻成列 id 并带上字段类型；
- **只读字段**（公式 / 自动编号 / 查找引用 / 创建时间 / 最后修改时间）会被**拒绝**，
  不要试图写入；
- 同一字段写两次会被拒绝（后写的会静默覆盖前一个）。

**条目形态与界面一致（2026-09-17 实测；不用你操心，但要知道回读长什么样）**：
底层每条 `key_values` 条目 = **该列在表结构里的定义对象原文 + `__sheetId` + `extraConfig` + `value`**
（链接列实测 27 键）。工具会自动这么写，你照上面的 DSL 写即可。

回读（`get_flow` / `pull`）时：能从表结构复现的元数据会被丢掉，只给你 **`{ "字段名": 值 }`**；
少数**无法复现**的键（如界面开了「仅设置字段样式」→ `extraConfig.styleOnly = true`）
会挂在条目级的 **`$meta`** 上：

```jsonc
"key_values": [
  { "field": "备注", "value": "已处理",
    "$meta": { "extraConfig": { "fontColor": null, "fullColor": null, "styleOnly": true } } }
]
```

- **把 `$meta` 原样带回去**（不要删、不要改）——它是界面配置，丢了会悄悄改掉用户的字段样式设置；
- 只有"带 `$meta`"的字段才会落成列表形态，其余情况仍是简洁的 `{字段: 值}`。

⚠️ **链接 / 多选 / 人员 / 附件列暂不能逐字段编辑**：界面给这些列写的值是 `flat_array` 表达式
（`{"__exp_kind__":"flat_array","__exp_value__":[…]}`），属工具未建模的值形态，因此这类字段的
`key_values` 会**整条退成 `$raw`**（原样保留）。要改就整条重写，不要在 `$raw` 里挑字段改。

### 分支条件（`branch` / `sub_branch` 的 `condition`）

> **这两个节点也在工具接口里**：`auto_sheet.get_context` 的 `structural_actions` 小节
> （**默认返回**，不必显式 include）会给出它们各自的必备字段与条件写法；
> `auto_sheet.list_actions` 的结果里也带 `structural: true` 标注。
> 它们**不在"可选动作"白名单**里 —— 分支是 `children` 表达的控制结构，不是"填几个参数"的动作，
> 所以别去白名单里找它们，也别以为不支持。

```jsonc
{ "key": "b1a", "action": "sub_branch", "label": "命中预警",
  "condition": { "mode": "AND", "criteria": [
      { "left": { "$var": "s2.__record_count" }, "op": "gt", "right": 0, "biz_type": "number" } ] },
  "children": [ /* 命中时才执行的步骤 */ ] }
```

- `left` / `right` 可以是字面量，也可以是 `{ "$var": … }` 引用（引用前序步骤或触发记录）；
- `op` 可用：`eq` / `ne` / `gt` / `gte` / `lt` / `lte` / `contains` / `not_contains` /
  `starts_with` / `ends_with` / `empty` / `not_empty` / `between` / `in`；
- **数值/日期比较要写 `biz_type`**（如 `number` / `date`），否则会按字符串比较得出错误结论；
- `criteria` 为空 = **总是命中**（这是底层语义，不是"没配"）；
- 底层要的是带类型前缀的 `operator`（如 `number_gt`），工具会按 `op` + `biz_type` 自动拼；
- 一个 `branch` 下可以有多个 `sub_branch`，各自独立判断；
  **兜底分支**写成"与前面条件相反"或留空条件。
- 实测口径：命中的子分支 `result=符合筛选条件` 且其 `children` 真的执行；
  未命中的 `result=不符合筛选条件` 且子节点不执行 —— 开启后可用运行日志核对。

### @人 与卡片按钮（`send_woa_message_v9`）

```jsonc
{ "key": "s2", "action": "send_woa_message_v9", "params": {
    "receiver_type": "person", "person_id_list": ["用户ID"],
    "at_type": "at_person",              // 不提醒=at_none / 提醒所有人=at_all / 提醒指定人=at_person
    "at_person_list": ["要@的用户ID"],     // 仅 at_person 时需要（也是 list-of-lists，工具会包）
    "message_title": "标题",
    "message_content": "正文",            // 字面量即可；要插值就写 ${s1.字段} 或 {"$template": …}
    "message_buttons": { "button_type": "url",
                         "buttons": [ { "title": "查看详情", "url": ["https://…"] } ] } } }
```

- 标题/正文**写普通字符串就行**（工具会按编辑器要求的 `template_string` 落库）；
  注意：如果手写成 `{"$raw": …}` 且落成 `original`+字符串，流程**能保存但永远开启不了**。
- `message_buttons.button_type`：`url`（跳转链接卡片）/ `interactive`（交互按钮）/
  `none`。`interactive` 需要界面拖拽生成的按钮流程（`actions` JSON 串），
  手工构造不出来时请用 `$raw` 照抄界面的配置。
- 发送对象 `person_id_list`、`group_id_list` 与 `@` 的 `at_person_list`
  都是"列表的列表"，**直接给一维数组即可**，工具会按底层要求包裹。

#### 卡片里的交互小组件（`interactive_widgets`，语法糖）

要让卡片上出现**可点击的按钮**，用 `interactive_widgets` —— **只写"要哪些按钮"**，
底层的两个参数（`interactive_Widgets_Switch` 与 `Interactive_Widget_Composer`）由服务端生成：

```jsonc
{ "key": "s2", "action": "send_woa_message_v9", "params": {
    "receiver_type": "person", "person_id_list": ["用户ID"],
    "at_type": "at_none",
    "message_title": "审批提醒", "message_content": "有一条待处理",
    "interactive_widgets": {
      "only_allow_one_button_click": false,
      "widgets": [
        // 需要"打开某个文档/记录"就填变量；固定链接也可以直接写字面量
        { "type": "BUTTON", "title": "打开详情", "url": "${s1.记录链接}" }
      ] } } }
```

- `widgets` 里每项：`type`（目前只有 `BUTTON` 有真实样例）、`title`（按钮文字）、
  `url`（跳转地址，**支持 `${...}` 变量**，例如指向某条记录的链接）、
  `title_after_click`（点击后显示的文字，可省）。
- **不写 `url` 会被拦下**（跳转类按钮没有地址等于点了没反应）；不写或 `widgets` 为空 =
  不开交互组件。
- ⚠️ **不要手写那两个底层参数**：`Interactive_Widget_Composer` 的结构非常反直觉
  （真实的组件列表是内层 `__exp_value__` 的**兄弟键**），写错的表现是
  "保存成功、卡片上没有按钮"。既有流程里它们会以 `$raw` 原样返回，**保持原样**即可。
- 与上面的 `message_buttons` 是**两套机制**：`message_buttons` 是卡片底部跳转链接
  （`button_type: "url"`），`interactive_widgets` 是卡片内可交互组件，二者互不影响。

> **已验证**：带交互小组件的消息真机发送成功（平台接受该结构），且**人工确认卡片上的
> 按钮能正常显示、点击能跳转**。**未验证**：其它组件类型（如输入项）—— 没有样例，不要猜结构。

### 人员字段的写入值

`Contact`（人员）字段**必须是带 id 的对象数组**：`[{"id": "用户ID"}]`。
只写 `"用户ID"` 或 `["用户ID"]` 都会被运行时拒为 `内核错误: E_INVALIDARG`——
工具会自动规整，但你自己拼 `$raw` 时要按这个形状写。
其余类型照常：数值 `5`、单选 `"待处理"`、多选 `["A","B"]`、日期 `"2026-09-15"`、完成率 `0.5`。
# 危险写动作必须限定范围（会被硬拦）

`db_delete_rows` / `db_update_rows` / `write_data` 这类动作，**范围参数必须真的限定住记录**：

- 写成"全部记录"（如 `search_range: { filter_type: "all", ... }`）或空条件树
  （`{ mode: "AND", criteria: [] }`）都会被判为**未限定范围**并拒绝 `dry_run`、草稿保存或启用；
- 动作模板缺失或未能识别出范围参数时也会硬拦截；工具无法确认范围安全，不会把它当 warning 后继续；
- 原因：底层对空条件的语义是**匹配全部记录**（`branch_condition` 里 `len(criteria)==0 → true`），
  一条"删除全部记录"的流程跑起来是不可逆的数据事故；
- 确需全表操作时，请让用户在界面里手动配置，不要用本工具绕过。
# 修改流程时的 `enabled` 语义

- `auto_sheet.save_flow` 的 `action=update` **不传 `enabled` 时沿用流程当前状态**（不会顺手停掉正在运行的流程）；
- 要改启停请**显式**传 `enabled`，或更推荐用 `auto_sheet.set_trigger_status`；
- 若本次修改的对象正在运行，返回里会给出一条显著 warning，请如实转达用户。

# 渲染侧：节点图标与渠道条目从哪来

**结论：画布上的节点图标由 `linkers[].action.kd_code` 决定，不是渠道条目的 `kd_code`。**
真机实证（2026-10-08）：`db_search_rows` 的渠道图标是 `<DbsFormat/>`、动作图标是 `<DocMagnifier/>`，
编辑器画布把这个节点渲染成 `kd-icon-doc_magnifier`（= 动作图标）。渠道条目的另外三个字段
（`name`/`label`/`kd_code`）决定"这个节点属于哪个指令分类"以及右侧「配置参数」用哪套模板。

**这份数据由工具自己保证，不需要（也不应该）逐个人工目视校验：**

- `auto_sheet.save_flow` 组包时，渠道元数据取自**编辑器自己的目录接口**
  （`/chatflow/api/v2/instruction/search_with_recommend`，「添加步骤」面板的数据源），
  静态登记表只在接口拿不到时兜底 —— 平台改图标/改渠道名，链路自动跟上；
- `kd_code` 必须落在**平台图标字典**内（73 个取值，抓自平台前端产物）。字典外的值**静默降级**成兜底图标、不报错 ——
  正因如此，"图标对不对"必须由**离线用例**守住，不能靠抽查；
- 覆盖度与口径由离线用例守住：白名单动作一条不缺、图标非空、`kd_code` 都落在平台字典内，
  以及目录解析与「逐字段取非空者」的合并口径。

**触发器图标不用我们写**：平台按 `trigger.type` / `event.type` 自己算
（chatflow 的 `TriggerKdCode` / `EventTypeKdCode`）。真机 10 种触发器（新增/修改/到达指定时间/新增或修改/
点击按钮/定时/webhook/手动/轻审批新增/轻审批状态变更）图标全部正常。

## 动作取用口径：以编辑器目录（online=1）为准

**判断"某个动作现在能不能用"，唯一判据是编辑器「添加步骤」面板的数据源**
`/chatflow/api/v2/instruction/search_with_recommend` —— 它**只返回 `online=1` 的指令**
（chatflow 源码 `ListByNameENs(..., true)`，"常见原因: instruction_info.online=0 仅上线指令参与展示"）。
⚠️ 不要拿 `/instruction/templates` 当判据：它**不过滤 online**，已下架动作照样返回模板。

chatflow 的动作是"一代一份 `instruction_info`"，每上一代都会被同步脚本 `SET online = 0`，
编辑器随即不再提供。**"能存、能渲染、能跑"不等于"该用"**：老动作在 runtime 仍有注册，
手工构造 DSL 能保存成功（真机 12 个节点图标与参数面板全部正常），
但用户在界面上根本选不到、改不了。

2026-10-08 真机核对并完成的迁移：

| 旧动作 | 编辑器面板里 | 现在用什么 |
|--------|--------------|------------|
| `notify_v3`（发送公众号和文档消息，2025-01 `2501a` 起 online=0） | **没有**（只有 `notify_v4`） | `notify_v4`：参数是 v3 的**超集**（多 `button_type`/`link`，`receiver`/`title`/`message_content`/`record_link` 同名同义）。⚠️ **`link` 是平台无条件必填**（真机 400001；即便 `button_type=fileUrl` 运行期用不到）—— 工具会自动代填文档链接并 warning，要自定义卡片跳转地址就显式传 `link` |
| `send_woa_robot_message_v6`（WPS 协作群机器人，2507/2509 起 online=0，无后继） | **没有** | `send_woa_message_v9` 的 `group_id_list`（群发已并入 v9） |
| `notify_v3`（ET 侧） | **没有**（ET 既无 v3 也无 v4） | `send_woa_message_v9` 的 `person_id_list`（通知个人） |

本工具侧已按此收口：db 白名单只用 `notify_v4`、不再提供群机器人动作，ET 白名单去掉 `notify_v3`；
并有离线用例守住「白名单 ⊆ 编辑器目录快照」，防止再漂移。
