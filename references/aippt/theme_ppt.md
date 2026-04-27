# 主题生成 PPT

## 1. aippt.theme_questions

#### 功能说明

根据用户输入的 PPT 主题或需求描述，生成一组可继续追问用户偏好的问卷题目。

**适用于**：用户只给了一个较宽泛的主题，还需要澄清受众、场景、风格、内容重点等信息时。

- 服务端会固定以 `stream=true` 调用上游接口，但最终工具返回为一次性 JSON 结果
- 建议将返回的问卷继续转成 `question_and_answers`，供 `aippt.theme_deep_research` 与 `aippt.theme_outline` 使用


> 如果主题已经足够明确、无需继续澄清，可跳过本工具，直接组织 `question_and_answers`

#### 调用示例

生成儿童科普主题问卷：

```json
{
  "input": "帮我做一份长颈鹿主题的儿童科普 PPT，用于小学课堂讲解"
}
```


#### 参数说明

- `input` (string, 必填): 用户输入的 PPT 主题、目标或需求描述

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "input": "帮我做一份长颈鹿主题的儿童科普 PPT",
    "questionnaire": [
      {
        "key": "properties1",
        "title": "这份长颈鹿主题的PPT主要用于什么场景？",
        "type": "radio",
        "enum": ["A", "B", "C", "D"],
        "enumNames": ["儿童科普教学", "幼儿园活动展示", "生日派对主题分享", "文创产品提案"],
        "required": false,
        "default_answer": ["儿童科普教学"]
      },
      {
        "key": "properties4",
        "title": "如果目标受众是低龄儿童，您是否希望在PPT大纲中规划互动环节模块？",
        "type": "input",
        "required": false,
        "default_answer": "是"
      }
    ]
  }
}

```

| 字段 | 类型 | 说明 |
|------|------|------|
| `code` | number | 0 表示成功 |
| `message` | string | 成功时通常为“成功” |
| `data.input` | string | 原始主题描述 |
| `data.questionnaire` | array | 生成的问卷列表 |
| `data.questionnaire[].key` | string | 问题唯一键 |
| `data.questionnaire[].title` | string | 问题标题 |
| `data.questionnaire[].type` | string | 题型，如 `radio` / `checkbox` / `input` |
| `data.questionnaire[].enum` | array[string] | 选项编码列表，题型为选择题时可能返回 |
| `data.questionnaire[].enumNames` | array[string] | 选项文案列表，顺序与 `enum` 对齐 |
| `data.questionnaire[].required` | boolean | 是否必答 |
| `data.questionnaire[].default_answer` | any | 默认答案，可能为字符串或字符串数组 |


---

## 2. aippt.theme_deep_research

#### 功能说明

根据用户主题和补充问答执行 AI PPT 联网研究，提炼研究目标完成情况与研究资料。

**适用于**：需要先补充事实资料、案例、背景知识，再继续生成 PPT 大纲时。

- 服务端固定以 `stream=true` 调用上游接口
- 最终工具结果只返回完成摘要，不直接返回完整研究资料文本
- 若调用端传入 MCP `_meta.progressToken`，服务会通过 `notifications/progress` 回传进度
- 若同时传入 `_meta.partialResults=true`，`notifications/progress` 中会追加逐段 partial result chunk
- 服务端还会兼容透传 `notifications/aippt/deep_research` 结构化事件


> 若后续要调用 `aippt.theme_outline`，应从流式通知中消费研究资料文本，并将其作为 `references` 传入
> 最终 tool result 不包含完整 `references`，不要仅依赖最终返回体获取研究内容

#### 调用示例

对儿童科普主题做研究：

```json
{
  "input": "帮我做一份长颈鹿主题的儿童科普 PPT",
  "question_and_answers": [
    {
      "question": "这份 PPT 主要用于什么场景？",
      "answer": "小学科学课课堂讲解"
    },
    {
      "question": "希望突出哪些内容模块？",
      "answer": "外形特征、生活习性、趣味冷知识"
    }
  ]
}
```


#### 参数说明

- `input` (string, 必填): 用户输入的 PPT 主题或需求描述
- `question_and_answers` (array[object], 必填): 问答列表，不能为空。每项含 `question`（string，必填）与 `answer`（string，必填）。


#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "streamed": true,
    "done": true,
    "goal_count": 3,
    "goals_done": 3,
    "learning_count": 6
  }
}

```

