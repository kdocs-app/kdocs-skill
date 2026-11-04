# 平台事实与踩坑

> 以下都是**真机验证过**的平台行为与坑，按 slug 索引。结论即用法：照做能跑通，反着做会静默失效。
> 新增事实直接在本节追加（本节为手写，不再由构建生成）；需要平台侧处理的写在条目里，不必单列汇总。

## array-each-panic

数组返回值 **不能用 `|each`**：平台直接 panic（reflect: Call using []map[string]interface {} as type []*flowjson.DataFrame），节点失败且不给可读原因。

## array-index-ok

数组可用**下标**引用：`${s1.resp_schema.items.0.name}` → 值；整体引用 `${s1.resp_schema.items}` 会把每个元素的 JSON 用**空格拼接**，且元素内 key 顺序不保证（不要拿它解析）。

## array-items-as-expressions-are-opaque-to-decompiler

**（已修，2026-09-21）「数组项是表达式对象」的形态，反编译器不建模 ⇒ 节点回读后退成 `$raw`，后续 update 被安全守卫拒绝**。第 14/15 轮把 `insert_values`/`flat_array` 的项改成表达式对象（与平台产物一致）后，这类节点在 `get_flow` 里成了不透明节点；再调 `save_flow(update)` 会报 `不得删除含复杂配置（$raw）的节点；请保留该节点，或先在界面中删除后再由 AI 接管`。真机四例：[M1] wecom-md-92127（at_mobile_list 表达式项）、[M1] wecom-md-92127（image_content 表达式项）、[M15] wecom-dash-tpl/[M15] wecom-dash-var（image_content 仪表盘项）—— 全部无法再被 AI 更新。⇒ **修法（已实施）**：让反编译器与编译器对称 —— `flat_array` 在**顶层**（非嵌套）时逐项解码回 DSL（`{"$var": …}` / `{"$template": …}` / 字面量），并加三道「可逐字节重建」门槛（外层 config 只允许 biz_type 或空、项只允许标量型表达式、项 config 必须可重建）；**嵌套**的 flat_array 项（woa `at_person_list` 的结构性空 flat_array）仍保持 `$raw` —— 金标准 `update_notify` 抓到了过度解码，故加了 nested 上下文标记。修复后：`opaqueKeys` 对 [M1] wecom-md-92127/[M1] wecom-md-92127/[M1] wecom-md-92127 全为空，且 [M1] wecom-md-92127 的 update 由被拒变为成功。

## biztype-follows-field-type

脚本入参绑变量时，`biz_type` 取**变量所指字段的真实类型**（MultiLineText→string、MultipleSelect→MultipleSelect），不是模板 default_type（恒为 string）。写错会让编辑器判空。

## branch-shape-diffs

**条件分支（branch/sub_branch）的 canonical 形状必须逐字段等于界面产物**，否则服务端全绿而编辑器画布/条件面板是空的（2026-09-24 真机缺陷，见 platform-facts 的 branch-shape-diffs）。① `sub_branch` 节点 id 前缀 `branch_`（画布按前缀认节点类型；写成 `flow_` 时整棵子分支树挂不上）；② `sub_branch` 的条件入参是 `data_type:2` + label「设置该分支条件」，`extra` 带 `isNeedParentStep:true` 且与容器同 step；③ 条件外层 `__exp_config__.biz_type` 是 `string`（写 `branch_condition` 时编辑器按未知类型分派，运算符/值整片退回占位符）；④ SingleSelect 相等 → `operator: single_select_in` + 值数组 `[{"type":"Text","value":"已入库"}]`；数值 → `number_gt` + `template_string{value_type:input}`；⑤ 变量左值 `__exp_config__` = `{biz_type:<字段真实类型>, _real_biz_type:<比较域>, from_dataframe:true, sheets:{}}` —— **`from_dataframe:true` 不只是渲染用**：chatflow 的 `extractDataframeReference` 只认带这个键的左值，缺它时条件根本不会编译成记录字段过滤，运行期也取不到值；⑥ linkers：分支容器条目 `ui_type=condition`，`sub_branch` 是**节点级条目**（顶层 `name=sub_branch`、`ui_type=normal`、`action.isNeedParentStep=true`），不走渠道条目形态。以上六项已全部修完：真机流程「示例流程V5」的 canonical 与界面金标准逐字段一致，闸②（平台配置检查）`passed`、编辑器画布与条件行完整渲染、闸③ 真机端到端跑通（两个 sub_branch 各命中一次、各真实发出企微消息，未命中分支的子节点确实不执行）。canonical 快照在验证时本地留存（含真实流程 id/名称，不入库）。

## branch-step-record-count-is-numeric

分支比较前序「查找内容」的 `__record_count` 时，左值必须按 number 编码；通用 `biz_type=variable` 会让编辑器显示「选择条件」空白。编译形状使用 `biz_type=number`、`_real_biz_type=number`，右值为数值输入；不要设置 `from_dataframe`，否则 chatflow 会把标量比较改写为 DataFrame.Filter。

## button-column-must-run-automation

**「点击按钮时」的自动化，必须把按钮列自身的动作配置成「触发自动化流程」；只写流程 canonical 不够**。平台把按钮动作存在按钮字段的 customConfig 里：actionType=0（触发自动化流程，actionConfig 为空）点击才会触发流程；actionType=2 是 URL 动作。真机对照（流程 [T8] button-x [T8] button-x，期间一直 enabled）：按钮列原为 actionType=2 且动作是打开 https://baidu.com → 点击后**运行记录 0 条**（事件根本没到流程）；把按钮动作改为「触发自动化流程」后 actionType=0 → 再点一次：运行 exec_status=1、节点「按钮触发」result=「某成员 在[门店销售]点击了[按钮]按钮成功触发运行」、消息节点「成功发送消息」。⇒ 我方 save_flow 只写 button.sheet_id + btn_field，**不建立这层绑定**；要程序化配置需写按钮列的 customConfig（可用 http.db.updateFields），或由界面配置。

## button-new-column-dsl-and-consent

**「新建按钮列」有专门的 DSL 写法，且写表结构必须由用户明确要求**。触发器里写 button.new_button（含 name 与可选 text）时，save_flow 会先建列（dbsheet 的 createFields，字段 customConfig 固定为 actionType 0 = 触发自动化流程）、用返回的字段 id 作为 button.btn_field 再建流程，并在结果里回显「已新建按钮列…」；若只给 button.btn_field（已有按钮），则**只建流程、一个字段都不写**。规则边界：**用户明确要求「新增一列按钮来触发某个自动化」时才用 new_button（按约定无需二次确认）；用户没提、由模型自行发起的建列，必须先告知表名/列名/类型/动作并取得用户同意**；已有按钮列的行为一律不改（本期实例：按钮列原本是 actionType 2 的 URL 动作，点它不会触发流程）。

## canonical-expr-passthrough

反编译读出来的值若是 **canonical 表达式对象**，重新编译必须**原样带回**；否则会被当成字面量再包一层 `original` —— 「读出→存回」就把流程改坏了。

## canonical-node-snapshots

canonical 里有一批**节点快照、推导不出来**：参数 `label`、`biz_type`（同一参数在不同节点值不同）、`sheets`、`value_label`、`_temp_id`、`data_type`、成员项的 DSL 糖。

## cron-time-key-depends-on-mode

**定时触发器的「触发时间」写哪个键随定时方式走，且两个键绝不同时非空**：`every_day` 写数组 `time_formats`（面板是「HH:mm 列表」，最多 20 个，平台描述「每天 18:00」也读数组）；其它方式（single/每周/每月/每工作日/国家工作日）写单值 `time_format` —— 平台的流程描述对这些类型**只读 `time_format`**，只写数组会得到「每周五 」（真机实测：every_week_day + `time_formats:["16:00"]` → `func_desc="「每周五 」，延迟"`，时间点丢失）。两条硬约束的由来：(1) 面板的「触发时间」是**同一个 chip 输入**，它把 `time_formats` 与**非空**的 `time_format` 并成一个 chips 列表 ⇒ 两个都给值时界面出现两个一模一样的时间（真机每天 18:00 显示成 `18:00 ×` `18:00 ×`）；(2) 空串是惰性的（`single_time` 一直写 ""、界面不渲染），所以未用到的键保持 ""。平台侧对两者都接受（`CheckTriggerCronReq` 只要其一非空；`ActionMarshalCronSpec` 优先数组、数组为空时回退单值；启用/停用走同一函数），v1 老路径（`FuncServiceImpl.Save` → `marshalCronSpec`）只读单值但它不被编辑器调用（编辑器与 MCP 都走 `/chatflow/api/v2/func/save`）。非 every_day 给多个时间点时按数组落库（排期每个时间点各触发一次）并回一条 warning：平台描述只带一个时间点。

