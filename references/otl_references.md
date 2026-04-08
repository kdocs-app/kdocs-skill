# 智能文档（otl）工具完整参考文档

金山文档智能文档（otl）提供了专属的内容写入接口，支持以 Markdown 格式向文档插入内容（标题、文本、列表等），系统自动转换为富文本格式。

---

## 通用说明

### 智能文档特点

- **推荐度**：⭐⭐⭐ **首选文档格式**
- 排版美观，支持标题、列表、待办、表格、分割线等丰富块组件
- 适合图文混排、报告撰写、知识文档、会议纪要等场景
- 是网页剪藏（`scrape_url`）的默认输出格式

### 创建智能文档

通过 `create_file` 创建，`name` 须带 `.otl` 后缀，`file_type` 设为 `file`：

```json
{
  "name": "项目周报.otl",
  "file_type": "file",
  "parent_id": "folder_abc123"
}
```

创建完成后用下文 **`otl.insert_content`** 写入 Markdown/文本。**勿**对 `.otl` 使用 `upload_file`：该工具面向本地文字/表格/演示/PDF 文件上传，不支持 `.otl` 智能文档。

### 读取智能文档

通过 `read_file_content` 读取，返回 Markdown 格式内容。

#### 步骤 1：提交读取任务

调用参数：

```json
{
  "drive_id": "drive_abc123",
  "file_id": "file_otl_001",
  "format": "markdown",
  "include_elements": ["all"]
}
```

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `drive_id` | string | 是 | 文件所在云盘 ID |
| `file_id` | string | 是 | 文件 ID |
| `format` | string | 是 | 固定传 `"markdown"`。|
| `include_elements` | array | 是 | 固定传 `["all"]`，必须是数组而非字符串。|

> `include_elements` 是数组类型，CLI 调用时需通过 `--args` 传递：
> ```
> read_file_content drive_id=<DRIVE_ID> file_id=<FILE_ID> format=markdown --args '{"include_elements":["all"]}'
> ```

#### 步骤 2：轮询获取内容

将步骤 1 返回的 `task_id` 加入参数再次调用：

```json
{
  "drive_id": "drive_abc123",
  "file_id": "file_otl_001",
  "format": "markdown",
  "include_elements": ["all"],
  "task_id": "步骤1返回的task_id"
}
```

> 轮询时参数与步骤 1 完全相同，仅追加 `task_id`。

> ⚠️ **`include_elements` 必须是数组** `["all"]`，不是字符串 `"all"`。传错类型会导致服务端仅返回段落文本。

---

## 智能文档（otl）专属接口

### 1. otl.insert_content

#### 功能说明

向智能文档写入 Markdown/纯文本内容。支持从文档开头或末尾插入，写入时系统自动转换为智能文档富文本格式。


#### 调用示例

从开头写入：

```json
{
  "file_id": "string",
  "title": "项目周报",
  "content": "# 项目周报\n\n## 本周进展\n\n- 完成需求评审\n- 启动开发任务",
  "pos": "begin"
}
```

在末尾追加：

```json
{
  "file_id": "string",
  "title": "补充内容",
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


#### 操作约束

- **前置检查**：先 read_file_content 读取现有内容，了解文档当前状态
- **提示**：仅支持插入操作（begin/end），不支持替换已有内容

---

### 2. otl.block_insert

#### 功能说明

向智能文档插入一个或多个块，适合在指定父块下按位置追加段落、列表、表格等结构化内容。


#### 调用示例

在文档开头插入段落：

```json
{
  "file_id": "string",
  "params": [
    {
      "blockId": "doc",
      "index": 0,
      "content": [
        {
          "type": "paragraph",
          "content": [
            {
              "type": "text",
              "text": "hello"
            }
          ]
        }
      ]
    }
  ]
}
```


#### 参数说明

- `file_id` (string, 必填): 智能文档文件 ID
- `params` (array, 必填): 插入操作列表，每项为一个插入块对象
  - `blockId` (string, 常用): 目标父块 ID，例如 `doc`
  - `index` (integer, 常用): 插入位置索引
  - `content` (array, 常用): 待插入的块内容数组

#### 返回值说明

```json
{
  "code": 0,
  "msg": "ok",
  "data": {
    "...": "..."
  }
}

```


#### 操作约束

- **前置检查**：先 otl.block_query 了解文档块结构，确认插入位置
- **提示**：返回结果因内容和文档状态不同而异，以 code == 0 判断成功

---

### 3. otl.block_delete

#### 功能说明

删除一个或多个块区间，适合按父块和索引范围删除内容。


#### 调用示例

删除指定块区间：

```json
{
  "file_id": "string",
  "params": [
    {
      "blockId": "父blockId",
      "startIndex": 0,
      "endIndex": 1
    }
  ]
}
```


#### 参数说明

- `file_id` (string, 必填): 智能文档文件 ID
- `params` (array, 必填): 删除操作列表，每项为一个删除区间对象
  - `blockId` (string, 常用): 目标父块 ID
  - `startIndex` (integer, 常用): 删除起始索引
  - `endIndex` (integer, 常用): 删除结束索引

#### 返回值说明

```json
{
  "code": 0,
  "msg": "ok",
  "data": {
    "...": "..."
  }
}

```


#### 操作约束

- **用户确认**：删除操作不可逆，执行前必须向用户确认删除范围
- **前置检查**：先 otl.block_query 确认待删除块的内容，避免误删

---

### 4. otl.block_query

#### 功能说明

查询指定块的结构与内容，适合在更新前先读取目标块信息。


#### 调用示例

查询指定块：

```json
{
  "file_id": "string",
  "params": {
    "blockIds": [
      "目标blockId"
    ]
  }
}
```


#### 参数说明

- `file_id` (string, 必填): 智能文档文件 ID
- `params` (object, 必填): 查询参数对象
  - `blockIds` (array, 常用): 要查询的块 ID 列表

#### 返回值说明

```json
{
  "code": 0,
  "msg": "ok",
  "data": {
    "...": "..."
  }
}

```


---

### 5. otl.convert

#### 功能说明

将 HTML、Markdown 等内容转换为智能文档块结构，适合在正式插入前先生成可复用的块内容。


#### 调用示例

转换内容：

```json
{
  "file_id": "string",
  "params": {
    "...": "..."
  }
}
```


#### 参数说明

- `file_id` (string, 必填): 智能文档文件 ID
- `params` (object, 必填): 转换参数对象。根据待转换内容类型填写对应字段

#### 返回值说明

```json
{
  "code": 0,
  "msg": "ok",
  "data": {
    "...": "..."
  }
}

```


---


## 工具速查表

| # | 工具名 | 功能 | 必填参数 |
|---|--------|------|----------|
| 1 | `otl.insert_content` | 向智能文档插入 Markdown/文本内容 | `file_id`, `content` |
| 2 | `otl.block_insert` | 向智能文档插入一个或多个块 | `file_id`, `params` |
| 3 | `otl.block_delete` | 删除智能文档中一个或多个块区间 | `file_id`, `params` |
| 4 | `otl.block_query` | 查询智能文档指定块的结构与内容 | `file_id`, `params` |
| 5 | `otl.convert` | 将 HTML/Markdown 转换为智能文档块结构 | `file_id`, `params` |

## 工具组合速查

| 用户需求 | 推荐工具组合 |
|----------|-------------|
| 新建文档并写入内容 | `create_file` → `otl.insert_content` |
| 读取现有文档内容 | `read_file_content` |
| 精确修改文档块 | `otl.block_query` → `otl.block_delete` / `otl.block_insert` |
| 外部内容转块后插入 | `otl.convert` → `otl.block_insert` |
