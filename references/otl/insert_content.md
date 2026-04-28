# otl.insert_content

## 1. otl.insert_content

#### 功能说明

向智能文档写入 Markdown/纯文本内容。支持从文档开头或末尾插入，写入时系统自动转换为智能文档富文本格式。

> ⚠️ **仅追加不替换**：本工具只能在文档**开头或末尾追加**内容，不支持替换或修改已有内容。如需替换，应改用 `otl.block_delete` + `otl.block_insert` 组合。
>
> ⚠️ **仅支持 .otl 文件**：传 `.docx`/`.ksheet`/`.dbt`/`.pdf` 等其他格式的 `file_id` 会返回 `InvalidFileKind` 错误，无法通过重试解决。
>
> ⚠️ **`content` 使用 CommonMark 子集**：不支持原生 HTML 标签；不支持 Mermaid/PlantUML 等自定义渲染语法；Unicode 制表图必须用围栏代码块包裹（见下方 param_detail）。违反上述约束会返回 `parse markdown data into prosemirror document error`。



#### 操作约束

- **前置检查**：先 otl.block_query 读取现有内容，了解文档当前状态
- **提示**：仅支持插入操作（begin/end），不支持替换已有内容
- **幂等**：否 — 非幂等操作，重复调用会导致内容重复插入；失败后应先用 otl.block_query 确认文档当前状态，再决定是否重新插入
> 返回 `InvalidArgument` 时不得原样重试；须重构 `content`（剔除 HTML 原生标签、检查 Markdown 合法性）或改用 `otl.block_delete` + `otl.block_insert` 精准操作
> 返回 `InvalidFileKind` 表示 `file_id` 非 .otl，应改用对应类型的工具（.docx → `wps.*`/`upload_file`；.xlsx/.ksheet → `sheet.*`；.dbt → `dbsheet.*`）
> 新建文档首次写入或写入空白文档时**必须传 `title`**：通过 `create_file` 新建的 `.otl` 文档没有标题，若不传 `title`，文档将缺少标题块，显示效果异常。`title` 的值可单独指定，也可从 `content` 中的开头一级标题提取
> `title` 与 `content` 去重：若 `title` 已指定，`content` 开头不应再包含相同的一级标题（`# 标题`），否则文档会出现重复标题。例如 `title` 为 `"珠海一日游"` 时，`content` 应去掉开头的 `# 珠海一日游`，直接从正文或二级标题开始

#### 调用示例

新建文档首次写入（必须带 title）：

```json
{
  "file_id": "string",
  "title": "项目周报",
  "content": "\n\n## 本周进展\n\n- 完成需求评审\n- 启动开发任务",
  "pos": "begin"
}
```

向已有文档开头插入：

```json
{
  "file_id": "string",
  "content": "## 新增章节\n\n这段内容将插入到正文最前面。",
  "pos": "begin"
}
```

在末尾追加：

```json
{
  "file_id": "string",
  "content": "## 补充说明\n\n以上数据截至本周五。",
  "pos": "end"
}
```


#### 参数说明

- `file_id` (string, 必填): 智能文档文件 ID
- `title` (string, 可选): 文档标题
- `content` (string, 必填): 写入内容，支持 Markdown 或纯文本
- `pos` (string, 可选): 插入位置，begin=从文档开头插入，end=在文档末尾追加。可选值：`begin` / `end`；默认值：`begin`

#### Unicode 制表图（架构图、数据流等）

**现象**：若 `content` 里用 `┌` `─` `│` `┐` `└` `┘` `┼` 等 Unicode 制表字符画的架构图、数据流、框图等**落在普通段落或列表里**，智能文档会按富文本渲染（比例字体、空白与换行处理），容易出现**线条错位、框线对不齐**。

**原则**：只对**改写前**的源 Markdown 判断一次——这段制表内容**是否已经**在 Markdown **围栏代码块**里。

| 源内容状态 | 写入 `content` 时怎么做 |
| :--- | :--- |
| **尚未**在围栏代码块里（制表字符裸露在段落、列表等中） | 把**整段**制表内容用围栏代码块包裹再写入；**只包裹图，不包裹全部Markdown内容**；语言标签推荐 `text` 或 `plain`。写入后，该段在文档中一般为 **Plain Text** 代码块，等宽显示，对齐可保留。 |
| **已经**在围栏代码块里（含 `text` / `plain` 或未标注语言的代码块） | **原样写入**，勿再套一层围栏。 |

**示例**：下列片段表示写入 `content` 时，**制表段已被围栏代码块包裹**的推荐形态。

````markdown
## 4.1 整体架构

```text
┌──────────────┐     ┌──────────────┐
│  用户交互层   │────▶│   智能体层    │
└──────────────┘     └──────────────┘
```
````

> **说明**：Mermaid、PlantUML 等需单独渲染引擎的制表图不在此列；此处仅针对**纯文本 Unicode/ASCII 制表图**。


#### 返回值说明

```json
{
  "code": 0,
  "msg": "ok",
  "data": {
    "result": "ok"
  }
}

```

| 字段 | 类型 | 说明 |
|------|------|------|
| `data.result` | string | ok 表示成功 |


---


# 将 Markdown/纯文本数据直接写入智能文档

调用 `otl.insert_content` 工具，向智能文档写入 Markdown 或纯文本内容。支持从文档开头或末尾插入。支持修改文档标题。

## 典型用法

1. **新建文档并写入**：先 `create_file` 创建 `.otl` 文件，再调用 `otl.insert_content` 写入内容。**必须传 `title` 参数**，否则新文档缺少标题，渲染效果差
2. **追加内容**：对已有文档，`pos=end` 在末尾追加新内容
3. **覆盖开头**：`pos=begin` 从文档开头插入（不会删除已有内容，而是插入到前面）
4. 如需精确修改文档中某个块，应使用 `otl.block_query` → `otl.block_delete` / `otl.block_insert` 组合
