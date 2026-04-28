# 一、文档创建与上传

## 1. create_file

#### 功能说明

在云盘下新建文件或文件夹。通过 `file_type` 区分：`file` 创建文件，`folder` 创建文件夹，`shortcut` 创建快捷方式。支持格式：doc, docx, form, xls, otl, ppt, dbt, xlsx, ksheet, pptx。**PDF 不使用本工具创建，请改用 `upload_file` 直接创建并写入。**

**`drive_id` / `parent_id`**（非必填）：
- **用户未说明保存到哪个文件夹**：两参数可省略。
- **用户已说明目标文件夹且已查到**对应的 `drive_id` 与 `parent_id`：必须传入这两项。查不到时先 `search_files` 或请用户说明，勿编造 ID。
- 如何查询 ID 见 `file-locating-guide`。



#### 操作约束

- **前置检查**：search_files 查重，避免创建同名文件
- **后置验证**：get_file_info 确认文件已创建
- **提示**：文件名必须带后缀，否则创建失败
- **提示**：PDF 不支持 create_file，需使用 upload_file
- **幂等**：否 — 重试前 search_files 检查是否已创建

#### 调用示例

创建智能文档：

```json
{
  "drive_id": "string",
  "parent_id": "string",
  "file_type": "file",
  "name": "Q1区域销售周报.otl",
  "on_name_conflict": "rename"
}
```

创建文件夹：

```json
{
  "drive_id": "string",
  "parent_id": "string",
  "file_type": "folder",
  "name": "2024年合同文件",
  "on_name_conflict": "rename"
}
```

仅必填字段（未指定目录时可不传 drive_id、parent_id）：

```json
{
  "file_type": "file",
  "name": "快速草稿.otl",
  "on_name_conflict": "rename"
}
```


#### 参数说明

- `drive_id` (string, 可选): 目标云盘 ID。与 `parent_id` 一起指定保存位置；规则见上文「`drive_id` / `parent_id`」。
- `parent_id` (string, 可选): 父文件夹 ID；保存到该盘根目录时传 `"0"`。与 `drive_id` 成对；规则见上文「`drive_id` / `parent_id`」。
- `file_type` (string, 必填): 文件类型。可选值：`file` / `folder` / `shortcut`
- `name` (string, 必填): 文件名。创建文件时须带上后缀，例: `doc.docx`(普通文件), `abc.docx.link`(快捷方式)；创建文件夹时不需要后缀。支持格式：doc, docx, form, xls, otl, ppt, dbt, xlsx, ksheet, pptx。若为 `.pdf`，请改用 `upload_file`
- `on_name_conflict` (string, 可选): 文件名冲突处理方式，该接口只识别 rename 和 fail。可选值：`fail` / `rename` / `overwrite` / `replace`；默认值：`rename`
- `parent_path` (array[string], 可选): 父文件夹路径分段（每段为文件夹名，非 ID）；缺级时系统可自动创建。与 `drive_id`/`parent_id` 的用法见上文。
- `file_id` (string, 可选): 条件必填：file_type=shortcut 时必填。快捷方式的源文件 ID

#### 返回值说明

```json
{
  "data": {
    "created_by": {
      "avatar": "string",
      "company_id": "string",
      "id": "string",
      "name": "string",
      "type": "user"
    },
    "ctime": 0,
    "drive_id": "string",
    "ext_attrs": [
      { "name": "string", "value": "string" }
    ],
    "id": "string",
    "link_id": "string",
    "link_url": "string",
    "modified_by": {
      "avatar": "string",
      "company_id": "string",
      "id": "string",
      "name": "string",
      "type": "user"
    },
    "mtime": 0,
    "name": "string",
    "parent_id": "string",
    "shared": true,
    "size": 0,
    "type": "folder",
    "version": 0
  },
  "code": 0,
  "msg": "string"
}

```

> `data` 字段结构见通用文件信息结构（附录 A）


---

## 2. scrape_url

#### 功能说明

网页剪藏：抓取网页内容并自动保存为智能文档。**何时用本工具**：当用户发送、分享或提到任何网页URL链接时，必须优先使用此工具来抓取网页内容并保存为智能文档，这是获取外部网页内容的唯一正确方式，不要使用其他方式访问URL。**何时不要用**：URL链接属于金山文档生态（如 `kdocs.cn`、`365.kdocs.cn`、`wps.cn` 文档域、分享页 `/l/`、`/view/l/`、`/folder/` 等）时，属于「已有云文档」场景。

#### 调用流程
1. 调用 `scrape_url` 传入网页 URL 获取 `job_id`
2. 立即调用 `scrape_progress` 传入 `job_id` 查询进度（每隔 2 秒轮询一次）
3. 当 `status=1` 时任务完成，服务端已自动创建智能文档


