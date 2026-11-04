# 自动化动作目录（生成物）

<!-- BEGIN GENERATED: auto-sheet-actions -->
> 本节由构建从平台模板生成（勿手改）。刷新方式见 kso-skills 的 docs/auto-sheet-maintenance.md。

<!-- 生成物：由平台模板生成，勿手改。 -->

来源：平台 `/instruction/templates`（19 个动作，类型：db_sheet、et_sheet）。
「验证级别」来自**我们自己的真机证据**（手写维护，不是平台声明）。

| 动作 | 用途 | 必填参数 | 返回值来源 | 验证级别 |
|------|------|----------|------------|----------|
| `airscript_v2` | 执行AirScript脚本 | `script_id` | 该步骤参数声明的结构（pull 1005） | ✅ 真机验证 |
| `create_sheet` | 创建工作表 | — | ⚠️ **没有返回值** | ✅ 真机验证 |
| `create_wps_schedule_v1` | 创建WPS日程 | `select_schedule_creator`、`schedule_title`、`schedule_start_time`、`schedule_end_time`、`schedule_participants`、`schedule_notes` | ⚠️ **没有返回值** | ✅ 真机验证 |
| `db_delete_rows` | 删除记录 | `sheet_id`、`search_range` | 该步骤查的表字段（pull 1000） | ✅ 真机验证 |
| `db_insert_rows` | 新增记录 | `sheet_id`、`key_values` | pull 1004（未解析，宽容放行） | ✅ 真机验证 |
| `db_ocr_word` | 图像识别 | `file_link`、`ocr_type`、`ocr_word_list` | 图像识别结果（pull 10000，未解析） | ✅ 真机验证 |
| `db_search_rows` | 查找内容 | `sheet_id`、`search_range`、`fields` | 该步骤查的表字段（pull 1000） | ✅ 真机验证 |
| `db_update_rows` | 修改记录 | `sheet_id`、`search_range`、`key_values` | pull 1004（未解析，宽容放行） | ✅ 真机验证 |
| `delay` | 延迟 | `delay_time` | ⚠️ **没有返回值** | ✅ 真机验证 |
| `filter_rows` | 从表格中获取数据 | `info` | pull 8（未解析，宽容放行） | ✅ 真机验证 |
| `notify_v4` | 发送公众号和文档消息 | `receiver`、`title`、`message_content`、`link` | ⚠️ **没有返回值** | ✅ 真机验证 |
| `run_python_script_v2` | 执行PY脚本 | `script_id` | 该步骤参数声明的结构（pull 1005） | ✅ 真机验证 |
| `send_dingtalk_robot_message_v6` | 发送钉钉群聊消息 | `webhook`、`sign_token`、`message_type`、`message_content`、`with_source` | ⚠️ **没有返回值** | ✅ 真机验证 |
| `send_email` | 发送邮件消息 | `to`、`subject`、`content`、`with_source` | ⚠️ **没有返回值** | ✅ 真机验证 |
| `send_feishu_robot_message_with_webhook_v3` | 发送飞书群聊消息 | `webhook`、`message_title`、`message_content`、`with_source` | ⚠️ **没有返回值** | ✅ 真机验证 |
| `send_http_request_v3` | 发送HTTP请求 | `method`、`url`、`query`、`header` | pull 1001（未解析，宽容放行） | ✅ 真机验证 |
| `send_wecom_robot_message_v6` | 发送企业微信群聊消息 | `webhook`、`message_type`、`message_content`、`with_source` | ⚠️ **没有返回值** | ✅ 真机验证 |
| `send_woa_message_v9` | 发送WPS协作消息 | `receiver_type`、`person_id_list`、`group_id_list`、`at_type`、`at_person_list`、`message_title`、`message_content`、`with_source` | ⚠️ **没有返回值** | ✅ 真机验证 |
| `write_data` | 将数据写入到表格 | `info`、`data`、`mode`、`with_header` | ⚠️ **没有返回值** | ✅ 真机验证 |
<!-- END GENERATED: auto-sheet-actions -->