## db-event-write-step-needs-u2005-terminator

⚠️ **（更正上一条误判）「db_event + 只有一个写记录动作」并不是开启被拒的原因 —— 真因是写记录节点里 `$trigger_dataframe` 的结束符写成了 ASCII 空格**。写记录节点的 `search_range.ever_result.__exp_value__` 必须是 `"$trigger_dataframe" + U+2005`（表达式终止符）；写成 `"$trigger_dataframe "`（ASCII 空格）时，`save_flow` 照常成功、`get_flow` 回读**逐字段正确**，但 `set_trigger_status(enabled=true)` 回 `400001 [workflow] 平台拒绝开启该流程（服务端配置校验未通过）：服务错误`（文案毫无指向性）。**单变量对照（2026-09-23）**：同一流程除这一个字符外逐字节相同 ⇒ ASCII 空格 ❌被拒 / U+2005 ✅可开启。**归因过程**：先用「照抄既有可开启流程 + 单点改动」二分 —— 只换写记录节点就从 ✅ 变 ❌，只换筛选/只删 queryRanges/只换通知节点都不影响；再对平台侧原始存储串做逐字段 diff，唯一实质差异就是这个结束符（既有流程是 `e2 80 85`）。旁证：只放通知动作的流程本就可开启，所以「动作数量/类型」不是变量。⇒ **排查口径**：开启期 400001 先查步骤里所有 `$…` 表达式串的**结尾字符**（是否 U+2005），再怀疑其它。

## db-trigger-filter-branch-combination-boundary

**记录事件触发器与分支的组合边界（2026-10-19）**：同一条 `db_search_rows → branch(__record_count) → delay` 链路，在手动触发下 `check_only` 与真跑都通过；type=4「修改记录」不带触发器筛选时也有真机运行成功记录。type=3「新增记录」、type=5「到达指定时间」、type=7「新增或修改记录」接同一分支链路时，本地 `validate_flow` 均通过，但平台 `check_only` 都返回 400001（`error_step=workflow`，提示“服务错误”），流程保持未启用。对照：type=3/5/7 各自只接只读查找时平台 `check_only` 通过；type=7 带 `filter_result` 的只读查找也通过。⇒ 当前证据指向平台对 **type=3/5/7 + branch 链路** 的组合校验不兼容，筛选不是必要条件；不要把这些触发器本身或 type=7 筛选判为不可用，也不要把本地 canonical 校验通过误报为该组合可运行。

## delete-dbt-clears-table

⚠️ `delete_dbt('表名')` **清空整张表**（没有条件参数），而且**不报错**（exec_status=1）。（人工探针，2026-09-17，一次性临时表；不可自动复现 —— 它太危险，不适合放进自动化用例）

## empty-queryranges-only-hurts-type4-filter

**空 `queryRanges` 不是普遍有害，只对「修改记录时(type=4) + 筛选条件」致命**：该组合下 `queryRanges:[]` 会让事件触发但运行**失败**（节点「事件触发 - 触发失败」`exec_status=2`、「触发数据为空」）；而补上非空 `queryRanges` 后**完全不触发**（[C,n]、[C]、[C,D,E,m,n] 三种内容都试过）。对照组：type=3（新增记录时）、type=5（到达指定时间）空 `queryRanges` **均正常工作**；type=7（新增或修改）+ 筛选 + 非空 `queryRanges` 正常工作（真机实发消息）。⇒ **type=4 的 canonical 形状是从 type=3/7 外推的，未与界面产物对照过，需要界面样本才能定案**（平台现网没有 type=4 的界面流程可参照）。

## et-structured-params-need-filerangeinfo

**et 侧的结构型参数必须落成平台的结构体对象，落成标量必 10001（真缺陷，已修）**：① `info`（声明类型 `et_range`/`et_file`）运行期是 `*et.FileRangeInfo`（`{file:{file_id}, sheet_id, sheet_name, range, hasHeader}`）；我们此前把它**压成标量** —— `write_data`/`filter_rows` 落成工作表 id 数字（且 `range` 被静默丢弃）、`create_sheet` 落成文件 id 字符串 —— 平台反序列化直接失败 ⇒ 运行前 `10001 流程存在配置错误`（真机 [P] et-filter_rows/[P] et-write_data/[P] et-create_sheet/[M19] et-mk-65647/[M20] et-wf-65650/[M20] et-wf-65831/[M20] et-wf-65873 全部命中）。② `write_data.data`（声明类型 `dataframe`）运行期是 `*flowjson.DataFrame`（`{cells, columns}`）；写裸二维数组同样 10001。③ 这也是 L1 里那 7 个 et 参数「从来没有任何用例」的**根因** —— 形状不对，写了也跑不起来。修复后真机：`create_sheet`「已创建工作表 […]」且 get_context 里出现新表（[M19] et-mk-65826）；`write_data`「已将 [2] 行 [2] 列数据写入到 [Sheet1]」+ `filter_rows`「获取 [1] 行 [1] 列数据」（[M20] et-wf-65931）。

## et-structured-params-render-undefined-in-editor

**（已修，2026-09-21）et 结构型参数的编辑器显示**：曾经运行与配置检查都正常，但编辑器把「工作簿」渲染成 `[undefined]undefined!undefined`；根因与修法见下，现已显示正常（M19/M20 界面复核）。真机+UI 实测（2026-09-21）：`create_sheet` 流程 [M19] et-mk-65826（动作节点「创建工作表」）配置参数面板里 **工作表名称 = 自动化验收-65826 ✅ 正确回显**，但 **工作簿 = `[undefined]undefined!undefined`**；该流程真机运行成功（节点「已创建工作表[…]」）、`get_context` 与 et 工作表列表里都能看到新建的表（列表里已出现 自动化验收-65826 / (2) / (3)，M19 跑了三次各建一张）⇒ **这是纯显示缺陷，不影响运行**。**已修一半（2026-09-21）**：① `file.file_id` 必须写**数字型 chatflow 文件 id**（v3，如 `460000000001`），不能写 v7 字符串 id —— 平台读流程时做 `fileInfo["file_id"] == origFileId`（`origFileId = strconv.Itoa(fileId)`，比对用的是裸 `==`），匹配上才会补 `file_name` 并生成显示串 `[文件名]工作表名!区域`；写成 v7 id 时编辑器显示 `[undefined]undefined!undefined`。改后真机复验（[M19] et-mk-67081）：编辑器「工作簿」从 `[undefined]undefined!undefined` 变成 **`[undefined]Sheet1!undefined`** —— **`sheet_name` 段已正常**（编辑器直接读值里的 `sheet_name`）。② **仍缺 `file_name`**：编辑器按 `[${file_name}]${sheet_name}!${range}` 拼串，`file_name` 段要由**值本身**带（平台那条注入只在特定路径跑，编辑器取流程时不注入）；而我们的编译器目前拿不到文件名（`valueResolver` 只拉工作表列表）。**已修完（2026-09-21 真机 + 界面复验）**：③ 保存 et 流程时从 `get_office_file_info`/文件信息取文件名（`automationTarget.FileName` → `buildContext.fileName`），写进 `info.file.file_name`；④ **按用户口径区分两种声明类型**：`et_file`（`create_sheet.info`）**只带文件**（编辑器是"选文件"控件，带 sheet 会显示成 `[undefined]Sheet1!undefined`）、`et_range`（`write_data`/`filter_rows`）才带 工作表+区域；biz_type 也分别写 `et_file`/`et_range`。复验（[M19] et-mk-67469）：编辑器「工作簿」现在显示为 **`表格AI功能测试`（选文件控件）**、「工作表名称」= `自动化验收-67469`，配置检查无错误；真机仍成功、副作用断言通过。

