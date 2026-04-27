# AI 主题生成演示文稿

> 主题生成 PPT 标准链路：澄清需求、研究资料、大纲与生成上传

**适用场景**：用户希望从一句话主题生成可编辑的云端演示文稿

## 执行流程

> 该流程对应「主题生成 PPT」标准链路：先澄清需求，再研究资料，再出大纲，最后生成并上传云端演示文稿。
>
> 调用本流程中的步骤时，请为**每一步**单独设置超时时间为 **1800000 毫秒**。20 页以上的 PPT 生成可能需要 20-30 分钟。
>
> 所有中间文件必须写入会话专属临时目录 `$AIPPT_WORK_DIR/`，详见 `references/aippt.md` 中「临时文件管理」章节。

```
步骤 0: 初始化会话工作目录
        → conversation_id 获取方式（按优先级）：
          1. 从 Agent 运行时上下文获取对话标识（如 VS Code Copilot 从模板变量 VSCODE_TARGET_SESSION_LOG 路径末段提取 UUID）
          2. 若当前环境无可用对话标识，则调用系统 UUID 生成器（如 PowerShell: [guid]::NewGuid()；bash: uuidgen）
        → **禁止**使用自定义名称、缩写或硬编码字符串代替 conversation_id
        → 目录名格式固定为 kdocs_aippt_<conversation_id>
        → 在 OS 临时目录下创建该子目录
        → 记录路径为 $AIPPT_WORK_DIR
        → 同时清理超过 24 小时的残留 kdocs_aippt_* 目录

步骤 1: aippt.theme_questions(input="用户主题")
        → 返回 questionnaire
        → 写入 $AIPPT_WORK_DIR/01_questions.json

步骤 2: 使用 AskQuestion 向用户展示问卷并收集选择
        → radio 题：allow_multiple=false，enumNames 转 options
        → checkbox 题：allow_multiple=true，enumNames 转 options
        → input 题：在对话中向用户确认，或回退到 default_answer

步骤 2.5: 整理用户选择为 question_and_answers
        → 将用户选中的 id 映射回 enumNames 文案
        → 生成 [{ question, answer }] 数组
        → 写入 $AIPPT_WORK_DIR/selections.json

步骤 3: aippt.theme_deep_research(input, question_and_answers)
        → 流式提取完整研究资料 references
        → 写入 $AIPPT_WORK_DIR/02_research.json

步骤 4: aippt.theme_outline(input, question_and_answers, references)
        → 返回结构化 outline
        → 写入 $AIPPT_WORK_DIR/03_outline.json

步骤 5: 本地格式转换
        → 把 outline 转成 { topic, outlines[] }
        → 每项必须同时包含 type（原始类型）、page_type（映射类型）、contents（正文，可为空串）、design_style（该页风格描述）
        → page_type 映射：
           title/cover -> pt_title
           toc         -> pt_contents
           chapter     -> pt_section_title
           text/content/section -> pt_text
           end/ending  -> pt_end
        → design_style：优先从 aippt.theme_outline 流式输出提取每页风格；无法获取时根据问卷风格偏好和页面内容独立生成
        → 写入 $AIPPT_WORK_DIR/04_config.json

步骤 6: aippt.theme_generate_html_pptx(topic, outlines)
        → 返回 merged_url
        → 写入 $AIPPT_WORK_DIR/05_ppt_result.json

步骤 7: 获取 drive_id（**建议**执行：`upload_file` 的 `drive_id`、`parent_id` 接口上非必填，本流程仍**建议**步骤 9 显式带入步骤 7 的 `drive_id` 与 `parent_id="0"`）
        → 调用 search_files(type="all", page_size=1, scope=["personal_drive"])
        → 不传 keyword，仅靠 scope=["personal_drive"] 限定范围，确保能返回个人云文档盘中的任意文件
        → 从返回结果中提取 data.data.items[0].file.drive_id
        → 若返回结果 items 为空（个人盘内无任何文件），改用 create_file 在根目录创建目标文件夹（parent_id="0"），从创建结果中获取 drive_id

步骤 8: 下载 PPTX 并转 Base64
        → 从步骤 6 的 05_ppt_result.json 中读取 merged_url
        → 下载 PPTX 二进制文件并转为 Base64 编码

步骤 9: upload_file(drive_id=步骤7所得, parent_id="0", parent_path=["应用", "AI生成PPT"], name="<主题>.pptx", content_base64=...)
        → 新建云端演示文稿（**建议**始终带齐 `drive_id`、`parent_id`）
        → 上传目标目录固定为 **我的云文档/应用/AI生成PPT**：`parent_id="0"` + `parent_path=["应用", "AI生成PPT"]`
        → 若该路径不存在，系统会自动创建

步骤 10: get_file_link(file_id)
        → 返回最终云文档链接
        → 写入 $AIPPT_WORK_DIR/06_cloud_result.json

步骤 11: 清理会话工作目录
        → 递归删除 $AIPPT_WORK_DIR
        → 若步骤 1-10 中任一步失败且无法恢复，也应在报告错误后执行清理
```
