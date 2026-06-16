# 二、文档读取与下载

## 1. list_files

#### 功能说明

获取指定文件夹下的子文件列表，通过 `filter_type` 可筛选仅返回文件夹。



#### 调用示例

列出根目录内容：

```json
{
  "drive_id": "8001234567",
  "parent_id": "0",
  "page_size": 50,
  "order": "desc",
  "order_by": "mtime"
}
```


#### 参数说明

- `drive_id` (string, 必填): 云盘 ID
- `parent_id` (string, 必填): 父目录 ID，根目录时为 "0"
- `page_size` (integer, 必填): 每页条数；建议 50；范围 1–500
- `page_token` (string, 可选): 分页 token，首次请求不传
- `order` (string, 可选): 排序方式。可选值：`desc` / `asc`
- `order_by` (string, 可选): 排序字段。可选值：`ctime` / `mtime` / `dtime` / `fname` / `fsize`
- `filter_exts` (string, 可选): 过滤扩展名，以英文逗号分隔，全部小写
- `filter_type` (string, 可选): 按文件类型筛选。可选值：`file` / `folder` / `shortcut`
- `with_permission` (boolean, 可选): 是否返回文件操作权限
- `with_ext_attrs` (boolean, 可选): 是否返回文件扩展属性

#### 返回值说明

```json
{
  "data": {
    "items": [
      {
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
      }
    ],
    "next_page_token": "string"
  },
  "code": 0,
  "msg": "string"
}

```

| 字段 | 类型 | 说明 |
|------|------|------|
| `data.items` | array[FileInfo] | 文件列表，结构见附录 A |
| `data.next_page_token` | string | 下一页 token，为空表示已是最后一页 |


---

## 2. download_file

#### 功能说明

获取文件下载信息。

**`drive_id`**（非必填）：

- **有明确的 drive_id** 必传。
- **没有**：不传。



> 不支持在线文档类型的下载，仅支持上传的二进制文件（.docx / .xlsx / .pdf / .pptx 等）。读取在线文档内容请使用 `read_file` 工具

#### 调用示例

获取下载链接：

```json
{
  "drive_id": "string",
  "file_id": "string",
  "with_hash": true
}
```

file_id：

```json
{
  "file_id": "string",
  "with_hash": true
}
```


#### 参数说明

- `drive_id` (string, 可选): 目标云盘 ID
- `file_id` (string, 必填): 文件 ID
- `with_hash` (boolean, 可选): 是否返回校验值，对应响应里的 hashes
- `internal` (boolean, 可选): 是否返回内网下载地址；默认值：`false`
- `storage_base_domain` (string, 可选): 签发的存储网关地址，根据 base_domain 优先匹配。可选值：`wps.cn` / `kdocs.cn` / `wps365.com`

#### 返回值说明

```json
{
  "data": {
    "hashes": [
      {
        "sum": "string",
        "type": "sha256"
      }
    ],
    "url": "string"
  },
  "code": 0,
  "msg": "string"
}

```

| 字段 | 类型 | 说明 |
|------|------|------|
| `data.url` | string | 下载地址。公网环境下一级域名为 wps.cn 或 kdocs.cn 时需携带登录凭据 |
| `data.hashes` | array | 文件散列值（仅 `with_hash=true` 时返回），公网可能返回 md5/sha1/sha256 中的一个或多个 |
| `data.hashes[].sum` | string | 哈希结果 |
| `data.hashes[].type` | string | 哈希类型：`sha256` / `md5` / `sha1` / `s2s` |


---

## 3. download_attachment

#### 功能说明

查询文档附件的下载信息。根据文件 ID 与附件 ID 获取附件下载链接、名称与大小；链接为有效期内可直接下载的 URL。



> 返回的 url 有时效限制，应在获取后尽快使用
> attachment_id 来源：`upload_attachment` 返回的 `object_id`

#### 调用示例

获取附件下载信息：

```json
{
  "file_id": "string",
  "attachment_id": "1234567890"
}
```


#### 参数说明

- `file_id` (string, 必填): 文件 ID
- `attachment_id` (string, 必填): 附件 ID（通过 `upload_attachment` 返回的 `object_id`）

#### 返回值说明