## et-write-data-needs-upstream-reference

**`write_data.data` 是「选上一步输出」的变量控件：写字面量表能存能跑，但编辑器参数面板会空白**。`data` 声明是 `kind=variable` / `type=dataframe`（平台侧 `compileEtVariableValue` 把步骤输出编成`{"__exp_kind__":"variable","__exp_config__":{"biz_type":"dataframe"},"__exp_value__":"$<步骤id> "}`（尾部 U+2005 终止符）—— 步骤结果在 kflow 里以**步骤 id 命名的全局变量**存在）。真机实证：字面量表写法（[M20] et-wf-65931）节点成功、`info`/`range` 都对，但**参数面板整片空白**（用户截图）；改成两步「筛选行 → 写入数据（`data={"$var":"s1"}`）」（[M20] et-wf-68226）后真机成功且**数据真的从 s1 流入 s2**（「获取 [1] 行 [1] 列数据」→「已将 [1] 行 [1] 列数据写入到 [Sheet1]」）。⇒ 口径：**dataframe 类参数一律用变量引用上游输出**；编译器已对字面量表加保存期警告（能存能跑但面板渲染不出来）。**界面复验（2026-09-21，[M20] et-wf-68226）**：写入数据节点面板**完整渲染** —— 「写入表格区域」= `[表格AI功能测试]Sheet1!AB1:AC2`（`file_name` 修复在 `et_range` 上同样生效）、「写入的内容」= `第2步|序号`（上一步输出的变量芯片）、「写入方式」= ⦿追加写入、「附带写入表头」= ☐。

## ever-result-trigger-binding-fails

❌ **被证伪的假设**：曾以为 `db_search_rows` 的 `search_range.ever_result = {"$var":"trigger_dataframe"}` 能把查找范围绑到「本次触发的那条记录」，从而实现零分支的按条件通知。**真机结果：流程运行失败**（保存/dry_run/开启都能过，运行期不成立）——`ever_result` 的语义是「沿用**上游查找**的结果」，不是「绑触发记录」。

## external-link-recalc-does-not-trigger

⚠️ **「别的表反向关联」引起的计算列重算，不算这条记录的一次「修改」⇒ 不触发 type=7 流程**（2026-09-23 真机，商品库存表）。商品库存的 `入库数量(b)=SUM([@[采购数量 (采购入库)]])`、`出库数量(d)=SUM([@销售数量(门店销售)])` 都是 Formula，来源是**别的表的记录**；在「采购入库」新增一条 `采购数量=1000` 且 `W 采购商品` 指向某商品库存行的记录后，该行 `入库数量` 0→1000、`剩余库存` 同步重算，但**这一行没有任何单元格被用户编辑** ⇒ 该表上 type=7 流程 **runs 不变**。三条独立证据排除了其它解释：① 探针流程用**恒真筛选**（零售价≥0）→ 仍不触发（不是筛选没满足）；② 同一行当时**已满足筛选**（入库数量早就 1000）时再挂「门店销售」记录使 `出库数量` 0→1000 → 仍不触发（排除「按改动前取值」的解释）；③ 紧接着改该行一个**监听字段**（零售价 / 商品型号）→ 立刻 runs+1（证明流程与筛选都是活的，且执行器真发了企微）。⇒ 口径：**公式列的重算只有在同一行有用户编辑时才会跟随触发**（见 formula-result-change-counts-as-change 的边界）；业务含义：要「入库/出库发生的那一刻就提醒」，触发器应建在**源表**（采购入库 / 门店销售）上；挂在汇总表上只能等到「下次有人编辑这一行」才提醒。真机流程：`商品库存大额变动提醒`（type=7，监听字段=商品名称/商品型号/零售价，筛选 opType=`Or`：入库数量≥1000 或 出库数量≥1000，动作=发企微），已启用并通过真机验收。

## feishu-robot-is-et-only

**飞书群机器人是 ET 侧能力，db（多维表格）已不支持** —— 产品口径（2026-09-21 用户确认）：「多维表格目前不再支持飞书群聊消息发送，**只有 ET 支持**」。⚠️ 平台**并未硬拦**：在 db 文件里真机跑飞书群机器人仍报「成功发送消息」（2026-09-21 真机实测）—— 属「能跑但产品不支持」，与 `db_update_ai_field` 同类（模板 online、产品不支持）⇒ **以产品口径为准**：db 白名单移除该动作、et 白名单加入；db 侧用例改为显式跳过并注明需 et 文件。这同时解释了此前的界面取证结论（db 文件的动作面板里没有「飞书」分组）——不是灰度或环境问题，而是产品口径。

## field-ids-insert-nonempty-semantics

**`field_ids` 在「新增路径」上是「新增时以下字段不为空」条件**（不是单纯的监听列表）。⚠️ 界面上这个控件的**标题原文就是**「*新增时以下字段不为空，或修改以下字段时触发*」——平台自己把「新增=全部非空」「修改=监听白名单」这两条语义写在标签里了（用户 2026-09-23 界面截图，商品库存表）。真机实证：type=7 + `field_ids=["C"]` + 新增一条 C=142 的记录 → **触发**；同样流程新增一条只填备注、C 为空的记录 → **不触发**。⇒ 因此**「没指定监听字段就默认补全全部可触发字段」是错的**：本表 14 个可触发字段全部非空几乎不可能，新增路径会**永不触发**（真机 B1：自动补全 14 字段 + 只填 C 的新记录 → 不触发；这也解释了 T4a 的失败）。**已撤销该默认补全**：type=4/7 现在都要求显式给出 `field_ids`，缺失即报错并在文案里说明这条语义。 2026-09-23 在「采购入库」用**干净对照**（暂停兄弟流程、逐项单变量）再证一次：同一触发器 + 同一筛选「采购总金额≥30000 且 备注说明 不包含 大单」，监听字段=6（全选）时新增一条只填「采购数量/采购单价/采购日期」的记录 → **不触发**；监听字段=3 时填满这 3 个 → **触发**（同一次新增同时建的两条流程，一条跑一条不跑）。⇒「全部监听字段非空」成立，且**界面默认全选 ⇒ 默认形态下新增路径事实上失效**（详见 trigger-watch-fields-default-select-all）。

## flow-count-cap-per-file

**单个文件最多 100 条自动化流程**：`save_flow(create)` 超限时平台返回 `code=400001「一个文件中最多可创建100个流程」`（真机：动作矩阵回归 sweep 打到上限，两条用例在创建阶段即失败、流程未产生）。⇒ 大批量真机验证前需先清理历史流程；工具侧把该文案原样透传（已足够可读），未做前置计数提示。

## flow-name-max-30-chars

**流程名最长 30 个字符**（超出时 create/save 直接失败：`[workflow] 流程名最长 30 个字符（当前 N 个字符）`）。批量建流程时要注意命名长度。

## formula-result-change-counts-as-change

**公式列的结果变化本身就算一次「修改」**:后端对修改类订阅会把「公式结果变化」一并计入修改。⇒ 「改关联列『商品』→ 查找列『零售价』→ 公式列『销售总额』变化」这条链路**能触发**流程,不需要(也无法)把公式列写进监听字段。(我先前判过「平台层面监听不了」,是错的,已更正。)⚠️ **边界（2026-09-23 真机补充）**：这条只在「**同一行上有用户编辑**」时成立。若公式列的来源是**别的表反向关联**（如商品库存的 入库数量=SUM(采购数量[关联采购入库])，由采购入库新增记录汇总而来），该行没有任何单元格被编辑 ⇒ **不算修改、不触发**（见 external-link-recalc-does-not-trigger）。

## http-return-path