| 字段 | 类型 | 说明 |
|------|------|------|
| `code` | number | 0 表示成功 |
| `message` | string | 成功时通常为“成功” |
| `data.streamed` | boolean | 固定为 true，表示研究过程通过流式事件执行 |
| `data.done` | boolean | 是否已完成 |
| `data.goal_count` | integer | 研究目标总数 |
| `data.goals_done` | integer | 已完成的研究目标数 |
| `data.learning_count` | integer | 已提取的研究资料条数 |


---

## 3. aippt.theme_outline

#### 功能说明

根据主题描述、补充问答以及研究资料，生成用于 AI PPT 的结构化大纲结果。

**适用于**：已经明确主题方向，并且希望先拿到可审阅的大纲，再决定是否继续生成 HTML PPTX。

- 服务端会固定以 `stream=true` 调用上游 `image_to_slide/outline` 接口
- `references` 建议直接传入 `aippt.theme_deep_research` 流式阶段产出的研究资料文本
- `question_and_answers` 不能为空，否则服务端会直接报参数错误


> outline 的具体内部结构由上游模型决定，后续若要生成 HTML PPTX，建议优先从其中提取正式的 outlines 数组再传给 aippt.theme_generate_html_pptx

#### 调用示例

生成儿童科普主题大纲：

```json
{
  "input": "帮我做一份长颈鹿主题的儿童科普 PPT",
  "question_and_answers": [
    {
      "question": "这份 PPT 主要用于什么场景？",
      "answer": "小学科学课课堂讲解"
    },
    {
      "question": "希望整体风格更偏向哪种？",
      "answer": "插画风、适合儿童"
    }
  ],
  "references": "长颈鹿是现存最高的陆生动物，舌头很长，常以树叶为食。"
}
```


#### 参数说明

- `input` (string, 必填): 用户输入的 PPT 主题或需求描述
- `question_and_answers` (array[object], 必填): 问答列表，不能为空。每项含 `question`（string，必填）与 `answer`（string，必填）。

- `references` (string, 可选): 研究资料文本，建议直接传入 `aippt.theme_deep_research` 产出的研究内容

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "input": "帮我做一份长颈鹿主题的儿童科普 PPT",
    "question_and_answers": [
      {
        "question": "这份 PPT 主要用于什么场景？",
        "answer": "小学科学课课堂讲解"
      }
    ],
    "references": "长颈鹿是现存最高的陆生动物，主要生活在非洲稀树草原。",
    "outline": {
      "type": "outline",
      "title": "长颈鹿主题演示",
      "slides": [
        { "title": "封面" },
        { "title": "习性介绍" }
      ]
    }
  }
}