```json
{
  "code": 0,
  "msg": "ok",
  "data": {
    "url": "https://cdn.example.com/attachments/abc123?token=xxx",
    "filename": "设计稿.png",
    "size": 1048576
  }
}

```

| 字段 | 类型 | 说明 |
|------|------|------|
| `data.url` | string | 附件下载地址，有效期内可直接下载 |
| `data.filename` | string | 附件文件名 |
| `data.size` | number | 附件大小（字节） |


---

## 4. read_file

#### 功能说明

读取云文档正文，按文件格式自动返回 Markdown 或结构化数据。



#### 操作约束

- **前置检查**：用户提供 URL/分享链时可直接传入 url 参数；否则通过 search_files 获取 file_id
- **禁止**：禁止对 .csv 调用本工具；CSV 不支持在线读取，建议转为 .xlsx
- **提示**：整表精细读取或海量数据：表格用 sheet.*，多维表用 dbsheet.*，智能文档块级读取用 otl.*
- **提示**：PDF 拆页、合并、转格式等页级操作用 pdf.*，不要用本工具代替
- **提示**：WPS 文字文档导出为其他格式（如导出 PDF/图片）用 wps.*，本工具仅读取正文内容

**幂等性**：是

> 表格/多维表：未指定 sheet 时使用默认工作表；未传 sheet_range 时读默认首屏区域；单次返回数据有限
> PDF 复杂排版可能有精度损失，提取结果为近似文本
> 必须检查返回的 warnings 字段：内容可能未读全（如仅返回首屏、默认工作表等），warnings 会提示实际读取范围和建议

#### 调用示例

首次调用（通过链接读取）：

```json
{
  "url": "https://www.kdocs.cn/l/example"
}
```

异步轮询（首次返回 status=pending 后）：

```json
{
  "url": "https://www.kdocs.cn/l/example",
  "task_id": "90cacdde6ac0cfafa2c2d1f12fa70220"
}
```

读取表格指定区域：

```json
{
  "file_id": "EQfLmhXnmxMB7UQE4v3urx2YuK982rEbE",
  "sheet_name": "Sheet1",
  "sheet_range": {
    "row_from": 0,
    "row_to": 99,
    "col_from": 0,
    "col_to": 25
  }
}
```


#### 参数说明

- `url` (string, 三选一必填: `url` / `link_id` / `file_id`): 要读取的金山文档链接（与 link_id、file_id 三选一至少填一个）
- `link_id` (string, 三选一必填: `url` / `link_id` / `file_id`): 分享链接 ID
- `file_id` (string, 三选一必填: `url` / `link_id` / `file_id`): 文件 ID（通过 search_files 或 get_file_info 获取）
- `task_id` (string, 可选): 轮询任务 ID；上次返回 data.status=pending 时，与首次调用时传入的 url/link_id/file_id 一并传入
- `format` (string, 可选): 【文档类 docx/doc/pdf/wps/otl】文档内容目标格式。可选值：`markdown` / `plain`（纯文本）/ `kdc`（结构化表示）；默认不传，服务端自动匹配
- `enable_upload_medias` (boolean, 可选): 【文档类 docx/pdf/wps/doc】是否将正文内图片等附件转为可下载 URL，默认 false；仅 format=markdown 或 kdc 时生效
- `sheet_name` (string, 可选): 【表格类 xlsx/ksheet 等、多维表 dbt】工作表或数据表名称
- `sheet_id` (number, 可选): 【表格类、多维表】工作表或数据表 ID；与 sheet_name 同传时优先使用
- `sheet_range` (object, 可选): 【表格类】读取区域，0-based，起止均含；不传则读默认首屏区域
  - `row_from` (number): 起始行
  - `row_to` (number): 结束行
  - `col_from` (number): 起始列
  - `col_to` (number): 结束列

#### 返回值说明