HTTP 步骤的返回引用路径是 **`${sN.body}` / `${sN.body.<字段>}`**（按 `response_schema` 声明的结构逐层取），**不带 `response_schema` 这一层**；⚠️ `${sN.status_code}` **恒为 0**（平台 `stdHttpResult` 初始化后只回填 body，从不写状态码）。

## input-face-must-cover-template-params

**节点 `input` 必须与模板声明的可见参数面逐条、同序一一对应；少一条（或中间错位）会让编辑器「配置参数」整个面板渲染成空白**（服务端保存 / dry-run / 平台配置检查全绿，只有人眼看界面才发现）。2026-09-24 真机：分支流程「示例流程V4」的企微节点入参只有 6 条（模型没写 `at_mobile_list`，而编译器当时的规则是「本次没配的参数不造空值」），点开节点右侧只有标题、没有任何控件；对照平面流程「示例流程·大单提醒」的**同一个动作**有 7 条 → 面板完整。根因：模板给 `at_mobile_list` 的默认值是**空对象 `{}`**（`default_kind=insert_values`、`default_type=string_array`，界面把它原样写进入参），而 `editorDefault` 只认带 `__exp_kind__` 的表达式与标量值，空对象被当成「没有默认值」跳过 ⇒ 入参面短一条。修法：容器类 kind（`insert_values`/`flat_array`）的空对象默认值要采纳，并落成界面同形的 `insert_values{__exp_value__:{}}`（biz_type 取模板 `default_type`，不是 `inferBizType` 的 `json_object`）。修后「示例流程V5」的企微节点入参面 7/7 与模板声明面同序、`at_mobile_list` 与界面产物逐字段一致。⚠️ 反向也要守住：同名参数在模板里声明多次（企微 `message_content` 隐藏 + 可见）时**只能发一条**（可见的那条），多发一条会把后面的控件整体挤位。

## insert-values-items-must-be-expressions

**`insert_values` 内层 `flat_array` 的每一项都必须是表达式对象，不能是裸标量**。平台产物实证： `send_email.to`（同为 insert_values/string_array）内层项 = `{__exp_config__:{biz_type:"string"}, __exp_kind__:"original", __exp_value__:"someone@example.com"}`。我方原先写裸标量：**运行期正常**（接收人/通知真机均成功），但**编辑器的列表控件读不出来** —— 企微 `at_mobile_list` 裸标量版在界面上「成员名单」显示空 + 报「手机号格式有误，请检查输入」+ 配置检查 1（真机 [M12] wecom-at-61476）；改成表达式项后（就地更新 [M1] wecom-md-92127）界面**回显「13800000000」、配置检查 0**，真机仍「成功发送消息」。项的 biz_type 取外层去掉 `_array`（string_array → string）。⚠️ 同轮还修掉一个**掩盖漂移的测试缺陷**：金标准回环用例原先拿「被反编译就地改写过的」夹具当期望值（insert_values 内层会被改成裸标量），于是我方这处漂移长期没被这条金标准发现；改为**先深拷贝再比对**后夹具恢复真实形状（测试随即因该漂移报红，修复后转绿）。

## invalid-receiver-silent-zero

⚠️ **接收人 id 无效时，消息动作报「发送 [0] 条消息」而不报错** —— 极易被误判成「上游引用 / each 没取到数据」。本轮排查就因此绕了一圈：查找明明查到 10 行，消息却是 0 条。

## linker-channel-meta-from-platform-catalog

**渲染侧口径（2026-10-08 全量核对）：画布节点图标 = linker 里 `action.kd_code`**（渠道条目的 `kd_code` 只决定分类与右侧参数模板）——真机实证：`db_search_rows` 的渠道图标 `<DbsFormat/>`、动作图标 `<DocMagnifier/>`，节点渲染成 `kd-icon-doc_magnifier`（= 动作图标）。而 `kd_code` 必须落在**平台图标字典**内（73 个取值，抓自平台前端产物）：字典外的值**静默降级**成兜底图标且不报错 ⇒ 「图标对不对」只能由离线用例守，不能靠肉眼抽查。渠道元数据（渠道名/中文名/渠道图标/动作图标/ui_type）的**权威来源是编辑器自己的目录接口** `/chatflow/api/v2/instruction/search_with_recommend`（「添加步骤」面板的数据源，只返回 online=1 的指令，且带 ui_type：分支容器是 condition）；保存时按「逐字段取非空者」合并进静态登记表 —— 平台改图标/改渠道名/加动作，链路自动跟上，不再需要人工补登记（delay、ET 四动作两次漏登记都是这个缺陷类）。触发器图标不由我们写：平台按 trigger.type/event.type 自己算（chatflow 的 TriggerKdCode/EventTypeKdCode），真机 10 种触发器（新增/修改/到达指定时间/新增或修改/点击按钮/定时/webhook/手动/轻审批新增/轻审批状态变更）图标全部正常。

## linkers-drive-editor-render

**linkers 形态直接决定编辑器能不能渲染**（我先前判成「无所谓」，是错的——当时只验了运行期 each，没验界面）：顶层 `name` 必须是**指令/渠道名**（如 `wecom_v2`/`db_table_v2`/`wps_cooperation_v2`）、`label` 是渠道中文名、`kd_code` 是指令图标；内嵌 `action` 里才是动作名 + 动作图标(`kd_code`) + `restriction` + `step` + `type`。写成「顶层 name = 动作名」的简化形态时：**界面把节点显示成别的渠道（企微节点显示成飞书图标）、右侧「配置参数」整片空白**，用户无法核对/修改。 2026-09-17 用户实测又逮到一例：`linkerChannelMetas` **只登记了 db_search_rows 与消息/脚本类**，新增/修改/删除记录三个动作漏登记 → 编辑器里「新增记录」节点显示**飞书图标**。 对照界面流程的 linker 原文补登记（渠道层 `db_table_v2/多维表/<DbsFormat/>`、`action.kd_code=<Insert/>`、`restriction=在指定数据表中新增记录`）后，我方 linker 与界面**逐字节一致**（仅节点 id 不同）。 注意：`icon` URL 两边都是模板给的 `instruction/feishu.svg`（**平台模板数据如此**，不是我们写错）——图标由渠道条目的 `kd_code` 决定。

## no-return-actions

消息 / 通知 / 延迟 / 邮件 / 机器人等动作的平台模板 `return` 为 **null** —— **没有返回值**；引用 `${sN.x}` 会**静默取空**，故在编译期拦下。查找类（pull 1000）与脚本类（pull 1005）才有返回值。

## notify-record-link-needs-record-reference

**notify_v4（原 notify_v3）的 `record_link` 必须是记录引用，不能填字面量 URL**：① 字面量在**运行前校验**就被拒（`/func/run` 回 `10001 流程存在配置错误`，[M10] notify-link-98478）—— 模板 `allow_types=[db_record]`、界面是「请选择跳转记录」记录选择器，字面量的 biz_type=string 不在集合内；② `{"$var":"trigger_dataframe.record_link"}` 在 manual 触发下通过校验但节点失败「选择步骤已删除」（[M1] wecom-md-92127，manual 没有记录上下文）；③ **正确形态 = 先查找记录，再引用 `{"$var":"s1.record_link"}`**（record_link 是记录结果的合成系统列，运行期只做 URL R 参数编码）—— 真机 [M10] notify-link-99303「发送 [1] 条消息」。已加编译期 warning 指出正确写法。

## notify-v4-link-is-unconditionally-required

**`notify_v4` 的 `link` 是平台无条件必填**（真机 2026-10-08 平台配置检查回 400001「[s1] 必填参数 link（跳转链接地址）为空」，`kind=missing_required`）——即便 `button_type` 是模板默认的 `fileUrl`（运行期 `getRecordLinkByButtonType` 用的是 session 的 FileLink、根本不读我们填的 link）也照拦；而模板给 `link` 的默认值是**空串** ⇒ 只填 receiver/title/message_content 的流程能存、能回读、却**永远开不起来**。真机矩阵（manual 触发 + check_only）：仅三个消息参数 → 400001；+button_type=fileUrl → 400001；+link=<任意非空> → passed；button_type=url+link → passed；想用 record_link 替代 → 400001。⇒ 工具为 notify_v4 代填非空 link（优先调用方给的文档 url，否则文档域名）并显式 warning；用户要自定义卡片跳转地址时显式传 link。