```

| 字段 | 类型 | 说明 |
|------|------|------|
| `code` | number | 0 表示成功 |
| `message` | string | 成功时通常为“成功” |
| `data.input` | string | 原始主题描述 |
| `data.question_and_answers` | array | 原样回传的问答列表 |
| `data.references` | string | 原样回传的研究资料文本 |
| `data.outline` | object | 上游返回的结构化大纲对象；字段会随模板与场景变化 |


---

## 4. aippt.theme_generate_html_pptx

#### 功能说明

根据主题与 `outlines` 调用 structppt 接口，生成逐页 HTML PPTX，并返回合并后的演示文件地址。

**适用于**：已经拿到稳定的大纲结构，希望进一步生成可下载的演示文稿结果。

- 服务端固定使用 `json2ppt-banana` 场景
- `outlines` 每项须包含 `title`、`content_description`、`design_style`、`page_type` 四个必填字段；推荐传入本地格式转换后的标准 outlines
- 返回值同时包含逐页结果和合并后的完整 PPTX 链接


> 返回的 `merged_url` 和 `pages[].file_url` 一般为临时链接，调用后应尽快消费
> `outlines` 各字段值不得为空字符串，否则服务端返回 badRequest
> `page_type` 只接受上述 5 个枚举值，其他值会导致 badRequest

#### 调用示例

生成儿童科普主题 HTML PPTX：

```json
{
  "topic": "长颈鹿：自然奇迹与文化象征",
  "outlines": [
    {
      "title": "长颈鹿：自然奇迹与文化象征",
      "content_description": "封面页展示主题标题，副标题说明这是一份探索长颈鹿生物学特征、进化奥秘与文化内涵的科普报告。",
      "design_style": "--- 本页版式 ---\n画面铺满全屏，标题居中偏上布局，副标题紧随其下，整体呈现简洁大气的博物馆档案风格",
      "page_type": "pt_title"
    },
    {
      "title": "目录",
      "content_description": "展示本次报告的递进式认知结构：从生物构造基础到进化理论，再到文化历史关联。",
      "design_style": "--- 本页版式 ---\n画面铺满全屏，标题位于上方居中，内容以纵向分布的四级目录列表呈现",
      "page_type": "pt_contents"
    },
    {
      "title": "探索永无止境",
      "content_description": "结束页总结长颈鹿从生物进化奇迹到文化符号的多重价值。",
      "design_style": "--- 本页版式 ---\n画面铺满全屏，居中布局，主标题位于视觉中心",
      "page_type": "pt_end"
    }
  ]
}
```


#### 参数说明

- `topic` (string, 必填): 演示文稿标题
- `outlines` (array[object], 必填): PPT 大纲数组，每项须同时包含以下四个必填字段（缺少任何一个都会返回 badRequest）。数组至少 1 项，首项 `page_type` 建议为 `pt_title`，末项建议为 `pt_end`
  - `title` (string, 必填): 该页标题，不得为空
  - `content_description` (string, 必填): 该页正文与内容要点描述，不得为空
  - `design_style` (string, 必填): 该页版式与视觉风格描述，不得为空
  - `page_type` (string, 必填): 页面类型，仅支持：`pt_title` / `pt_contents` / `pt_section_title` / `pt_text` / `pt_end`

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "topic": "长颈鹿主题演示",
    "merged_url": "https://meihua-service.ks3-cn-beijing.ksyuncs.com/file/20260401/pptx/b74af503-ef35-445f-8106-2c1deac0dcb3.pptx?Expires=1775027041&AWSAccessKeyId=xxx&Signature=xxx",
    "total_pages": 4,
    "pages": [
      {
        "slide_index": 3,
        "file_url": "https://meihua-service.ks3-cn-beijing.ksyuncs.com/temp/20260401/pptx/3097fecf-bf39-454a-92eb-700b49e0e773.pptx?Expires=1775199540&AWSAccessKeyId=xxx&Signature=xxx",
        "title": "探索永无止境",
        "type": "ending"
      }
    ]
  }
}

```

| 字段 | 类型 | 说明 |
|------|------|------|
| `code` | number | 0 表示成功 |
| `message` | string | 成功时通常为“成功” |
| `data.topic` | string | 演示文稿标题 |
| `data.merged_url` | string | 合并后的完整 PPTX 下载地址 |
| `data.total_pages` | integer | 总页数 |
| `data.pages` | array | 逐页生成结果 |
| `data.pages[].slide_index` | integer | 幻灯片序号 |
| `data.pages[].file_url` | string | 单页 PPTX 文件地址 |
| `data.pages[].title` | string | 对应大纲标题 |
| `data.pages[].type` | string | 对应大纲类型，如 `cover` / `content` / `ending` |


---