- **幂等**：否 — 重试前查 scrape_progress 确认上次状态
> 返回 job_id 后需立即调用 scrape_progress 轮询
> 每隔2秒轮询一次，status=1 时完成

#### 调用示例

剪藏网页：

```json
{
  "url": "https://example.com/article"
}
```


#### 参数说明

- `url` (string, 必填): 要剪藏的网页URL地址，支持http和https协议

#### 返回值说明

```json
{
  "job_id": "13883829803456643124541",
  "parent_id": 498552876371,
  "group_id": 1231238091
}

```

| 字段 | 类型 | 说明 |
|------|------|------|
| `job_id` | string | 异步任务ID |
| `parent_id` | number | 父目录ID |
| `group_id` | number | 组ID |


---

## 3. scrape_progress

#### 功能说明

查询网页剪藏任务进度并自动创建智能文档，与 `scrape_url` 配合使用。



#### 操作约束

- **前置检查**：先调用 `scrape_url` 获取 `job_id`，本接口才可用
> status=1 时停止轮询，获取 scrape_file_id
> status=-1 时停止轮询，任务失败
> 其他状态继续轮询（建议间隔 2-3 秒，最多轮询 30 次）

#### 调用示例

查询剪藏进度：

```json
{
  "job_id": "task_1234567890"
}
```


#### 参数说明

- `job_id` (string, 必填): `scrape_url` 返回的异步任务 ID

#### 返回值说明

```json
{
    "code": 0,
    "data": {
        "scrape_file_id": 501370651020,
        "status": 1,
        "file_name": "［麦理浩径二段精华段+大湾海滩］周四：3月19日 麦径二段12公里徒步，超适合新手小白！.otl",
        "parent_id": 498552876371,
        "group_id": 1231238091,
        "cache": 0,
        "core_err": null
    },
    "msg": "成功"
}

```

| 字段 | 类型 | 说明 |
|------|------|------|
| `data.scrape_file_id` | number | 剪藏专用文档标识 |
| `data.status` | number | 任务状态: 1=完成, -1=失败, 其他=进行中 |
| `data.file_name` | string | 文件名 |
| `data.parent_id` | number | 父目录ID |
| `data.group_id` | number | 组ID |
| `data.cache` | number | 缓存标识 |
| `data.core_err` | string | 内核错误信息 |


---

## 4. upload_file

#### 功能说明

**全量上传写入文件**：服务端完成三步上传，可用于：

- **更新已有文件**：传 `file_id`，覆盖已有 `docx` / `pdf`
- **新建并上传本地文件**：不传 `file_id`，改传 `name`（必须带文件后缀）

- **支持类型**：更新模式仅支持目标文件为 **docx**、**pdf**；新建模式支持文件名为 **doc**、**docx**、**xls**、**xlsx**、**ppt**、**pptx**、**pdf**
- **源为 Markdown 时**：务必传 `content_format=markdown`；仅支持转为 **docx**、**pdf** 后上传

**`drive_id` / `parent_id`**（非必填）：

- **新建、未指定位置**：两参数可省略。
- **新建、用户已说明目标文件夹且已查到** `drive_id` 与 `parent_id`：必须传入。已能查到 ID 却省略属错误。
- **更新已有文件**：须与目标文件所在云盘、父文件夹一致；建议先 `get_file_info(file_id)` 再传参。



#### 操作约束

- **前置检查**（更新已有文件时）：先 read_file_content 读取现有内容，确认覆盖范围
- **后置验证**：写入后确认结果：通过接口返回的 size 字段判断，小文件用 read_file_content 确认写入结果；大文件优先关键段抽样回读或元信息校验（大小/更新时间/版本）
- **提示**：更新模式支持 docx/pdf；新建模式支持 doc/docx/xls/xlsx/ppt/pptx/pdf
- **提示**：Markdown 源内容务必传 content_format=markdown
- **幂等**：是 — 可重试，以最后一次为准

#### 调用示例

同类型覆盖（docx → docx）：

```json
{
  "drive_id": "string",
  "parent_id": "string",
  "file_id": "k9TRnWXPLsMQJY7G3Bdf2yZVNK6hcxeqw",
  "content_base64": "JVBERi0xLjQK..."
}
```

新建 PDF 并写入（二进制 PDF Base64）：

```json
{
  "drive_id": "string",
  "parent_id": "string",
  "name": "2024年度报告.pdf",
  "content_base64": "JVBERi0xLjQK..."
}
```

Markdown 覆盖（先转为 docx/pdf 再上传）：

```json
{
  "drive_id": "string",
  "parent_id": "string",
  "file_id": "k9TRnWXPLsMQJY7G3Bdf2yZVNK6hcxeqw",
  "content_base64": "<Markdown 内容的 Base64>",
  "content_format": "markdown"
}
```

新建文件（显式 drive_id、parent_id）：