## ocr-return-is-wordlist

图像识别的返回列 = 你在 `ocr_word_list` 里配置的识别项（源码：ObjectArrayToDataFrame(objArr, schemas, OcrWordList)）；因为列集是**源码推断**而非真机核对，引用未匹配项只 warning 不报错。

## python-dbt-sheet-name

Python 取数/写入 **必须显式传 `sheet_name`**：`dbt` / `insert_dbt` / `update_dbt` 的默认空串在自动化里解析不出「当前激活表」→ `RuntimeError: sheet with name [] not exists`。

## python-return-result

Python 的返回机制是 **`return_result({...})`**（平台注入到脚本 globals 的函数）；Python 顶层 `return` 是语法错误。编辑器（Pyright）会把它标成 not defined —— **静态误报**，运行时存在，**不要自己 def**。

## raw-string-param-literal

脚本入参里的**裸 `$...` 字符串不会被解析**，脚本拿到的是字面量文本（参考实现 dbsheet_agent 的示例写法就是这样，取不到值）。变量必须写成表达式对象。

## resp-schema-not-enforced

`resp_schema` 是**给编辑器的结构声明**，不是运行期类型约束：声明 `"name": 0` 而实际返回字符串，引用照样取到值（不报错、不取空）。

## script-return-object

脚本**必须 return 一个对象**：顶层 `return [1,2,3]` 会被平台判为「AS脚本输出错误，输出结果不是json格式」并让节点失败；数组只能放在对象的字段里。

## script-writes-data

**脚本动作不是只读**：脚本体内可读写数据（Python 侧注入 insert_dbt/update_dbt/delete_dbt）、发外部请求。因此影响面预审对含脚本的流程只说「影响面未知」，绝不说「只读」。

## search-filter-time-value-must-be-literal

**「查找内容」的筛选里，时间类字段（Date/CreatedTime/LastModifiedTime）**只能**给字面量 —— 动态值一律不可用**（2026-09-24 真机三形状对照）：① 编译出的表达式形态（DSL 写 `{"$var":"var_time_recent_day.7"}` → canonical `__exp_kind__:variable`）→ 查找节点**运行期**报 `内核错误: E_INVALIDARG`（真机）；② 把变量串塞进 canonical 文本值（`{type:Text,value:"$var_time_recent_day.7␅"}`）→ **不报错、条件被静默忽略**（命中全表 21 条）—— 这个形态最危险：能存能启用、运行日志「成功」，只是过滤条件形同不存在（结果是发出错误通知）；③ 字面量日期 `{type:Text,value:"2026-09-17"}` → 正常命中 17 条。⇒「采购日期早于当前 N 天」这类需求**不能**用查找筛选表达：要么写死字面量日期、要么在表里加一列公式（把超期判定算成可筛选的列）、要么改用「执行脚本」节点自己筛自己发。⚠️ 同轮还证伪了平台自己的相对日期类型（`{type:DynamicSimple,dynamicType:Last7Days}` + `NotIntersected`）→ `E_FAIL`。⚠️ 真机验证的是 **Date** 字段；`CreatedTime`/`LastModifiedTime` 同为时间类、编译器一并告警，但未单独真机验证。（注：数值条件「裸值必失败、必须包 `{type:Text,value:…}`」已由 `search-range-filter-values-must-be-wrapped` 单独登记；本条讲的是**值形态正确也救不了动态取值**。）

## search-range-filter-values-must-be-wrapped

**数据动作「查找内容」的筛选值必须包成界面形态 `{type:Text,value:…}`**：裸值在真机上**文本条件侥幸能跑、数值条件必失败** —— 平台运行时的数值操作转换器只认 map（读 `map[value]`），裸值被解析成空 ⇒ 节点 `CORE_ERROR 内核错误: E_INVALIDARG`（2026-09-20 真机：`销售数量 > 0` 这种良性条件都跑不通）。修法：`compileCriterion` 把字面量统一包成 `{type:Text|Contact|Link, value:「显示文本」}`（数值转字符串、布尔转 1/0），与界面/参考实现（dbsheet_agent）同形；已经是 canonical 值对象的原样透传。⚠️ Contact/Link 两种 type 只有参考实现证据、**无真机验证**。

## search-range-modes-are-exclusive

**`search_range` 是单选**：`filter.criteria` 与（非空）`ever_result` **只能给一个** —— 同时给值时保存/`dry_run`/回读**全部成功**、但流程跑到该节点**必失败**「流程配置存在错误」，且报文案不指向真因。2026-10-08 真机单变量（真机，门店销售表）：同一节点 `filter_type=filter` + `ever_result={"$var":"trigger_dataframe"}` ⇒ `exec_status=2`、result「流程配置存在错误」；**摘掉 `ever_result`、其余逐字节相同** ⇒ `exec_status=1`「查找到 [1] 条记录」。旁证：① 编辑器里「设置查找范围」就是**同一格 cascader**（要么按条件筛选、要么选某个步骤的结果），平台自己的默认形态是 `{"filter_type":"all","ever_result":{},"filter":{}}`（两个键都在，但都**空**）；② 源码 `db_search_rows_action.go` 按 `filter_type` 三选一，`FilterTypeEverResult` 分支直接 `df = SearchRange.EverResult`、**不再过滤**。⇒ 「绑本次触发记录 + 再筛条件」平台**表达不出来**（这也是 `ever-result-trigger-binding-fails` 的机制解释）；要在事件触发里判条件，就用 `filter_type=filter` 的条件查（判公式列记得先「延迟」，见 `trigger-snapshot-formula-column-lags`）。

## t7-queryranges-not-needed-for-modify-trigger

**type=7 的触发只看 `event.field_ids`，`queryRanges` 与它不一致不影响触发**（⇒ 编辑监听字段时**不必**重算 queryRanges）。真机（采购入库「示例流程·大单提醒」，筛选=采购总金额≥30000 且 备注说明 不包含 大单）：把监听字段从 [采购数量,采购日期,采购单价] 改成再加「状态」，用 `auto_sheet.save_flow(action=update)` 落库后 `queryRanges` **仍是旧列表（不含状态列）**，重新启用后把一条已达标记录的「状态」改成「未入库」→ **照常触发**（runs 10→11，备注被写成「大单，需要注意」）。顺带证实编辑是**外科式**的：除 `field_ids` 外，`filter_result` / `queryRanges` / `sheet_id` / `sheet_name` / `time` / `time_formats` / `sheet_all_field` 逐字段不变；`dry_run` 会回一份 diff（kept_step_keys + added existing:true）。⇒ 不要为了「一致性」去补 queryRanges —— queryRanges 致命的地方只有 type=4+筛选（见 empty-queryranges-only-hurts-type4-filter）。

## trigger-at-time-day-is-offset

**「到达记录中的指定时间」(type=5) 的 `day` 是相对记录日期的偏移，且过去时刻不回溯补发**。实证：`day=0`+记录日期=今天 → 触发；`day=+1`+今天 → 不触发（反例）；`day=-1`+记录日期=**明天** → 今天的目标时刻触发（提前一天提醒的真实用法）；`day=-1`+记录日期=今天 → 目标时刻是昨天、**不触发**（不是缺陷）。canonical：`{"day":-1,"field_ids":["D"],"time":"13:03","time_formats":[],"type":5}`，且 `queryRanges` 为空**可正常工作**。

## trigger-dsl-subobjects-and-typenums

**触发器 DSL 与类型号（真机全量验证）**：类型号 `manual=1 / cron=2 / event=3 / db_event=4 / webhook=5 / button=6 / 轻审批=7,8`；**cron / webhook / button 的配置都是触发器下的子对象**：`{"type":"cron","cron":{...}}`、`{"type":"webhook","webhook":{...}}`、`{"type":"button","button":{...}}`。必填：`cron_type` 入参字符串→落库数字 1..7（single/every_day/every_week_day/every_month_day/every_hour/every_work_day/country_work_day），single 必须给 `single_date`（2006-01-02）且**时点写 `time_formats`**（实测 `single_time` 落库为空、被忽略），every_week_day 必填 `every_week_day`（如 "1,3,5"），every_month_day 必填 `every_month_day`（1-31 或 L），every_hour 用 `time_minute`；`webhook` 必须由平台生成（见 webhook-url 那条）；`button` 必须带 `sheet_id` + `btn_field`（按钮列 id）。

