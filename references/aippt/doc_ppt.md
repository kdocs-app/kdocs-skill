# 文档生成 PPT

## 1. aippt.doc_create_session

#### 功能说明

为“文档生成 PPT”链路创建服务端 AI 会话，返回后续 `aippt.doc_outline_options` / `aippt.doc_outline` 需要使用的 `session_id`。

**适用于**：已经确定要基于某份文档生成 PPT，准备进入文档解析和大纲生成流程时。



#### 调用示例

创建文档生成会话：

```json
{}
```


#### 参数说明


#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "session_id": "9dbea4d8-b9f7-419c-a4ad-208d4515b8d5"
  }
}

```

| 字段 | 类型 | 说明 |
|------|------|------|
| `code` | number | 0 表示成功 |
| `message` | string | 成功时通常为“成功” |
| `data.session_id` | string | 文档生成链路会话 ID |


---

## 2. aippt.doc_outline_options

#### 功能说明

基于文档引用和会话 ID 调用文档 Agent，对文档做初步解析，并返回一组需要与用户确认的意图问题（questions）。

**适用于**：用户已经给出文档，但还需要补充“制作目标 / 受众 / 重点”等选项时。


> 拿到 `questions` 后，应先与用户交互收集选择，再整理成 `resume_info`

#### 调用示例

获取文档大纲选项：

```json
{
  "session_id": "9dbea4d8-b9f7-419c-a4ad-208d4515b8d5",
  "input": [
    {
      "type": "text",
      "content": "生成PPT"
    },
    {
      "type": "file_id",
      "content": "100239253236"
    }
  ]
}
```


#### 参数说明

- `session_id` (string, 必填): AI 会话 ID，来自 `aippt.doc_create_session`
- `input` (array[object], 必填): 输入内容数组。通常包含一条 `{type:"text", content:"生成PPT"}` 和一条文档引用（如 `{type:"file_id", content:"100239253236"}`）。


#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "session_id": "9dbea4d8-b9f7-419c-a4ad-208d4515b8d5",
    "checkpoint_id": "419e8c77-5442-459e-87eb-2637ba53e132",
    "interrupt_id": "45caf5dc-2dd4-48a7-9ba3-8ba2edc67cdd",
    "user_input": "生成PPT",
    "questions": [
      {
        "type": "choice",
        "field": "制作目标",
        "label": "制作目标",
        "options": ["内部技术培训宣讲", "新功能发布介绍"]
      }
    ]
  }
}

```

| 字段 | 类型 | 说明 |
|------|------|------|
| `data.session_id` | string | 当前 AI 会话 ID |
| `data.checkpoint_id` | string | 继续生成完整大纲时需要的 checkpoint |
| `data.interrupt_id` | string | follow_up 中断 ID |
| `data.user_input` | string | 从 input 中提取的文本输入摘要 |
| `data.questions` | array | 需要向用户确认的问题列表 |
| `data.assistant_messages` | array[string] | 过程中的可见提示文案，可能为空 |


---

## 3. aippt.doc_outline

#### 功能说明

基于 `session_id`、`checkpoint_id`、原始文档输入和 `resume_info`，继续文档 Agent 流程，生成完整 Markdown 大纲。

**适用于**：用户已经完成大纲选项确认，需要拿到可继续做风格美化的正式大纲时。



#### 调用示例

生成文档完整大纲：

```json
{
  "session_id": "9dbea4d8-b9f7-419c-a4ad-208d4515b8d5",
  "checkpoint_id": "419e8c77-5442-459e-87eb-2637ba53e132",
  "input": [
    {
      "type": "text",
      "content": "生成PPT"
    },
    {
      "type": "file_id",
      "content": "100239253236"
    }
  ],
  "resume_info": [
    {
      "type": "follow_up",
      "id": "45caf5dc-2dd4-48a7-9ba3-8ba2edc67cdd",
      "data": {
        "items": [
          {
            "type": "choice",
            "field": "制作目标",
            "label": "制作目标",
            "options": [
              "内部技术培训宣讲"
            ]
          },
          {
            "type": "choice",
            "field": "目标受众",
            "label": "目标受众",
            "options": [
              "技术团队成员"
            ]
          },
          {
            "type": "text",
            "field": "补充说明",
            "label": "补充说明",
            "text_input": "重点突出架构设计和性能优化部分"
          }
        ]
      }
    }
  ]
}
```


#### 参数说明

- `session_id` (string, 必填): AI 会话 ID
- `checkpoint_id` (string, 必填): 检查点 ID，来自 `aippt.doc_outline_options`
- `input` (array[object], 必填): 与获取大纲选项时一致的输入内容数组
- `resume_info` (array[object], 必填): 恢复信息数组，固定包含一个 `follow_up` 对象。结构如下：
  - `type` (string): 固定为 `"follow_up"`
  - `id` (string): 来自 `aippt.doc_outline_options` 返回的 `interrupt_id`
  - `data.items` (array[object]): 与 `doc_outline_options` 返回的 `questions` 保持相同结构，每项包含：
    - `type` (string): 题型，与 questions 中一致（`"choice"` 或 `"text"`）
    - `field` (string): 字段标识，与 questions 中一致
    - `label` (string): 字段名称，与 questions 中一致
    - `options` (array[string]): 选择题时填入用户选中的选项（可多选）
    - `text_input` (string): 文本题时填入用户输入的文本