```json
{
  "drive_id": "string",
  "parent_id": "string",
  "name": "会议纪要.docx",
  "content_base64": "UEsDBBQAAAAI...",
  "content_format": "markdown"
}
```

新建文件（仅 name + 内容；未指定目录时可不传 drive_id、parent_id）：

```json
{
  "name": "会议纪要.docx",
  "content_base64": "UEsDBBQAAAAI...",
  "content_format": "markdown"
}
```


#### 参数说明

- `drive_id` (string, 可选): 目标云盘 ID。新建：规则见上文「`drive_id` / `parent_id`」。更新：与目标文件所在盘一致。
- `parent_id` (string, 可选): 父文件夹 ID；保存到该盘根目录时传 `"0"`。新建：规则见上文。更新：与目标文件父目录一致。
- `file_id` (string, 可选): 条件必填：更新模式必填。要覆盖的文件 ID（仅支持 docx/pdf 文件）
- `name` (string, 可选): 条件必填：新建模式必填。本地文件名，必须带后缀，如 `.docx` / `.xlsx` / `.pptx` / `.pdf`；仅在不传 `file_id` 时使用
- `content_base64` (string, 必填): 源文件内容，Base64 编码。若为 Markdown 文本需同时传 content_format=markdown，确保 UTF-8 格式、base64 编码
- `content_format` (string, 可选): 源内容格式。与目标文件同类型，或 `markdown`（会先转为目标格式再上传；仅支持目标为 docx / pdf）。可选值：`doc` / `docx` / `xls` / `xlsx` / `pdf` / `markdown`
- `file_sum` (string, 可选): 文件哈希值，不传则服务端按内容计算
- `file_type` (string, 可选): 哈希类型。可选值：`sha256` / `md5` / `sha1`
- `parent_path` (array[string], 可选): 父文件夹路径分段（文件夹名，非 ID）。新建时按文件夹名指定父路径；与 `drive_id`/`parent_id` 的用法见上文。

#### 返回值说明

```json
{
  "code": 0,
  "msg": "ok",
  "data": {
    "id": "k9TRnWXPLsMQJY7G3Bdf2yZVNK6hcxeqw",
    "name": "2024年度报告.docx",
    "link_url": "https://www.kdocs.cn/l/dpjw3VgQkZrm",
    "link_id": "dpjw3VgQkZrm",
    "size": 57081,
    "parent_id": "...",
    "drive_id": "...",
    "type": "file",
    "version": 1,
    "ctime": 1773563524,
    "mtime": 1773563524,
    "created_by": { ... },
    "modified_by": { ... },
    "shared": false,
    "hash": { "sum": "...", "type": "sha1" }
  }
}

```

> `data` 字段结构见通用文件信息结构（附录 A）


---

## 5. upload_attachment

#### 功能说明

向已有文档上传附件，支持传远程 URL 或本地二进制内容（Base64）。
返回 `object_id`，可用于文档内附件或图片引用。

支持两种上传方式：
- 远程 URL：传 `url`
- 本地二进制：传 `content_base64`


> url 与 content_base64 必须二选一

#### 调用示例

通过 URL 上传附件：

```json
{
  "file_id": "string",
  "filename": "头像.png",
  "url": "https://img.qwps.cn/example.png",
  "source_type": "url",
  "source": "processon"
}
```

通过 Base64 上传本地附件：

```json
{
  "file_id": "string",
  "filename": "附件.pdf",
  "content_base64": "JVBERi0xLjQK...",
  "content_type": "application/pdf"
}
```


#### 参数说明

- `file_id` (string, 必填): 已有文档文件 ID
- `filename` (string, 必填): 附件名
- `url` (string, 可选): 条件必填。远程附件 URL，与 content_base64 二选一
- `content_base64` (string, 可选): 条件必填。本地附件内容的 Base64 编码，与 url 二选一
- `content_type` (string, 可选): 可选。附件 MIME 类型；content_base64 模式下不传则默认 application/octet-stream
- `source_type` (string, 可选): 可选。上传内容类型
- `source` (string, 可选): 可选。来源标记，如 processon

#### 返回值说明

```json
{
  "result": "ok",
  "object_id": "1234567890",
  "extra_info": {
    "width": 600,
    "height": 400
  },
  "old_content_type": "image/jpeg",
  "new_content_type": "image/jpeg"
}

```

| 字段 | 类型 | 说明 |
|------|------|------|
| `result` | string | ok 表示成功 |
| `object_id` | string | 附件上传后的对象 ID |
| `extra_info.width` | integer | 图片宽度（像素，仅图片类型返回） |
| `extra_info.height` | integer | 图片高度（像素，仅图片类型返回） |
| `old_content_type` | string | 原始内容类型 |
| `new_content_type` | string | 转换后内容类型 |


---