## trigger-filter-judges-post-change-value

**「某字段改成 X 就通知」的正解：把触发器筛选（`event.filter_result`）写成【目标态】X** —— 记录变更类触发的筛选与「本次触发记录」里的**非公式列**都是【改后】的新值，**消息正文 / 分支条件里的变量芯片同样取新值**。2026-09-24 单变量真机（采购入库 sheet_id=3，记录 CG，真机（流程名「状态变更通知」）：监听字段=状态，筛选 `状态 ∈ [未入库]`）：① 改「已入库→未入库」⇒ **触发**（runs 2→3），trigger 行 `状态=未入库`（新值）、正文「当前状态：未入库」；② 紧接着改「未入库→已入库」⇒ **不触发**（runs 3→3）；③ 新增一条 `状态=未入库` ⇒ 触发（新增路径快照=插入值）。⇒ ① **筛选写成改前值就是把方向写反**：需求那一跳（未入库→已入库）**永不命中**，反而在「改成未入库」时触发，且 type=7 的新增路径会误报一条「已变更为 X」（Agent 真机产物「状态变更通知」就是这么错的，正文还出现「已变更为「已入库」…当前状态：未入库」自相矛盾）；② 平台不给旧值 ⇒ **无法表达严格的「从 A 改到 B」**（分支条件同样吃新值，也做不到）；要严格只能加一列「上次状态」由流程自己回写。⚠️ 旧条目 `trigger-modify-filter-uses-pre-change-value` 里「筛选判【改前】值」的口径**已证伪**：它当时只能解释**公式列**的现象（`trigger-snapshot-formula-column-lags`），对非公式列恰好相反。

## trigger-filter-numeric-needs-valuetype

**触发器筛选（`event.filter_result`）里，数值/公式列的每一行必须带 `valueType: "Fvt_Number"`，否则编辑器会把这行整行静默丢弃** —— 落库是对的、编辑器里却显示成空的「+ 筛选条件」（2026-09-18 真机踩过：`销售总额 > 20000`）。原因：编辑器的筛选行按 `valueType` 决定能否渲染（与动作 `search_range` 同一个机制，那里早有兜底）。⇒ 工具已在触发器透传路径上兜底补全（`fillFilterValueTypes`），模型不必记这条；但手写 canonical 时要知道它。日期等其它类型是否也要各自的值类型**未取证**。

## trigger-filter-numeric-shape

**触发器筛选的数值型形状（真机验证：照此生成并成功开启）**：`values` 也是 `[{"type":"Text","value":"50"}]`（**字符串**，不是 `Number`）；`numberFormat` 必须用**字段真实格式串**（Number `0_ `、Currency `￥#,##0.00;￥-#,##0.00`）——`get_context` 现在会返回该字段；`filter_result` 顶层还有 `sheetId`；`op` 用平台枚举 `Greater`，组合用 `opType: And`。

## trigger-filter-optype-and-or

**触发器筛选的「且 / 或」由 `filter_result.opType` 决定：`"And"` = 界面「添加同时满足的条件」，「`Or`」= 界面筛选区标题行右侧下拉的「满足任一条件」—— 两者都被平台真正用于求值**。界面里新建筛选时只出现一个「+ 添加同时满足的条件」按钮（那是**空态**入口，等价于 And）；切「或」的控件是**筛选区标题行右侧**的下拉「满足所有条件 ▾」（与「收起/删除该筛选区」按钮同一行；用户 2026-09-23 的界面截图确认，见「商品库存」表触发器设置面板），下拉项字面量为「满足所有条件」/「满足任一条件」，筛选区标题文案是「新增或修改记录时同时满足以下条件」，下方还有一个「+ 筛选条件」按钮用于继续追加条件行。ein 侧 `opType` 是**原样透传**（无 DSL 糖），所以只能整段照抄界面形状。**真机判定（2026-09-23，单变量对照）**：同一流程、同一两条条件（采购总金额≥30000、采购数量≥50）、只改 `opType` —— 新增一条「数量=50 单价=1（总金额 50）」的记录：`And` 流程 runs=0（不触发），`Or` 流程 runs=1（触发）；再新增「数量=1 单价=1（总金额 1）」：`Or` 流程 runs 不增 ⇒ **不是恒真，确实按「或」求值**。两个 `opType` 都能开启（落库/回读/开启全绿）。

## trigger-filter-passthrough-works

**「只在特定状态变更时通知」的可行做法（真机启用成功）**：把界面形状的 `event.filter_result` + `event.queryRanges` **原样透传**给 `save_flow`（工具侧对 `filter_result` 只原样透传，不改写形状），步骤里直接引用 `${trigger_dataframe.<列>|each,plain}` —— 一次变更发一条，**零查找零分支**。实测：状态∈[已入库] 的 `filterFields` 与 `queryRanges` 正确落库，与界面流程逐字段一致。⚠️ 这属于「照抄界面规范形状」，工具**没有**可读的 DSL 糖。

## trigger-snapshot-formula-column-lags