#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "session_id": "9dbea4d8-b9f7-419c-a4ad-208d4515b8d5",
    "checkpoint_id": "419e8c77-5442-459e-87eb-2637ba53e132",
    "user_input": "生成PPT",
    "user_intention": {
      "items": [
        {
          "field": "制作目标",
          "options": ["内部技术培训宣讲"]
        }
      ]
    },
    "markdown_outline": "# 示例主题 {.topic}\n\n## 封面 {.title}\n- 副标题"
  }
}

```

| 字段 | 类型 | 说明 |
|------|------|------|
| `data.session_id` | string | 当前 AI 会话 ID |
| `data.checkpoint_id` | string | 本次流程对应 checkpoint |
| `data.user_input` | string | 从 input 中提取的文本输入摘要 |
| `data.user_intention` | object | 规范化后的用户意图对象，可直接传给 `aippt.doc_beautify` |
| `data.markdown_outline` | string | 完整 Markdown 大纲 |
| `data.assistant_messages` | array[string] | 过程中的可见提示文案，可能为空 |


---

## 4. aippt.doc_beautify

#### 功能说明

根据完整 Markdown 大纲和用户意图，生成全局风格和逐页排版设计描述。

**适用于**：已经拿到正式大纲，准备进入最终 PPT 渲染前的风格美化阶段。



#### 调用示例

生成文档大纲风格：

```json
{
  "topic": "航空业碳中和监管与合规应对",
  "outline": "# 航空业碳中和监管与合规应对 {.topic}\n\n## 封面 {.title}\n- 汇报主题",
  "user_intention": "{items:[...]}"
}
```


#### 参数说明

- `topic` (string, 必填): 演示文稿主题
- `outline` (string, 必填): 完整 Markdown 大纲
- `user_intention` (object, 必填): 用户意图对象，建议直接传入 `aippt.doc_outline` 返回的 `user_intention`
- `user_input` (string, 可选): 用户原始输入文本；已知时建议传入
- `model` (string, 可选): 风格模型，默认 `IMAGE_V2`

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "topic": "航空业碳中和监管与合规应对",
    "theme": "航空业碳中和监管与合规应对",
    "global_style": "全局风格描述",
    "slides": [
      {
        "index": 0,
        "page_type": "title",
        "design_style": "封面页风格描述"
      }
    ]
  }
}

```

| 字段 | 类型 | 说明 |
|------|------|------|
| `data.topic` | string | 演示主题 |
| `data.theme` | string | 上游生成的主题名 |
| `data.global_style` | string | 全局设计风格说明 |
| `data.slides` | array | 逐页风格数组 |
| `data.slides[].index` | integer | 页序号（从 0 开始） |
| `data.slides[].design_style` | string | 对应页面的风格描述 |


---

## 5. aippt.doc_generate_ppt

#### 功能说明

根据主题与 `outlines` 调用 structppt 接口，生成逐页 HTML PPTX，并返回合并后的演示文件地址。

**适用于**：已经拿到稳定的大纲结构，希望进一步生成可下载的演示文稿结果。



#### 调用示例

生成文档 PPT：

```json
{
  "topic": "航空业碳中和监管与合规应对",
  "outlines": [
    {
      "type": "title",
      "title": "封面",
      "page_type": "pt_title",
      "contents": "",
      "design_style": "封面页风格描述"
    }
  ]
}
```


#### 参数说明

- `topic` (string, 必填): 演示文稿主题
- `outlines` (array[object], 必填): PPT 大纲数组，建议直接传入标准 outlines 结果。每项为 object，至少需为非空对象。


#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "topic": "航空业碳中和监管与合规应对",
    "config": {
      "topic": "航空业碳中和监管与合规应对",
      "outlines": [
        {
          "type": "title",
          "title": "封面",
          "page_type": "pt_title",
          "contents": "",
          "design_style": "封面页风格描述"
        }
      ]
    },
    "merged_url": "https://example.com/final.pptx",
    "total_pages": 12,
    "pages": [
      {
        "slide_index": 0,
        "file_url": "https://example.com/0.pptx",
        "title": "封面",
        "type": "title"
      }
    ]
  }
}

```

| 字段 | 类型 | 说明 |
|------|------|------|
| `data.config` | object | 透传的标准配置对象（topic + outlines） |
| `data.config.outlines[]` | array | 传给 structppt 的标准页面数组 |
| `data.merged_url` | string | 合并后的完整 PPT 地址 |
| `data.total_pages` | integer | 总页数 |
| `data.pages` | array | 逐页生成结果 |


---