```json
// status=ok（文档类 docx/pdf/otl）
{
  "code": 0,
  "data": {
    "status": "ok",
    "file_id": "nUaJ8MnXS1MKhbGG1VGdrxGJ7AygCANhG",
    "drive_id": "2101657290",
    "name": "报告.docx",
    "suffix": ".docx",
    "content_format": "markdown",
    "content": "# 标题\n\n正文内容…"
  },
  "msg": "ok"
}

// status=pending（需轮询）
{
  "code": 0,
  "data": {
    "status": "pending",
    "task_id": "90cacdde6ac0cfafa2c2d1f12fa70220"
  },
  "msg": "ok"
}

// status=ok（表格类）
{
  "code": 0,
  "data": {
    "status": "ok",
    "file_id": "EQfLmhXnmxMB7UQE4v3urx2YuK982rEbE",
    "drive_id": "2101657290",
    "name": "数据.xlsx",
    "suffix": ".xlsx",
    "content_format": "sheet_range",
    "content": {
      "sheets_info": { "detail": { "sheetsInfo": [{"sheetId": 1, "sheetIdx": 0, "sheetName": "Sheet1", "rowFrom": 0, "rowTo": 4, "colFrom": 0, "colTo": 2}] }, "result": "ok" },
      "range_data": { "detail": { "rangeData": [{"cellText": "姓名", "originRow": 0, "originCol": 0, "understandableType": {"type": "string", "value": "姓名"}}, {"cellText": "100", "originRow": 1, "originCol": 1, "understandableType": {"type": "double", "value": 100}}] }, "result": "ok" }
    },
    "warnings": ["未指定工作表名称，已默认读取第一张可解析的工作表；若需指定表请传入 sheet_name。"]
  },
  "msg": "ok"
}

```

| 字段 | 类型 | 说明 |
|------|------|------|
| `data.status` | string | `ok` 内容就绪 / `pending` 需带 task_id 轮询 |
| `data.task_id` | string | 轮询任务 ID（仅 status=pending 时返回）；下次调用时传入此值 |
| `data.file_id` | string | 文件 ID |
| `data.drive_id` | string | 盘 ID |
| `data.name` | string | 文件名 |
| `data.suffix` | string | 文件后缀（如 .docx .xlsx .otl .pdf） |
| `data.content_format` | string | 内容格式标识，由服务端根据文件类型决定 |
| `data.content` | string|object | 正文内容，类型和结构由 content_format 决定 |
| `data.warnings` | array | 提示信息（仅在有提示时出现，如未指定工作表名称、建议用 otl.block_query 等） |


---

## 5. get_file_info

#### 功能说明

获取文件（夹）信息。通过 `file_id` 获取单个文件或文件夹的详细信息，包含 `drive_id` 等关键字段，可用于获取其他接口所需的 `drive_id`。



#### 调用示例

获取文件信息：

```json
{
  "file_id": "string",
  "with_permission": true,
  "with_drive": true
}
```


#### 参数说明

- `file_id` (string, 必填): 文件（夹）ID
- `with_permission` (boolean, 可选): 是否返回文件操作权限
- `with_ext_attrs` (boolean, 可选): 是否返回文件扩展属性
- `with_drive` (boolean, 可选): 是否返回文件所在 drive 信息

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
    "drive": {
      "allotee_id": "string",
      "allotee_type": "user",
      "company_id": "string",
      "created_by": {
        "avatar": "string",
        "company_id": "string",
        "id": "string",
        "name": "string",
        "type": "user"
      },
      "ctime": 0,
      "description": "string",
      "ext_attrs": [
        { "name": "string", "value": "string" }
      ],
      "id": "string",
      "mtime": 0,
      "name": "string",
      "quota": {
        "deleted": 0,
        "remaining": 0,
        "total": 0,
        "used": 0
      },
      "source": "string",
      "status": "inuse"
    },
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
    "permission": {
      "comment": true,
      "copy": true,
      "copy_content": true,
      "delete": true,
      "download": true,
      "history": true,
      "list": true,
      "move": true,
      "new_empty": true,
      "perm_ctl": true,
      "preview": true,
      "print": true,
      "rename": true,
      "saveas": true,
      "secret": true,
      "share": true,
      "update": true,
      "upload": true
    },
    "shared": true,
    "size": 0,
    "type": "folder",
    "version": 0
  },
  "code": 0,
  "msg": "string"
}

```

返回通用文件信息结构，详见附录 A。当 `with_drive=true` 时额外返回 `drive` 对象（含盘的 id、name、quota 等信息）。


---