**记录变更类触发（type=3/4/7）里，【公式列】在「本次触发记录」中是【改动前】的值** —— 修改路径**慢一拍**、新增路径取 **0**；引用它的**分支条件与消息正文一并吃旧值**（非公式列不受影响，见 `trigger-filter-judges-post-change-value`）。2026-09-24 真机（采购入库，流程「示例流程V5」，记录 采购数量=123 / 单价=456 / 采购总金额=123×456）：① 一次写入「采购数量 123→500」⇒ trigger 行里 `采购数量=500`（新）但 `采购总金额=56088`（**旧**），基于公式列的分支条件按旧值判 ⇒ 不命中；② 紧接着再写「状态」触发一次 ⇒ trigger 行里 `采购总金额=228000`（= 上一拍 500×456），这次才命中。筛选面同理：条件是公式列 `采购总金额≥30000` 时，① 记录 1×10=10 → 一次改成 100×500=50000（**由下穿上**）→ **不触发**（两个独立记录各复现：Bn、Bm）；② 记录**本来就在阈值上**再改（50000→60000、42000→49000、42000→56000）→ **全部触发**。而且**消息正文里打出的公式列数字是旧的**（另一次实测正文出现了「采购数量：123 / 采购总金额：136800」这种自相矛盾的组合，123×456 应为 56088）。⇒ ① **以公式列为条件时**「把不达标的记录改成达标」**不会**提醒，需求若写「改成满足条件就提醒」必须补边界说明；② 真机验收必须**两拍写入**（先把公式预热到目标值，再用非公式列触发第二次），否则会把「快照是旧值」误判成「分支没命中」；③ 要覆盖「改成达标就提醒」，只能把条件改用公式的**输入列**（条件里做不了乘法，公式列是表达「金额 > N」的唯一办法）。**2026-10-08 机制补充 + 正解**：不只是「快照是旧值」—— 公式列在**库里也是异步重算**的，写入后约 **1~2 秒**才更新；而变更触发是**写入瞬间**触发的、查找节点也是那一刻读库的，所以「改完立刻查公式列」**必然**读不到新值。真机单变量实测（测试文件「门店销售」表，`销售总额=[@零售价]*[@销售数量]`，记录 G）：把「销售数量 462→599」后， ① **+0s** 跑查找（条件 `销售总额 > 100000`、`filter_type=filter`）⇒ **查找到 [0] 条记录**；② **+2s / +3s / +5s / +10s** 各跑一次 ⇒ **全部查找到 [1] 条记录**（101231）；③ 同一跑内的对照：条件换成**非公式列** `销售数量 > 590` ⇒ **立刻**命中 [2] 条 —— 证明这不是「写入可见性」延迟，而是**公式重算**延迟。⇒ **正解：在查找之前插一个「延迟」节点**（`{key:s0, action:delay, params:{delay_time:10}}`，单位=秒、平台限 1~60），形状 = `事件触发 → 延迟(≥3s) → 查找(filter_type=filter，条件判公式列，on_not_found=exit) → 消息`。2026-10-08 该形状真机端到端验收通过（真机「示例流程·门店销售」：改一条记录 → 延迟 [10] 秒 → 查找到 [1] 条记录 →企微群收到「【大额销售】商品：穆勒鞋　销售总额：101231」）。⚠️ **别想用 `ever_result={"$var":"trigger_dataframe"}` 绕开这一步**：它与 `filter` 互斥，写了该节点运行必失败（见 `search-range-modes-are-exclusive`）。**2026-10-08 追加对照（回答「能不能用触发器筛选代替延迟」）**：同一张表、同一时刻、只差筛选字段的两个 type=7 流程，监听字段都是 `销售数量`、动作都只是一步「延迟 3 秒」（无副作用）：A 的触发器筛选 = `销售总额 > 100000`（**公式列**），B 的 = `销售数量 > 590`（**非公式列**）。① 把记录 G 的销售数量 462→600（总额 78078→**101400**，**由下穿上**阈值）⇒ **A runs 0→0（不触发）**、**B runs 0→1（立刻触发）**；② 紧接着 600→601（公式值已稳定在 101400，本就在阈值上）⇒ **A runs 0→1**、B 1→2。⇒ 触发器筛选判**公式列**时，「**由下穿上阈值**」那一次**必然漏掉**（而这正是需求要捕获的那一次），只有「本来就在阈值上」的记录再改才触发 ⇒ ⇒ 离线用例已把上述三条固化成可复跑的真机回归（A=触发器筛选判公式列 / B=判非公式列 / C=延迟+查找，断言：跨阈值那一次 A **不触发**、B 触发、C 触发且**查找到 [1]**；本就在阈值上再改时 A 才触发）。**不能用触发器筛选代替「延迟」**，判公式列只有「延迟 + 按条件查找」这一条路（唯一能算乘法的是脚本节点 `airscript_v2`/`run_python_script_v2`，但要自备脚本、且 `零售价` 在 DataFrame 里是带格式货币文本需自行解析、未真机验证 ⇒ 不作为推荐）；判**非公式列**时触发器筛选完全正常（这就是手册 ① 正解）。另：条件三处（触发器筛选 / 查找筛选 / 分支条件）都只有「字段 + 比较操作符 + 字面量」，**算不了乘法** —— 查找筛选最终下发 `dbsheet.ListRecordsByFilter`（左值只能是字段 id、右值只能是字面量，值类型只有 `Text`/`DynamicSimple`）；分支条件操作符全集是比较类（`Greater/GreaterOrEqual/Less/LessOrEqual/Equal/In/NotEqual/Empty/NotEmpty`）。

## trigger-watch-fields-default-select-all

**界面新建「新增或修改记录时」时，监听字段默认就是全选**（= 该表**全部可触发字段**，采购入库为 6 个：采购数量/采购日期/状态/采购单价/提交人/备注说明；可选集规则见 triggerable-fields-are-scalars-only），用户一般不会主动裁剪。**这一默认与「新增时以下字段不为空」叠在一起就是一对互斥取舍**：全选 ⇒ 新增路径要求这 6 个字段同时非空 ⇒ 事实上永不触发；要覆盖「新增就提醒」，只能把监听字段**裁到「业务上必然同时填写的那几列」**（如采购数量/采购日期/采购单价）。⇒ 手册与工具文案必须显式提示这对取舍；「默认全选」这个事实本身要写进文档，否则模型会以为是我们选歪了。界面旁证（2026-09-23 用户截图，商品库存表）：监听字段框里一次列了 6 个（商品名称/商品型号/零售价/商品图/款式/网站）——与采购入库默认 6 个（数量/日期/状态/单价/提交人/备注说明）同形：**不裁剪就等于把新增路径废掉**。

## trigger-watch-fields-gate-modify-path

**「监听字段」是修改路径的硬条件**：它决定「**哪些变化会触发**」，筛选条件只决定「触发后是否执行」——原判「两者正交、任何字段变化只要满足筛选就会触发」是**错的**（2026-09-23 真机更正，旧 ID `trigger-field-ids-vs-filter-are-orthogonal`）。真机（采购入库，type=7 + 筛选「采购总金额≥30000 且 备注说明 不包含 大单」）：① 监听字段=采购数量/采购日期/采购单价时，改「状态」**不触发**、只改「备注说明」（它在筛选里但不在监听里）也**不触发**；② 把「状态」纳入监听字段后，同一次改「状态」**立刻触发**（run 数 0→1）。⇒ 想让「入库（改状态）」这类动作也提醒，**必须把该列写进监听字段**，筛选条件替代不了它。平台源码里 type=7 建的修改类订阅 `DbUpdateFieldCells`（带 FieldIds）与「有筛选时另加的 `DbUpdateRecords`(只带 FilterResult)」**并不**等价于「任改必触发」。

## trigger-webhook-url-needs-chatflow-prefix

**webhook 回调地址 = `<kdocs_base>/chatflow` + 平台返回的相对 path**。`GET /chatflow/api/v2/func/webhook_url`（带 `file_id`）返回 `{"webhook":"/api/v2/func/webhook/<token>"}` —— 这是**相对路径**，直接 POST/GET 到它（不加 `/chatflow`）会 **404 page not found**；加上前缀后 POST 立即返回 `{"code":0,"data":{"task_id":"..."}}` 并成功触发。该接口**每次调用都 mint 新 token**（带不带 `function_id` 一样），所以要把它当作「可用 token」而非「已绑定的流程地址」；创建流程时需把该路径写进 `trigger.webhook.webhook`。

## triggerable-fields-are-scalars-only

**（仅限 type=7「新增或修改记录时」的「监听字段」）可选集 = 有独立单元格值(`fieldValueEditable=true`)且不是关联列(Link)的列**；注意 type=4 的「选择字段」范围更宽（含公式/关联/查找列，见 type4-sheet-all-field-ui-mapping）。公式(Formula)/查找引用(Lookup)/插件生成的对象列(Object)/关联列(Link)**不在选择器里**(AutoNumber、创建/修改时间等计算列同理)。实证(2026-09-18,用户给「门店销售」加了 8 个字段后点「全选」):下拉全集 = **10 个** —— 销售数量/销售日期/门店/负责人(Contact)/备注(MultiLineText)/购买款式(MultipleSelect)/服务照片·电子发票(Attachment)/开票日期(Date)/税前金额(Number);而 销售记录·销售总额(Formula,editable=false)、零售价(Lookup,false)、**电子发票识别(Object,editable=false)**、商品(Link,editable=true)全部未出现 ⇒ editable=false 与 Link 两类排除缺一不可。⇒ 工具「没指定监听字段就默认全部字段」必须按**可触发类型**过滤(`isTriggerableFieldType`),写成表结构全部列会得到界面无法表达的值(库里存得下,编辑器一保存就被规范化)。

## type4-sheet-all-field-ui-mapping

**type=4「修改记录时」的「设置生效范围」就是 `sheet_all_field`**，界面只有两个值（真机切换后保存、回读实证）：
- 「记录中**任何字段**被修改时生效」⇒ `sheet_all_field: 0` + `field_ids: []`；
- 「记录中**指定字段**被修改时生效」⇒ `sheet_all_field: 1` + `field_ids: ["C"]`（「选择字段」在该模式下是**必填**）。
⇒ 由此得出一条重要推论：**界面只在「指定字段」模式下才渲染「添加同时满足的条件」（筛选）入口** —— 界面**永远不会**产生 `sheet_all_field=0` + 筛选 的组合。另外 type=4 的「选择字段」选择器**范围比 type=7 的「监听字段」宽**（含公式/关联/查找列：销售记录/销售总额/商品/零售价），故 `triggerable-fields-are-scalars-only` 只适用于 type=7 的监听字段。

## type4-with-filter-is-a-platform-defect

**「修改记录时(type=4) + 筛选条件」平台侧不工作（平台缺陷，非我方形状问题）**：用界面自产的流程定案 —— 界面建「修改记录 → 发送企微群消息」，配「设置生效范围=指定字段 / 选择字段=销售数量 / 筛选 销售数量 > 100」，保存后回读 canonical（真机回读）：field_ids=[C] + sheet_all_field=1 + queryRanges=[{fieldIds:[C],sheetId:1}] + filter_result{op=Greater, values=[{type:Text,value:100}]}；启用后把一条记录的销售数量改成 601（满足 >100）→ **120 秒内无任何运行记录**。我方同形流程（C3/T3e）同样失败（不触发 / 触发但运行失败「触发数据为空」）。⇒ 该组合视为**平台不可用**；需要「修改记录时 + 条件」时应改用 **type=7 + 筛选**（已验证可用：真机实发消息）或**分支条件**。

## type4-with-filter-never-works-in-our-shapes

**「修改记录时(type=4) + 筛选条件」在我方能构造的所有 canonical 形态下都不工作**（真机四连测）：
- `sheet_all_field=0` + 空 queryRanges → 事件触发但**运行失败**（节点「事件触发 - 触发失败」`exec_status=2`、「触发数据为空」）；
- `sheet_all_field=0` + 非空 queryRanges（[C,n]/[C]/[C,D,E,m,n] 三种）→ **完全不触发**；
- `sheet_all_field=1` + 非空 queryRanges → 仍**运行失败**「触发数据为空」。
对照：type=3 / type=5 空 queryRanges 正常，type=7 + 筛选 + 非空 queryRanges 正常（真机实发消息）。
⇒ **已定案**：界面自产的带筛选 type=4 流程同样不工作 ⇒ **平台缺陷**，非我方形状问题（见 type4-with-filter-is-a-platform-defect）。

## unbounded-dangerous-write-rejected-in-dry-run

危险写动作的未限定范围，或动作模板缺失导致无法确认范围时，必须在所有 `save_flow` 路径上硬拦截：`dry_run` 不能对全表删除返回 `ok=true`，未启用草稿也不能先落库等待之后再校验。显式筛选且能通过模板识别范围后才可继续。

## wecom-at-all-works-despite-source-comment

**企微群机器人的 `at_type=at_all` 真机生效 —— 平台源码那句「企微不支持 @all」的注释已过时**。证据：流程 [M3] wecom-atall-92145/[M3] wecom-atall-63723（`at_type=at_all`）真机投递成功，且**用户人眼确认群里确实 @ 了所有人**（2026-09-21）。编辑器里该值是合法枚举（渲染成「提醒群内成员=提醒所有人」）。⇒ 需要 @所有人 时可放心用 `at_all`；不要再据源码注释劝退。对照：`at_person` + `at_mobile_list` 非空也真机成功（[M1] wecom-md-92127/[M17] dt-at-64637，@不到人不影响节点成功）。

## wecom-dashboard-image-shape

**企微 `image_content` 的「仪表盘图片」形态**：界面样本里项是 `{__exp_kind__:variable, __exp_value__:"$var_dashboard.<id>-{\"dashboard_type\":\"url\"} "}`，且**项与外层的 `__exp_config__` 都是空对象**（不写 `biz_type`）。我方原来只写 `$var_dashboard.<id>`（**缺方法后缀**）⇒ 运行前 `服务错误`（真机 [M15] wecom-dash-tpl/[M15] wecom-dash-var 两写法均失败）；补上方法后缀后表达式值已与界面**逐字节相同**，但仍 `服务错误`（[M15] wecom-dash-ok），而**界面同形流程真机跑通**（节点「成功发送消息」）⇒ 判定差异在 `biz_type`（我方 `et_range`/`object_array` vs 界面空对象），已改为空对象；**并发现另一处漂移**：模板把 `message_content` 声明两次（uiType=20 隐藏 + 21 可见），我第 4 轮「按声明面逐条下发」导致多写一条（7→8），会把 image_content 挤位；已改为**同名只发最后一条**（与界面样本的 7 条一致，离线用线上模板验证 8 声明→7 入参、顺序与界面相同）。**根因已定位并修复（2026-09-21）**：值的**结尾字符**必须是表达式终止符 **U+2005**，而我们写成了 **ASCII 空格** —— 逐字节实测：界面样本尾部 `e2 80 85`（U+2005）、我方 [M15] wecom-dash-ok/[M1] wecom-md-92127 尾部 `20`（空格）。平台编译动态字符串时 `var_dashboard` 的路径段会带上垃圾，`strconv.ParseInt(block.Path[0][0])` 直接 panic⇒ 运行前「服务错误」。这也解释了「条目看起来逐字节相同却一个能跑一个不能」。修后真机 [M1] wecom-md-63707：节点「成功发送消息」；**面板「图片内容」渲染为仪表盘「商品库存看板」芯片**（⑤ 通过）。已加离线回归测试（同时断言空 `__exp_config__`）。

## wecom-image-content-editor-shape

**企微群机器人发图：`flat_array` 的项必须是表达式对象，且界面原生来源只有「仪表盘」**。① 运行期图片与 `message_type` 无关（`formatImageContent` 无条件执行，逐项当 URL GET、≤2MB、转 base64+md5）⇒ 发图 = `message_type` 保持 text/markdown + 图片写 `image_content`；② `message_type` 的**界面枚举只有 text/markdown**（模板 opt_list 实测），写 `image` 会让渠道组件**渲染失败**（「配置参数」整片空白）—— 已编译期拦截；③ `image_content` 是 `flat_array`，**数组项必须是表达式对象、不能是裸标量**：写裸字符串数组时运行期能发出图，但编辑器面板**整片空白**（[M2] wecom-img-95489）；每项包成 `{__exp_config__:{biz_type:string}, __exp_kind__:original, __exp_value__:<url>}` 后**面板完整渲染**，`图片内容` 显示为占位 chip「步骤已被删除」（值保留、运行期照发，真机 [M2] wecom-img-61013）；④ 界面**原生**的图片项是**仪表盘变量**：`{__exp_config__:{}, __exp_kind__:variable, __exp_value__:"$var_dashboard.<id>-{\"dashboard_type\":\"url\"} "}`（界面样本）；「图片内容」控件的值来源弹窗**只有「仪表盘」**，没有自由 URL 输入 —— 所以裸 URL 永远只能显示为兜底 chip。对照：`message_type=markdown` 且 `image_content` 为空的流程面板完整（[M1] wecom-md-94521）。

## whitelist-must-track-editor-catalog

**动作白名单必须以编辑器目录（`/chatflow/api/v2/instruction/search_with_recommend`，只返回 online=1 的指令）为准**：chatflow 的动作是「一代一份 instruction_info」，上一代会被同步脚本显式 `SET online = 0`，编辑器随即不再提供。2026-10-08 真机核对发现白名单里有两个**已被下架**的动作，且二者**能存、能渲染、能跑**（真机 12 个节点图标与右侧参数面板全部正常）——「能跑」不等于「该用」：用户在编辑器里无法新建/配置它们。已按目录迁移：`notify_v3`（自 2025-01 `2501a` 起 online=0）→ `notify_v4`（参数超集：`receiver`/`title`/`message_content`/`record_link` 同名同义，多 `button_type`/`link`）；`send_woa_robot_message_v6`（自 2507/2509 起 online=0，且**无后继专属动作**）→ **移除**，WPS 协作群通知改用 `send_woa_message_v9` 的 `group_id_list`；ET 侧既无 `notify_v3` 也无 `notify_v4` ⇒ 一并移除，通知个人用 `send_woa_message_v9` 的 `person_id_list`。⚠️ 判据只能用**目录接口**：`/instruction/templates` **不过滤 online**，下架动作仍返回模板。

## writerows-return-is-sheet-fields

新增/修改记录（pull 1004）的返回字段就是**写回的那张表的字段**，走与其他表动作相同的「列名 → 列 id」解析（写错字段名编译期报错）。
