# 在线文字（wps）工具完整参考文档

本文件包含金山文档 Skill 中在线文字（`wps.*`）工具的操作说明。该类工具面向在线编辑中的文字文档，适合创建空白文档、导出和原子能力执行等场景。

---

## 通用说明

### 在线文字特点

- 面向在线文字文档，不是本地 `.docx` 文件直传接口
- 支持创建空白在线文档、导出为 DOCX / PDF / 图片 / AP
- 提供 Core Execute 原子能力，对文档进行段落/区间级别的增删改查和格式设置等操作
- 若只是读取正文内容，仍优先使用通用工具 `read_file`

### 何时使用 `wps.*`
- 需要新建一个空白在线文字文档
- 需要把在线文字导出为 DOCX、PDF、图片或 AP 文稿
- 需要对文档执行原子操作：读取/修改指定段落内容、查找替换、设置段落格式、设置字符格式等

### 何时不要用 `wps.*`
- 创建空白文档 `.docx` 文件：用 `create_empty_file`
- 创建并写入，优先用工具 `create_file_with_content`
- 上传本地 docx/pdf 等文件：用 `upload_new_file`；覆盖已有文档：用 `upload_replace_file`
- 写 Markdown 富文本内容到智能文档：用 `otl.*`

### `wps.*` 工具调用说明

- 格式：服务名和工具分开: 服务名 wps.xx
  例如：kdocs wps.export

## 导出能力总览

`wps.*` 中的导出能力对外拆分为三个工具：

- `wps.export`：导出 DOCX、创建 PDF 导出任务、发起 AP 导出流程
- `wps.export_image`：导出 PNG / JPEG 图片
- `wps.query_export`：统一查询异步导出结果

### 选择建议

- 需要拿到 `.docx` 下载地址：用 `wps.export`，传 `format=docx`
- 需要导出图片：用 `wps.export_image`，传 `format=png/jpeg`  和 `url` / `link_id` / `id` 三选一
- 需要导出 PDF：先 `wps.export`，传 `format=pdf`；再按需用 `wps.query_export`
- 需要导出 AP：先 `wps.export`，传 `format=ap`；再用 `wps.query_export`（`format=ap` 时须传 export 返回的 `file_id`）

## Core Execute 概述

`wps.core_execute` 是在线文字的统一原子操作入口，通过 `command` 选择操作类型，`param` 传递命令参数。

当前已上线 3 个模块。命令查找、完整路由表和参数速查见：[execute.md](wps/execute.md)

| 模块 | 能力 | 详细参考 |
|------|------|---------|
| 文档内容 | 段落/区间读写、查找替换 | [content.md](wps/content.md) |
| 段落格式 | 对齐、缩进、行间距 | [paragraph-format.md](wps/paragraph-format.md) |
| 字符格式 | 字体样式、高亮色 | [character-format.md](wps/character-format.md) |
| 枚举值 | 对齐/行距/颜色/下划线常量 | [enums.md](wps/enums.md) |

---

## 一、导出

### 1. wps.export

#### 功能说明

统一导出在线文字文档，按 `format` 分发到不同导出分支：

- `docx`：返回 DOCX 下载结果
- `pdf`：创建 PDF 导出任务
- `ap`：发起 AP 导出流程



**幂等性**：否 — 导出为异步任务，用 task_id 轮询结果而非重复提交

#### 调用示例

`format=docx` 导出 DOCX：

```json
{
  "link_id": "link_xxx",
  "format": "docx",
  "with_checksums": "md5,sha256"
}
```

`format=pdf` 导出 PDF：

```json
{
  "link_id": "link_xxx",
  "format": "pdf",
  "from_page": 1,
  "to_page": 10
}
```

`format=ap` 导出 AP 文稿：

```json
{
  "link_id": "link_xxx",
  "format": "ap",
  "name": "季度经营分析"
}
```


#### 参数说明

- `url` (string, 三选一必填: `url` / `link_id` / `file_id`): 文档 URL
- `link_id` (string, 三选一必填: `url` / `link_id` / `file_id`): 分享链接 ID
- `file_id` (string, 三选一必填: `url` / `link_id` / `file_id`): 文件 ID
- `format` (string, 必填): 导出格式。可选值：`docx` / `pdf` / `ap`
- `with_checksums` (string, 可选): `format=docx` 时可传，校验算法列表，如 `md5,sha256`
- `cid` (string, 可选): `format=docx` 时可传，分享链接 ID
- `from_page` (number, 可选): `format=pdf` 时可传，起始页码；默认值：`1`
- `to_page` (number, 可选): `format=pdf` 时可传，结束页码；默认值：`9999`
- `client_id` (string, 可选): 导出时可选的客户端标识
- `password` (string, 可选): `format=pdf` 时可传，源文档密码
- `store_type` (string, 可选): `format=pdf` 时可传，如 `ks3`、`cloud`
- `multipage` (number, 可选): `format=pdf` 时可传；默认值：`1`
- `opt_frame` (boolean, 可选): `format=pdf` 时可传；默认值：`true`
- `export_open_password` (string, 可选): `format=pdf` 时可传，导出 PDF 打开密码
- `export_modify_password` (string, 可选): `format=pdf` 时可传，导出 PDF 修改密码
- `name` (string, 可选): `format=ap` 时必填，智能文档名称，不含后缀

---

### 2. wps.export_image

#### 功能说明

将在线文字导出为 `png` 或 `jpeg` 图片。该接口走图片导出链路。



**幂等性**：否 — 导出为异步任务，用 task_id 轮询结果而非重复提交

#### 调用示例

导出为 PNG 长图：

```json
{
  "link_id": "link_xxx",
  "format": "png",
  "dpi": 150,
  "from_page": 1,
  "to_page": 3,
  "combine_long_pic": true
}
```


#### 参数说明

- `url` (string, 三选一必填: `url` / `link_id` / `file_id`): 文档 URL
- `link_id` (string, 三选一必填: `url` / `link_id` / `file_id`): 分享链接 ID
- `file_id` (string, 三选一必填: `url` / `link_id` / `file_id`): 文件 ID
- `format` (string, 必填): 导出图片格式。可选值：`png` / `jpeg`
- `dpi` (number, 可选): 导出图片 DPI。可选值：`96` / `150` / `300`；默认值：`96`
- `water_mark` (boolean, 可选): 是否添加水印；默认值：`true`
- `from_page` (number, 可选): 起始页码；默认值：`1`
- `to_page` (number, 可选): 结束页码；默认值：`9999`
- `combine_long_pic` (boolean, 可选): 是否合并为长图；`false` 表示逐页；默认值：`true`
- `use_xva` (boolean, 可选): 是否启用 XVA 渲染
- `client_id` (string, 可选): 导出时可选的客户端标识
- `password` (string, 可选): 源文档密码
- `store_type` (string, 可选): 存储类型，如 `ks3`、`cloud`

#### 返回值说明

```json
{
  "code": 0,
  "data": {
    "url": "https://xxx.wps.cn/export/image.png",
    "file_id": "string"
  }
}

```

| 字段 | 类型 | 说明 |
|------|------|------|
| `data.url` | string | 导出图片的下载地址 |
| `data.file_id` | string | 导出图片的文件 ID |

---

### 3. wps.query_export

#### 功能说明

统一查询异步导出结果：

- `format=pdf`：查询 PDF 导出任务
- `format=ap`：查询 AP 导出任务



#### 调用示例

`format=pdf` 查询 PDF 导出结果：

```json
{
  "format": "pdf",
  "task_id": "task_xxx",
  "task_type": "normal_export"
}
```

`format=ap` 查询 AP 导出结果：

```json
{
  "format": "ap",
  "file_id": "ap_file_xxx",
  "task_id": "task_xxx"
}
```


#### 参数说明

- `format` (string, 必填): 导出格式。可选值：`pdf` / `ap`
- `task_id` (string, 必填): 导出任务 ID
- `task_type` (string, 可选): `format=pdf` 时可传，通常为 `normal_export`
- `file_id` (string, 可选): `format=ap` 时必填，传 `wps.export` 返回的新智能文档文件 ID
- `extra_query` (object, 可选): `format=ap` 时可传，补充查询参数

---

## 二、文档文本

### 4. wps.read_text

#### 功能说明

按 `action` 读取文档或段落/区间范围内的文本与元信息。

可用 action：
- full_content：全文内容
- page_count：页数
- word_count：字数
- doc_info：文档基本信息
- paragraph_count：段落数量
- paragraph：指定段落文本（需 paragraph_index）
- paragraph_range：段落字符范围（需 paragraph_index）
- paragraph_format：段落格式（需 paragraph_index）
- paragraph_font：段落字体（需 paragraph_index）
- paragraph_page_number：段落所在页码（需 paragraph_index）
- range_content：区间文本（需 begin、end）
- range_font：区间字体（需 begin、end）



> 段落索引从 1 开始；字符位置 begin/end 从 0 开始
> 推荐优先使用本工具，而非 wps.core_execute 的 getFullContent 等老命令
> id 为 file_id，不是 link_id

#### 调用示例

读取全文：

```json
{
  "id": "0adce7c06a112f869cd1d24bbe598cbe",
  "action": "full_content"
}
```

读取第 2 段内容：

```json
{
  "id": "0adce7c06a112f869cd1d24bbe598cbe",
  "action": "paragraph",
  "paragraph_index": 2
}
```

读取字符区间 10–50：

```json
{
  "id": "0adce7c06a112f869cd1d24bbe598cbe",
  "action": "range_content",
  "begin": 10,
  "end": 50
}
```


#### 参数说明

- `url` (string, 三选一必填: `url` / `link_id` / `id`): 文档 URL
- `link_id` (string, 三选一必填: `url` / `link_id` / `id`): 分享链接 ID
- `id` (string, 三选一必填: `url` / `link_id` / `id`): 文件 ID
- `action` (string, 必填): 读取操作类型，见 description.detail 中的 action 列表
- `paragraph_index` (number, 可选): 段落索引，从 1 开始；action 为 paragraph* 时必填
- `begin` (number, 可选): 起始字符位置，从 0 开始；action 为 range* 时必填
- `end` (number, 可选): 结束字符位置；action 为 range* 时必填

#### 返回值说明

```json
{"code": 0, "message": "成功", "data": {"content": "文档正文..."}}

```

---

### 5. wps.write_text

#### 功能说明

按 `action` 在文档、段落或字符区间写入或删除文本。

可用 action：
- insert：在文档追加文本（需 text，可选 is_br 控制是否换行插入）
- append_heading：在文档末尾追加标题（需 text、heading_level）
- delete_all：删除全部正文
- paragraph_insert：在段落前/后插入（需 paragraph_index、text，可选 paragraph_position）
- paragraph_heading_insert：在段落处插入标题（需 paragraph_index、text、heading_level，可选 paragraph_position）
- paragraph_update：更新指定段落的文本内容（需 paragraph_index、content）
- paragraph_delete：删除指定段落内容（需 paragraph_index）
- range_insert：在字符区间插入（需 begin、end、text）
- range_update：更新字符区间的文本内容（需 begin、end、content）
- range_delete：删除字符区间（需 begin、end）



#### 操作约束

- **前置检查**：action=delete_all 会不可逆清空正文，执行前用 read_text 备份或确认；range_delete/paragraph_delete 同理先确认区间

**幂等性**：否 — 删除类操作不可重试；插入类操作重试前先用 read_text 确认当前内容，避免重复插入

> delete_all 会清空文档正文，操作前请确认
> paragraph_position 仅支持 before / after
> append_heading 与 paragraph_heading_insert 需配合 heading_level 使用
> heading_level 推荐使用正整数级别号（1=标题1，2=标题2，...9=标题9），也兼容 WPS 负整数（-2=标题1，-3=标题2）

#### 调用示例

在文档开头插入文本：

```json
{
  "id": "0adce7c06a112f869cd1d24bbe598cbe",
  "action": "insert",
  "text": "【摘要】",
  "position": 0
}
```

在第 1 段后插入正文：

```json
{
  "id": "0adce7c06a112f869cd1d24bbe598cbe",
  "action": "paragraph_insert",
  "paragraph_index": 1,
  "paragraph_position": "after",
  "text": "本段为补充说明。"
}
```

在第 2 段后插入二级标题：

```json
{
  "id": "0adce7c06a112f869cd1d24bbe598cbe",
  "action": "paragraph_heading_insert",
  "paragraph_index": 2,
  "paragraph_position": "after",
  "text": "1.1 引言",
  "heading_level": 2
}
```

在文档末尾追加一级标题：

```json
{
  "id": "0adce7c06a112f869cd1d24bbe598cbe",
  "action": "append_heading",
  "text": "第二章 方法",
  "heading_level": 1
}
```

删除字符区间：

```json
{
  "id": "0adce7c06a112f869cd1d24bbe598cbe",
  "action": "range_delete",
  "begin": 100,
  "end": 150
}
```


#### 参数说明

- `url` (string, 三选一必填: `url` / `link_id` / `id`): 文档 URL
- `link_id` (string, 三选一必填: `url` / `link_id` / `id`): 分享链接 ID
- `id` (string, 三选一必填: `url` / `link_id` / `id`): 文件 ID
- `action` (string, 必填): 写入操作类型
- `text` (string, 可选): 要插入或追加的文本内容
- `content` (string, 可选): 更新后的文本内容（action=paragraph_update/range_update 时必填）
- `is_br` (boolean, 可选): 是否以换行方式插入（action=insert 时可选，默认 false）
- `paragraph_index` (number, 可选): 段落索引，从 1 开始
- `paragraph_position` (string, 可选): 相对段落的位置，before 或 after
- `heading_level` (number, 可选): 标题级别，支持两种传参方式： 1）直接用级别数字（推荐）：1=标题1, 2=标题2, 3=标题3, 4=标题4, 5=标题5, 6=标题6, 7=标题7, 8=标题8, 9=标题9 2）WdBuiltinStyle 负整数：-2=标题1, -3=标题2, -4=标题3, -5=标题4, -6=标题5, -7=标题6, -8=标题7, -9=标题8, -10=标题9, -1=正文

- `begin` (number, 可选): 区间起始字符位置
- `end` (number, 可选): 区间结束字符位置

#### 返回值说明

```json
{"code": 0, "message": "成功", "data": {}}

```

---

### 6. wps.search_replace

#### 功能说明

全文搜索或替换指定文本。

可用 action：
- search：查找 find_text 的出现位置
- replace：将 find_text 替换为 replace_text



#### 操作约束

- **前置检查**：replace 前先用 action=search 确认 find_text 匹配处；is_all=true 时影响全文所有出现处

**幂等性**：否 — replace 不可盲目重试；重试前用 action=search 确认匹配位置与次数

> is_all 省略时默认为全部匹配
> replace 不会修改 find_text 为空时的行为，请确保 find_text 非空

#### 调用示例

搜索关键词：

```json
{
  "id": "0adce7c06a112f869cd1d24bbe598cbe",
  "action": "search",
  "find_text": "季度报告",
  "is_all": true
}
```

替换首处匹配：

```json
{
  "id": "0adce7c06a112f869cd1d24bbe598cbe",
  "action": "replace",
  "find_text": "草稿",
  "replace_text": "定稿",
  "is_all": false
}
```

全文替换：

```json
{
  "id": "0adce7c06a112f869cd1d24bbe598cbe",
  "action": "replace",
  "find_text": "2024",
  "replace_text": "2025",
  "is_all": true
}
```


#### 参数说明

- `url` (string, 三选一必填: `url` / `link_id` / `id`): 文档 URL
- `link_id` (string, 三选一必填: `url` / `link_id` / `id`): 分享链接 ID
- `id` (string, 三选一必填: `url` / `link_id` / `id`): 文件 ID
- `action` (string, 必填): search 或 replace
- `find_text` (string, 必填): 要查找的文本
- `replace_text` (string, 可选): 替换后的文本；action=replace 时必填
- `is_all` (boolean, 可选): 是否匹配/替换全部出现处，默认 true

#### 返回值说明

```json
{"code": 0, "message": "成功", "data": {"matches": [{"begin": 12, "end": 18}]}}

```

---

### 7. wps.format_text

#### 功能说明

通过 `scope` 指定段落级或区间级，再按 `action` 设置格式。

scope：paragraph（段落）或 range（字符区间）

可用 action：
- alignment：对齐方式（alignment）
- font：单属性字体（font_key、font_value）
- highlight：高亮（highlight_color）
- line_spacing：行距（spacing_rule、spacing_value）
- indent：缩进（indent_type、indent_value、indent_unit）
- heading：标题级别（heading_level，正整数 1-9 或负整数）
- format：段落单项格式（key、value）
- format_batch：批量段落格式（format_batch）
- font_color：字体颜色（r、g、b，RGB 分量 0-255）
- font_batch：批量字体（font_style 或 format_batch）
- font_style：字体样式对象（font_style）
- clear_format：清除格式



#### 操作约束

- **前置检查**：scope=range 时先用 read_text 确认 begin/end；scope=paragraph 时确认 paragraph_index

**幂等性**：是 — 格式操作可重复执行，失败或效果不符时可调整参数后再次调用

> scope=range 时必须同时提供 begin 与 end
> font_style 支持多属性自动路由到 font_batch；也可显式使用 action=font_batch + font_items
> action=font 的 font_key 使用 PascalCase（如 Superscript），扩展属性自动走 TEXT_FONT_BATCH
> highlight（高亮）和 font_color（字体颜色）是独立 property，必须分别调用， 不能和 font/font_batch 合并为一次调用
> font_key/font_items 中的 key 使用 PascalCase（如 Bold, Italic, Name, Size, Underline）
> heading_level 推荐使用正整数级别号（1=标题1，2=标题2，...9=标题9），也兼容 WPS 负整数（-2=标题1，-3=标题2），-1=正文

#### 调用示例

第 1 段居中：

```json
{
  "id": "0adce7c06a112f869cd1d24bbe598cbe",
  "scope": "paragraph",
  "action": "alignment",
  "paragraph_index": 1,
  "alignment": 1
}
```

区间加粗：

```json
{
  "id": "0adce7c06a112f869cd1d24bbe598cbe",
  "scope": "range",
  "action": "font_style",
  "begin": 0,
  "end": 20,
  "font_style": {
    "bold": true,
    "size": 14
  }
}
```

第 2 段首行缩进 2 字符：

```json
{
  "id": "0adce7c06a112f869cd1d24bbe598cbe",
  "scope": "paragraph",
  "action": "indent",
  "paragraph_index": 2,
  "indent_type": "firstLine",
  "indent_value": 2,
  "indent_unit": "char"
}
```

区间设置删除线：

```json
{
  "id": "0adce7c06a112f869cd1d24bbe598cbe",
  "scope": "range",
  "action": "font_style",
  "begin": 0,
  "end": 20,
  "font_style": {
    "strike_through": true
  }
}
```

第 1 段设置上标：

```json
{
  "id": "0adce7c06a112f869cd1d24bbe598cbe",
  "scope": "paragraph",
  "action": "font",
  "paragraph_index": 1,
  "font_key": "Superscript",
  "font_value": "true"
}
```

区间设置上标+缩放（前5个字设为上标且缩放200%）：

```json
{
  "id": "0adce7c06a112f869cd1d24bbe598cbe",
  "scope": "range",
  "action": "font_style",
  "begin": 0,
  "end": 5,
  "font_style": {
    "superscript": true,
    "scaling": 200
  }
}
```

区间设置下标+缩放（第6到10个字设为下标且缩放150%）：

```json
{
  "id": "0adce7c06a112f869cd1d24bbe598cbe",
  "scope": "range",
  "action": "font_style",
  "begin": 5,
  "end": 10,
  "font_style": {
    "subscript": true,
    "scaling": 150
  }
}
```

区间批量设置字体（推荐）：

```json
{
  "id": "0adce7c06a112f869cd1d24bbe598cbe",
  "scope": "range",
  "action": "font_batch",
  "begin": 0,
  "end": 20,
  "font_items": [
    {
      "key": "Bold",
      "value": true
    },
    {
      "key": "Italic",
      "value": true
    },
    {
      "key": "Name",
      "value": "仿宋"
    },
    {
      "key": "Size",
      "value": 18
    },
    {
      "key": "Underline",
      "value": 9
    }
  ]
}
```

区间设置字符间距和缩放：

```json
{
  "id": "0adce7c06a112f869cd1d24bbe598cbe",
  "scope": "range",
  "action": "font_style",
  "begin": 0,
  "end": 50,
  "font_style": {
    "spacing": 2.5,
    "scaling": 150
  }
}
```


#### 参数说明

- `url` (string, 三选一必填: `url` / `link_id` / `id`): 文档 URL
- `link_id` (string, 三选一必填: `url` / `link_id` / `id`): 分享链接 ID
- `id` (string, 三选一必填: `url` / `link_id` / `id`): 文件 ID
- `scope` (string, 必填): 作用范围，paragraph 或 range
- `action` (string, 必填): 格式操作类型
- `paragraph_index` (number, 可选): 段落索引（scope=paragraph 时必填）
- `begin` (number, 可选): 区间起始（scope=range 时必填）
- `end` (number, 可选): 区间结束（scope=range 时必填）
- `alignment` (number, 可选): 对齐方式枚举值
- `heading_level` (number, 可选): 标题级别，支持两种传参方式： 1）直接用级别数字（推荐）：1=标题1, 2=标题2, 3=标题3, 4=标题4, 5=标题5, 6=标题6, 7=标题7, 8=标题8, 9=标题9 2）WdBuiltinStyle 负整数：-2=标题1, -3=标题2, -4=标题3, -5=标题4, -6=标题5, -7=标题6, -8=标题7, -9=标题8, -10=标题9, -1=正文

- `highlight_color` (number, 可选): 高亮颜色索引（WdColorIndex 枚举）： 0=自动, 1=黑, 2=蓝, 3=青, 4=绿, 5=品红, 6=红, 7=黄, 8=白, 9=深蓝, 10=深青, 11=深绿, 12=深品红, 13=深红, 14=深黄, 15=深灰, 16=浅灰

- `spacing_rule` (number, 可选): 行距规则
- `spacing_value` (number, 可选): 行距值
- `indent_value` (number, 可选): 缩进值
- `indent_type` (string, 可选): 缩进类型，left / right / firstLine
- `indent_unit` (string, 可选): 缩进单位，pt 或 char
- `font_key` (string, 可选): 字体属性名（action=font 时）。支持：Bold、Italic、Name（字体名）、Size（字号）、 Underline（WdUnderline 枚举值：0=无,1=单线,3=双线,4=虚线,6=粗线,7=短划线,9=点-划线,10=点-点-划线,11=波浪线,20=粗点,23=粗短划线,25=粗-点-划,26=粗-点-点-划,27=粗波浪线,39=长划线,43=双波浪线,55=粗长划线）、 ColorIndex（WdColorIndex 枚举值）、StrikeThrough、DoubleStrikeThrough、 Superscript、Subscript、Spacing（字符间距,磅）、Scaling（字符缩放,百分比）。 注意：font_key 使用 PascalCase 属性名（如 Bold 而非 bold），因为它直接映射到 WPS 内核 Font 对象属性

- `font_value` (string, 可选): 字体属性值（action=font 时）
- `font_style` (object, 可选): 字体样式对象（action=font_style 时）。 支持传入多个属性，系统会自动路由：单个基本属性走 TEXT_FONT， 多属性或含 StrikeThrough/Superscript/Spacing 等扩展属性时自动走 TEXT_FONT_BATCH。 也可直接使用 action=font_batch + font_items 明确指定批量模式。 支持字段：font_name(string)、font_size(float)、bold(bool)、italic(bool)、 underline(int, WdUnderline 枚举)、color_index(int, WdColorIndex 枚举)、 strike_through(bool)、double_strike_through(bool)、 superscript(bool)、subscript(bool)、spacing(float, 磅)、scaling(int, 百分比)。 underline 枚举(WdUnderline)：0=无, 1=单线, 2=仅单词, 3=双线, 4=虚线, 6=粗线, 7=短划线, 9=点-划线, 10=点-点-划线, 11=波浪线, 20=粗点, 23=粗短划线, 25=粗-点-划, 26=粗-点-点-划, 27=粗波浪线, 39=长划线, 43=双波浪线, 55=粗长划线

- `key` (string, 可选): 格式属性名（action=format 时，如 Alignment、LineSpacing）
- `value` (string, 可选): 格式属性值（action=format 时，会自动推断类型）
- `format_batch` (array, 可选): 批量格式项数组（action=format_batch 时）
- `font_items` (array, 可选): 批量字体项数组（action=font_batch 时），每项为 {key, value} 对象。 key 使用 PascalCase WPS Font 属性名：Name（字体名）、Size（字号）、Bold（true/false）、 Italic、Underline（WdUnderline 枚举数值）、Color（RGB 整数值）、ColorIndex（WdColorIndex）、 StrikeThrough、DoubleStrikeThrough、Superscript、Subscript、Spacing、Scaling。 示例：[{"key":"Bold","value":true},{"key":"Size","value":18},{"key":"Name","value":"仿宋"}]。 这是**同时设置多个字体属性的推荐方式**

- `r` (number, 可选): 红色分量 0-255（action=font_color 时）
- `g` (number, 可选): 绿色分量 0-255（action=font_color 时）
- `b` (number, 可选): 蓝色分量 0-255（action=font_color 时）

#### 返回值说明

```json
{"code": 0, "message": "成功", "data": {}}

```

---

## 三、文档表格

### 8. wps.read_table

#### 功能说明

按 `action` 查询文档内表格的数量、尺寸与单元格内容。

可用 action：
- count：表格数量
- dimensions：指定表格行列数（需 table_index）
- cell：单元格内容（需 table_index、row、col）
- row：整行内容（需 table_index、row）
- column：整列内容（需 table_index、col）
- range：表格范围信息（需 table_index）



> table_index、row、col 均从 1 开始
> 修改表格结构请使用 wps.write_table，格式请用 wps.format_table

#### 调用示例

查询表格数量：

```json
{
  "id": "0adce7c06a112f869cd1d24bbe598cbe",
  "action": "count"
}
```

查询第 1 个表格行列数：

```json
{
  "id": "0adce7c06a112f869cd1d24bbe598cbe",
  "action": "dimensions",
  "table_index": 1
}
```

读取单元格 (2,3)：

```json
{
  "id": "0adce7c06a112f869cd1d24bbe598cbe",
  "action": "cell",
  "table_index": 1,
  "row": 2,
  "col": 3
}
```


#### 参数说明

- `url` (string, 三选一必填: `url` / `link_id` / `id`): 文档 URL
- `link_id` (string, 三选一必填: `url` / `link_id` / `id`): 分享链接 ID
- `id` (string, 三选一必填: `url` / `link_id` / `id`): 文件 ID
- `action` (string, 必填): 查询类型
- `table_index` (number, 可选): 表格索引，从 1 开始；除 count 外通常必填
- `row` (number, 可选): 行号，从 1 开始
- `col` (number, 可选): 列号，从 1 开始

#### 返回值说明

```json
{"code": 0, "message": "成功", "data": {"count": 2}}

```

---

### 9. wps.write_table

#### 功能说明

按 `action` 插入、删除或调整表格结构。

可用 action：
- insert：在文档末尾插入新表格（需 rows、cols）
- delete：删除指定表格（需 table_index）
- delete_row：删除行（需 table_index、row）
- delete_column：删除列（需 table_index、col）
- delete_cell_content：清空单元格（需 table_index、row、col）
- delete_all：删除文档内所有表格
- insert_row：插入行（需 table_index、row、position）
- insert_column：插入列（需 table_index、col、position）
- paragraph_insert：在段落处插入表格（需 paragraph_index、paragraph_position、rows、cols）
- range_insert：在字符区间处插入表格（需 begin、end、rows、cols）



#### 操作约束

- **前置检查**：delete/delete_all 等删除操作不可逆，执行前用 read_table 确认 table_index 与行列号

**幂等性**：否 — delete/delete_row/delete_column/delete_all 不可重试；插入类操作重试前先 read_table 确认结构

> delete_all 会移除文档内全部表格，请谨慎使用
> insert_row / insert_column 的 position 表示在目标行/列之前或之后插入
> 单元格内容与样式请用 wps.format_table

#### 调用示例

插入 3×4 表格：

```json
{
  "id": "0adce7c06a112f869cd1d24bbe598cbe",
  "action": "insert",
  "rows": 3,
  "cols": 4
}
```

在第 1 段后插入表格：

```json
{
  "id": "0adce7c06a112f869cd1d24bbe598cbe",
  "action": "paragraph_insert",
  "paragraph_index": 1,
  "paragraph_position": "after",
  "rows": 2,
  "cols": 3
}
```

删除第 1 个表格的第 2 行：

```json
{
  "id": "0adce7c06a112f869cd1d24bbe598cbe",
  "action": "delete_row",
  "table_index": 1,
  "row": 2
}
```


#### 参数说明

- `url` (string, 三选一必填: `url` / `link_id` / `id`): 文档 URL
- `link_id` (string, 三选一必填: `url` / `link_id` / `id`): 分享链接 ID
- `id` (string, 三选一必填: `url` / `link_id` / `id`): 文件 ID
- `action` (string, 必填): 操作类型
- `rows` (number, 可选): 表格行数（插入时）
- `cols` (number, 可选): 表格列数（插入时）
- `table_index` (number, 可选): 表格索引，从 1 开始
- `row` (number, 可选): 行号
- `col` (number, 可选): 列号
- `paragraph_index` (number, 可选): 段落索引
- `paragraph_position` (string, 可选): 相对段落位置，before 或 after
- `begin` (number, 可选): 区间起始字符位置
- `end` (number, 可选): 区间结束字符位置
- `position` (string, 可选): 行列插入相对位置，before 或 after

#### 返回值说明

```json
{"code": 0, "message": "成功", "data": {}}

```

---

### 10. wps.format_table

#### 功能说明

对指定表格（table_index）的单元格、行、列或整表设置格式。

可用 action：
- cell_content、cell_font、cell_alignment、cell_vertical_alignment、cell_background
- row_height、row_font、row_alignment
- column_width、column_font、column_alignment
- borders、table_alignment
- merge_cells（合并矩形区域，需 start_row/start_col/end_row/end_col）
- split_cell、split_table
- merge_row（将指定行的所有单元格合并为一个，需 row）
- merge_column（将指定列的所有单元格合并为一个，需 col）
- append_text、batch_rows
- row_vertical_alignment、row_background、row_height_rule
- column_vertical_alignment、column_background
- split_cell_rows、split_cell_cols

【重要】合并含"最后/前/第N列/行"等相对位置的操作流程：
1. 必须先调 wps.read_table(action=dimensions, table_index=X) 获取 rows 和 cols 总数
2. 根据返回的总列数计算正确的 start_col/end_col（如"最后两列"= start_col=cols-1, end_col=cols）
3. 再调 merge_cells(start_row=1, start_col=计算值, end_row=rows, end_col=计算值)
禁止直接猜测列号！如5列表格的"最后两列"是 start_col=4,end_col=5 而非 1,2



#### 操作约束

- **前置检查**：先用 read_table 确认 table_index、row、col；merge/split 前确认目标区域

**幂等性**：是 — 格式与单元格内容设置可重复执行，失败或效果不符时可调整参数后再次调用

> table_index 为必填；merge_* 与 split_* 需配合行列范围参数
> merge_row 将指定 row 的所有列合并为一个单元格；merge_column 将指定 col 的所有行合并为一个单元格
> 合并跨多行或多列的矩形区域（如合并最后两列、合并前三行的第1-2列）必须使用 merge_cells + start_row/start_col/end_row/end_col，而非 merge_row/merge_column
> 使用 merge_cells 合并'最后N列'等相对位置时，需先调用 read_table(action=dimensions) 获取总行列数，再计算正确的 start_col/end_col
> 创建/删除表格请用 wps.write_table
> batch_data 结构以 API 返回字段为准，示例仅为示意

#### 调用示例

设置单元格文本：

```json
{
  "id": "0adce7c06a112f869cd1d24bbe598cbe",
  "action": "cell_content",
  "table_index": 1,
  "row": 1,
  "col": 1,
  "text": "项目名称"
}
```

合并单元格区域：

```json
{
  "id": "0adce7c06a112f869cd1d24bbe598cbe",
  "action": "merge_cells",
  "table_index": 1,
  "start_row": 1,
  "start_col": 1,
  "end_row": 1,
  "end_col": 3
}
```

合并整行（将第1行所有列合并为一个单元格）：

```json
{
  "id": "0adce7c06a112f869cd1d24bbe598cbe",
  "action": "merge_row",
  "table_index": 1,
  "row": 1
}
```

合并整列（将第2列所有行合并为一个单元格）：

```json
{
  "id": "0adce7c06a112f869cd1d24bbe598cbe",
  "action": "merge_column",
  "table_index": 1,
  "col": 2
}
```

合并最后两列（两步工作流：先查维度再合并）：

```json
{
  "id": "0adce7c06a112f869cd1d24bbe598cbe",
  "action": "merge_cells",
  "table_index": 3,
  "start_row": 1,
  "start_col": 4,
  "end_row": 5,
  "end_col": 5
}
```

合并第一行前两个单元格：

```json
{
  "id": "0adce7c06a112f869cd1d24bbe598cbe",
  "action": "merge_cells",
  "table_index": 3,
  "start_row": 1,
  "start_col": 1,
  "end_row": 1,
  "end_col": 2
}
```

批量更新多行：

```json
{
  "id": "0adce7c06a112f869cd1d24bbe598cbe",
  "action": "batch_rows",
  "table_index": 1,
  "batch_data": [
    {
      "row": 2,
      "cells": [
        {
          "col": 1,
          "text": "A"
        },
        {
          "col": 2,
          "text": "B"
        }
      ]
    }
  ]
}
```


#### 参数说明

- `url` (string, 三选一必填: `url` / `link_id` / `id`): 文档 URL
- `link_id` (string, 三选一必填: `url` / `link_id` / `id`): 分享链接 ID
- `id` (string, 三选一必填: `url` / `link_id` / `id`): 文件 ID
- `action` (string, 必填): 格式操作类型
- `table_index` (number, 必填): 表格索引，从 1 开始
- `row` (number, 可选): 行号（cell_*、row_*、merge_row 等操作需要）
- `col` (number, 可选): 列号（cell_*、column_*、merge_column 等操作需要）
- `text` (string, 可选): 单元格文本（cell_content、append_text）
- `font_style` (object, 可选): 字体样式对象
- `alignment` (number, 可选): 水平对齐方式
- `vertical_alignment` (number, 可选): 垂直对齐方式
- `color_index` (number, 可选): 背景色索引（cell_background）
- `height` (number, 可选): 行高（row_height）
- `width` (number, 可选): 列宽（column_width）
- `line_style` (number, 可选): 边框线样式（borders）
- `table_alignment` (number, 可选): 表格整体对齐方式
- `start_row` (number, 可选): 合并区域起始行
- `start_col` (number, 可选): 合并区域起始列
- `end_row` (number, 可选): 合并区域结束行
- `end_col` (number, 可选): 合并区域结束列
- `num_rows` (number, 可选): 拆分行数（split_cell、split_cell_rows）
- `num_cols` (number, 可选): 拆分列数（split_cell、split_cell_cols）
- `split_at_row` (number, 可选): 拆分表格位置行号（split_table）
- `datas` (array, 可选): 批量行数据（batch_rows）
- `height_rule` (number, 可选): 行高规则（row_height_rule）

#### 返回值说明

```json
{"code": 0, "message": "成功", "data": {}}

```

---

## 四、文档图片

### 11. wps.read_image

#### 功能说明

按 `action` 查询文档内嵌入图片的数量、列表或单张详情。

可用 action：
- count：图片数量
- list：图片列表（索引、尺寸等摘要）
- data：指定图片详情（需 index）



> 图片索引从 1 开始，与 list 返回顺序一致
> 插入/删除图片请用 wps.write_image，改尺寸请用 wps.resize_image

#### 调用示例

查询图片数量：

```json
{
  "id": "0adce7c06a112f869cd1d24bbe598cbe",
  "action": "count"
}
```

列出所有图片：

```json
{
  "id": "0adce7c06a112f869cd1d24bbe598cbe",
  "action": "list"
}
```

获取第 1 张图片详情：

```json
{
  "id": "0adce7c06a112f869cd1d24bbe598cbe",
  "action": "data",
  "index": 1
}
```


#### 参数说明

- `url` (string, 三选一必填: `url` / `link_id` / `id`): 文档 URL
- `link_id` (string, 三选一必填: `url` / `link_id` / `id`): 分享链接 ID
- `id` (string, 三选一必填: `url` / `link_id` / `id`): 文件 ID
- `action` (string, 必填): 查询类型，count / list / data
- `index` (number, 可选): 图片索引，从 1 开始；action=data 时必填

#### 返回值说明

```json
{"code": 0, "message": "成功", "data": {"count": 3}}

```

---

### 12. wps.write_image

#### 功能说明

按 `action` 在文档、段落或字符区间插入图片，或删除已有图片。

可用 action：
- insert：在文档插入图片（需 file_path，可选 width、height）
- delete：删除指定图片（需 index）
- delete_all：删除文档内全部图片
- paragraph_insert：在段落前/后插入（需 paragraph_index、paragraph_position、file_path）
- range_insert：在字符区间插入（需 begin、end、file_path）



#### 操作约束

- **前置检查**：delete/delete_all 不可逆，执行前用 read_image list 确认 index；file_path 须存在且可读

**幂等性**：否 — delete/delete_all 不可重试；插入类操作重试前先 read_image 确认是否已插入

> file_path 须为运行环境可读的本地路径
> 插入后可用 wps.resize_image 调整尺寸
> delete_all 不可恢复，操作前请确认

#### 调用示例

在文档末尾插入图片：

```json
{
  "id": "0adce7c06a112f869cd1d24bbe598cbe",
  "action": "insert",
  "file_path": "/tmp/chart.png",
  "width": 400,
  "height": 300
}
```

在第 2 段后插入图片：

```json
{
  "id": "0adce7c06a112f869cd1d24bbe598cbe",
  "action": "paragraph_insert",
  "paragraph_index": 2,
  "paragraph_position": "after",
  "file_path": "/tmp/logo.png"
}
```

删除第 1 张图片：

```json
{
  "id": "0adce7c06a112f869cd1d24bbe598cbe",
  "action": "delete",
  "index": 1
}
```


#### 参数说明

- `url` (string, 三选一必填: `url` / `link_id` / `id`): 文档 URL
- `link_id` (string, 三选一必填: `url` / `link_id` / `id`): 分享链接 ID
- `id` (string, 三选一必填: `url` / `link_id` / `id`): 文件 ID
- `action` (string, 必填): 操作类型
- `file_path` (string, 可选): 本地或可访问的图片文件路径（插入类 action 时必填）
- `width` (number, 可选): 插入时图片宽度（像素或磅，依 API 约定）
- `height` (number, 可选): 插入时图片高度
- `index` (number, 可选): 图片索引（action=delete 时必填）
- `paragraph_index` (number, 可选): 段落索引
- `paragraph_position` (string, 可选): before 或 after
- `begin` (number, 可选): 区间起始字符位置
- `end` (number, 可选): 区间结束字符位置

#### 返回值说明

```json
{"code": 0, "message": "成功", "data": {}}

```

---

### 13. wps.resize_image

#### 功能说明

对指定索引的图片调整宽高、按比例缩放或恢复原始尺寸。

可用 action：
- resize：设置绝对宽高（需 width、height）
- scale：按比例缩放（需 scale_width、scale_height，百分比）
- reset：恢复原始尺寸



#### 操作约束

- **前置检查**：先用 read_image list 确认图片 index 与当前尺寸

**幂等性**：是 — 尺寸调整可重复执行，效果不符时可再次调用 resize/scale

> index 必填；可先调用 wps.read_image list 确认索引
> scale 的 scale_width / scale_height 为百分比，100 表示保持原尺寸
> reset 不需要 width/height/scale 参数

#### 调用示例

设置为固定宽高：

```json
{
  "id": "0adce7c06a112f869cd1d24bbe598cbe",
  "action": "resize",
  "index": 1,
  "width": 320,
  "height": 240
}
```

按比例缩小一半：

```json
{
  "id": "0adce7c06a112f869cd1d24bbe598cbe",
  "action": "scale",
  "index": 1,
  "scale_width": 50,
  "scale_height": 50
}
```

恢复原始尺寸：

```json
{
  "id": "0adce7c06a112f869cd1d24bbe598cbe",
  "action": "reset",
  "index": 1
}
```


#### 参数说明

- `url` (string, 三选一必填: `url` / `link_id` / `id`): 文档 URL
- `link_id` (string, 三选一必填: `url` / `link_id` / `id`): 分享链接 ID
- `id` (string, 三选一必填: `url` / `link_id` / `id`): 文件 ID
- `action` (string, 必填): resize / scale / reset
- `index` (number, 必填): 图片索引，从 1 开始
- `width` (number, 可选): 目标宽度（action=resize）
- `height` (number, 可选): 目标高度（action=resize）
- `scale_width` (number, 可选): 水平缩放百分比，如 50 表示缩至 50%（action=scale）
- `scale_height` (number, 可选): 垂直缩放百分比（action=scale）

#### 返回值说明

```json
{"code": 0, "message": "成功", "data": {}}

```

---

## 五、文档元素

### 14. wps.read_element

#### 功能说明

通过 `type` 选择元素类别，再按 `action` 查询。

type 取值：bookmark、toc、hyperlink、comment

action 按 type 不同：
- bookmark：count、list、data、exists（data/exists 需 bookmark_name）
- toc：count、exists、data（data 可选 toc_index）
- hyperlink：count、list、data（data 需 index）
- comment：count、list、data、index、author（author 筛选作者）



> type 与 action 组合须匹配，否则会返回参数错误
> 增删改元素请用 wps.write_element
> hyperlink 的 index 从 1 开始，与 list 结果对应

#### 调用示例

统计书签数量：

```json
{
  "id": "0adce7c06a112f869cd1d24bbe598cbe",
  "type": "bookmark",
  "action": "count"
}
```

检查书签是否存在：

```json
{
  "id": "0adce7c06a112f869cd1d24bbe598cbe",
  "type": "bookmark",
  "action": "exists",
  "bookmark_name": "Chapter1"
}
```

列出全部批注：

```json
{
  "id": "0adce7c06a112f869cd1d24bbe598cbe",
  "type": "comment",
  "action": "list"
}
```


#### 参数说明

- `url` (string, 三选一必填: `url` / `link_id` / `id`): 文档 URL
- `link_id` (string, 三选一必填: `url` / `link_id` / `id`): 分享链接 ID
- `id` (string, 三选一必填: `url` / `link_id` / `id`): 文件 ID
- `type` (string, 必填): 元素类型，bookmark / toc / hyperlink / comment
- `action` (string, 必填): 查询操作，见 description.detail
- `bookmark_name` (string, 可选): 书签名称（type=bookmark 且 action=data/exists 时）
- `index` (number, 可选): 元素索引（hyperlink/comment 的 data 等）
- `toc_index` (number, 可选): 目录项索引（type=toc）
- `comment_index` (number, 可选): 批注自定义索引（type=comment，action=index 时）
- `author` (string, 可选): 批注作者（type=comment，action=author 时）

#### 返回值说明

```json
{"code": 0, "message": "成功", "data": {"count": 5}}

```

---

### 15. wps.write_element

#### 功能说明

通过 `type` 选择元素类别，再按 `action` 执行增删改。

type 取值：bookmark、toc、hyperlink、comment

action 按 type 不同：
- bookmark：add、rename、replace_content、delete、paragraph_insert、range_insert
- toc：insert、delete、delete_all、paragraph_insert、range_insert（insert 可选 upper_level、lower_level、toc_index）
- hyperlink：modify_address、delete、delete_all、paragraph_insert、range_insert（需 address、display_text；delete/delete_all 可选 is_del_text）
- comment：paragraph_insert、range_insert（需 text）



#### 操作约束

- **前置检查**：删除类 action 不可逆；执行前用 read_element 确认 type、索引或 bookmark_name

**幂等性**：否 — delete/delete_all 不可重试；增改操作重试前先 read_element 确认状态

> paragraph_insert / range_insert 各 type 含义不同，请对照 action 与必填参数
> rename 需同时提供 bookmark_name 与 new_name
> 查询元素请用 wps.read_element

#### 调用示例

添加书签：

```json
{
  "id": "0adce7c06a112f869cd1d24bbe598cbe",
  "type": "bookmark",
  "action": "add",
  "bookmark_name": "Section_A"
}
```

插入目录：

```json
{
  "id": "0adce7c06a112f869cd1d24bbe598cbe",
  "type": "toc",
  "action": "insert",
  "upper_level": 1,
  "lower_level": 3
}
```

在区间插入超链接：

```json
{
  "id": "0adce7c06a112f869cd1d24bbe598cbe",
  "type": "hyperlink",
  "action": "range_insert",
  "begin": 50,
  "end": 60,
  "address": "https://www.kdocs.cn",
  "display_text": "金山文档"
}
```


#### 参数说明

- `url` (string, 三选一必填: `url` / `link_id` / `id`): 文档 URL
- `link_id` (string, 三选一必填: `url` / `link_id` / `id`): 分享链接 ID
- `id` (string, 三选一必填: `url` / `link_id` / `id`): 文件 ID
- `type` (string, 必填): 元素类型，bookmark / toc / hyperlink / comment
- `action` (string, 必填): 操作类型，见 description.detail
- `bookmark_name` (string, 可选): 书签名称
- `new_name` (string, 可选): 新书签名（action=rename）
- `text` (string, 可选): 书签替换内容或批注正文
- `index` (number, 可选): 超链接/批注等元素索引
- `toc_index` (number, 可选): 目录项索引
- `upper_level` (number, 可选): 目录上限标题级别
- `lower_level` (number, 可选): 目录下限标题级别
- `address` (string, 可选): 超链接 URL
- `display_text` (string, 可选): 超链接显示文本
- `paragraph_index` (number, 可选): 段落索引（paragraph_insert）
- `paragraph_position` (string, 可选): before 或 after
- `begin` (number, 可选): 区间起始（range_insert 或与 range 二选一）
- `end` (number, 可选): 区间结束
- `range` (object, 可选): 范围对象 {"begin": n, "end": m}，与 begin/end 等价
- `is_del_text` (boolean, 可选): 删除超链接时是否同时删除链接文本（type=hyperlink，action=delete/delete_all，默认 false）

#### 返回值说明

```json
{"code": 0, "message": "成功", "data": {}}

```

---

## 六、文档属性

### 16. wps.read_info

#### 功能说明

按 `action` 查询修订、节与样式等文档级属性。

可用 action：
- revision_count：修订记录数量
- revision_status：修订跟踪是否开启
- section_count：节（分节）数量
- section_page_setup：指定节的页面设置（需 section_index）
- style_list：文档可用样式列表



> 修改修订/节属性请用 wps.write_info
> 列表与段落样式请用 wps.set_list_style
> section_index 从 1 开始

#### 调用示例

查询修订数量：

```json
{
  "id": "0adce7c06a112f869cd1d24bbe598cbe",
  "action": "revision_count"
}
```

查询修订跟踪状态：

```json
{
  "id": "0adce7c06a112f869cd1d24bbe598cbe",
  "action": "revision_status"
}
```

读取第 1 节页面设置：

```json
{
  "id": "0adce7c06a112f869cd1d24bbe598cbe",
  "action": "section_page_setup",
  "section_index": 1
}
```


#### 参数说明

- `url` (string, 三选一必填: `url` / `link_id` / `id`): 文档 URL
- `link_id` (string, 三选一必填: `url` / `link_id` / `id`): 分享链接 ID
- `id` (string, 三选一必填: `url` / `link_id` / `id`): 文件 ID
- `action` (string, 必填): 查询类型
- `section_index` (number, 可选): 节索引，从 1 开始；action=section_page_setup 时必填

#### 返回值说明

```json
{"code": 0, "message": "成功", "data": {"revision_count": 3}}

```

---

### 17. wps.write_info

#### 功能说明

按 `action` 修改修订跟踪、接受/拒绝修订，或节的页面设置与删除。

可用 action：
- revision_switch：开关修订跟踪（需 enable）
- revision_accept：接受指定修订（需 revision_index）
- revision_reject：拒绝指定修订（需 revision_index）
- revision_accept_all：接受全部修订
- revision_reject_all：拒绝全部修订
- section_page_setup：设置节页面属性（需 section_index、key、value）
- section_delete：删除节（需 section_index）



#### 操作约束

- **前置检查**：revision_accept_all 不可逆，执行前用 read_info revision_count 确认；section_delete 前确认 section_index

**幂等性**：否 — revision_accept_all、section_delete 不可重试；revision_switch 可重复设置

> revision_accept_all 不可撤销，执行前请确认
> page_setup 字段以 API 文档为准，示例键名仅为示意
> section_delete 可能影响分节与页码，请谨慎操作

#### 调用示例

开启修订跟踪：

```json
{
  "id": "0adce7c06a112f869cd1d24bbe598cbe",
  "action": "revision_switch",
  "enable": true
}
```

接受全部修订：

```json
{
  "id": "0adce7c06a112f869cd1d24bbe598cbe",
  "action": "revision_accept_all"
}
```

设置第 1 节页宽：

```json
{
  "id": "0adce7c06a112f869cd1d24bbe598cbe",
  "action": "section_page_setup",
  "section_index": 1,
  "key": "PageWidth",
  "value": "595"
}
```


#### 参数说明

- `url` (string, 三选一必填: `url` / `link_id` / `id`): 文档 URL
- `link_id` (string, 三选一必填: `url` / `link_id` / `id`): 分享链接 ID
- `id` (string, 三选一必填: `url` / `link_id` / `id`): 文件 ID
- `action` (string, 必填): 操作类型
- `revision_index` (number, 可选): 修订索引（accept/reject 时）
- `enable` (boolean, 可选): 是否启用修订跟踪（revision_switch）
- `section_index` (number, 可选): 节索引，从 1 开始
- `key` (string, 可选): 页面设置属性名（section_page_setup，如 PageWidth、TopMargin）
- `value` (string, 可选): 页面设置属性值（section_page_setup，会自动推断类型）

#### 返回值说明

```json
{"code": 0, "message": "成功", "data": {}}

```

---

### 18. wps.set_list_style

#### 功能说明

通过 `scope` 指定段落级或区间级，再按 `action` 查询/设置列表或应用样式。

scope：paragraph 或 range

可用 action：
- list_query：查询当前列表信息
- list_set：设置列表（需 gallery_type，可选 template_index、level、is_continue）
- style_set：应用命名样式（需 style_name）



#### 操作约束

- **前置检查**：style_set 前用 read_info action=style_list 确认 style_name 存在；list_set 前可用 list_query 查看当前列表

**幂等性**：是 — 列表与样式设置可重复执行，效果不符时可调整 list_info/style_name 后再次调用

> style_name 须为文档已有样式，可用 wps.read_info action=style_list 列举
> gallery_type 为列表样式类型，template_index/level/is_continue 为可选配置
> scope=range 时须提供 begin 与 end

#### 调用示例

查询第 3 段列表信息：

```json
{
  "id": "0adce7c06a112f869cd1d24bbe598cbe",
  "scope": "paragraph",
  "action": "list_query",
  "paragraph_index": 3
}
```

设置有序列表：

```json
{
  "id": "0adce7c06a112f869cd1d24bbe598cbe",
  "scope": "paragraph",
  "action": "list_set",
  "paragraph_index": 3,
  "gallery_type": 2,
  "template_index": 1,
  "level": 1
}
```

对区间应用标题样式：

```json
{
  "id": "0adce7c06a112f869cd1d24bbe598cbe",
  "scope": "range",
  "action": "style_set",
  "begin": 0,
  "end": 15,
  "style_name": "标题 1"
}
```


#### 参数说明

- `url` (string, 三选一必填: `url` / `link_id` / `id`): 文档 URL
- `link_id` (string, 三选一必填: `url` / `link_id` / `id`): 分享链接 ID
- `id` (string, 三选一必填: `url` / `link_id` / `id`): 文件 ID
- `scope` (string, 必填): 作用范围，paragraph 或 range
- `action` (string, 必填): list_query / list_set / style_set
- `paragraph_index` (number, 可选): 段落索引（scope=paragraph 时必填）
- `begin` (number, 可选): 区间起始（scope=range 时必填）
- `end` (number, 可选): 区间结束（scope=range 时必填）
- `gallery_type` (number, 可选): 列表类型（action=list_set 时必填，1=无序/项目符号 2=有序/编号 3=大纲编号）
- `template_index` (number, 可选): 列表模板索引（action=list_set，默认 1）
- `level` (number, 可选): 列表级别（action=list_set，默认 1）
- `is_continue` (boolean, 可选): 是否继续上一列表编号（action=list_set，默认 false）
- `style_name` (string, 可选): 样式名称（action=style_set，可先 read_info style_list 获取）

#### 返回值说明

```json
{"code": 0, "message": "成功", "data": {"list_type": "bullet"}}

```

---

## 七、原子操作（旧版降级）

### 19. wps.core_execute

#### 功能说明

⚠️ 本接口为旧版降级接口，新功能请优先使用 wps.read_text / wps.write_text / wps.search_replace / wps.format_text 等结构化接口。

通过 `id + command + param` 调用在线文字原子能力。
每个 command 对应一种原子操作，param 结构随 command 不同。

**一、文档内容**
- 读取: getFullContent / getParagraphContent / getRangeContent / getParagraphsCount
- 修改: modifyParagraphContent / modifyRangeContent
- 查找替换: findContent / replaceContent

**二、段落格式**
- 对齐: modifyParagraphAlignment / modifyRangeAlignment
- 缩进: modifyParagraph[Left|Right|FirstLine]Indent / modifyRange[Left|Right|FirstLine]Indent
- 行间距: modifyParagraphLineSpacing / modifyRangeLineSpacing

**三、字符格式**
- 字符样式: modifyParagraphFontStyle / modifyRangeFontStyle（key-value 模式）
- 高亮色: modifyParagraphHighlight / modifyRangeHighlight

各命令完整参数与枚举表见 wps 经验文档。



> param 结构随 command 变化，不传则为 {}
> 段落索引 n 从 1 开始，超出范围自动限制到最后一段
> 区间参数 begin/end 为字符位置，从 0 开始
> key-value 类命令（FontStyle）通过 key 选择属性，value 类型随 key 变化

#### 调用示例

读取全文：

```json
{
  "id": "file_xxx",
  "command": "getFullContent"
}
```

读取第 3 段：

```json
{
  "id": "file_xxx",
  "command": "getParagraphContent",
  "param": {
    "n": 3
  }
}
```

修改第 1 段内容：

```json
{
  "id": "file_xxx",
  "command": "modifyParagraphContent",
  "param": {
    "n": 1,
    "str": "新的段落内容"
  }
}
```

全文替换：

```json
{
  "id": "file_xxx",
  "command": "replaceContent",
  "param": {
    "findText": "旧词",
    "replaceText": "新词",
    "isAll": true
  }
}
```

设置段落居中：

```json
{
  "id": "file_xxx",
  "command": "modifyParagraphAlignment",
  "param": {
    "n": 1,
    "algMode": 1
  }
}
```

修改首行缩进 2 字符：

```json
{
  "id": "file_xxx",
  "command": "modifyParagraphFirstLineIndent",
  "param": {
    "n": 1,
    "indent": 2,
    "unit": "ch"
  }
}
```

设置 1.5 倍行距：

```json
{
  "id": "file_xxx",
  "command": "modifyParagraphLineSpacing",
  "param": {
    "n": 1,
    "spacingRule": 1
  }
}
```

设置段落字体加粗：

```json
{
  "id": "file_xxx",
  "command": "modifyParagraphFontStyle",
  "param": {
    "n": 1,
    "key": "Bold",
    "value": true
  }
}
```


#### 参数说明

- `url` (string, 三选一必填: `url` / `link_id` / `id`): 文档 URL
- `link_id` (string, 三选一必填: `url` / `link_id` / `id`): 分享链接 ID
- `id` (string, 三选一必填: `url` / `link_id` / `id`): 文件 ID
- `command` (string, 必填): 原子操作命令名，支持以下值：
文档内容: getFullContent / getParagraphContent / getRangeContent / getParagraphsCount / modifyParagraphContent / modifyRangeContent / findContent / replaceContent
段落格式: modifyParagraphAlignment / modifyRangeAlignment / modifyParagraphLeftIndent / modifyParagraphRightIndent / modifyParagraphFirstLineIndent / modifyRangeLeftIndent / modifyRangeRightIndent / modifyRangeFirstLineIndent / modifyParagraphLineSpacing / modifyRangeLineSpacing
字符格式: modifyParagraphFontStyle / modifyRangeFontStyle / modifyParagraphHighlight / modifyRangeHighlight

- `param` (object, 可选): 命令参数对象，结构随 command 变化。速查：
- getFullContent / getParagraphsCount: 无需参数
- getParagraphContent: {n}
- getRangeContent: {begin, end}
- modifyParagraphContent: {n, str}
- modifyRangeContent: {begin, end, str}
- findContent: {findText, isAll}
- replaceContent: {findText, replaceText, isAll}
- modifyParagraphAlignment: {n, algMode}
- modifyRangeAlignment: {begin, end, algMode}
- modifyParagraph[Left|Right|FirstLine]Indent: {n, indent, unit}
- modifyRange[Left|Right|FirstLine]Indent: {begin, end, indent, unit}
- modifyParagraphLineSpacing: {n, spacingRule, spacingValue}
- modifyRangeLineSpacing: {begin, end, spacingRule, spacingValue}
- modifyParagraphFontStyle: {n, key, value}
- modifyRangeFontStyle: {begin, end, key, value}
- modifyParagraphHighlight: {n, highColor}
- modifyRangeHighlight: {begin, end, highColor}
各命令完整参数说明与枚举值见 wps 经验文档。


#### 返回值说明

```json
{"ok": true, "message": "success", "data": "..."}

```

---


## 工具速查表

| # | 工具名 | 分类 | 功能 | 必填参数 |
|---|--------|------|------|----------|
| 1 | `wps.export` | export | 统一导出在线文字文档 | `url`\|`link_id`\|`file_id`, `format` |
| 2 | `wps.export_image` | export | 将在线文字导出为图片 | `url`\|`link_id`\|`file_id`, `format` |
| 3 | `wps.query_export` | export | 统一查询异步导出结果 | `format`, `task_id` |
| 4 | `wps.core_execute` | execute | [旧版降级] 在线文字原子操作入口，通过 command 指定操作类型 | `url`\|`link_id`\|`id`, `command` |
| 5 | `wps.read_text` | doc_text | 读取在线文字文档的文本内容 | `url`\|`link_id`\|`id`, `action` |
| 6 | `wps.write_text` | doc_text | 在在线文字文档中插入、追加或删除文本 | `url`\|`link_id`\|`id`, `action` |
| 7 | `wps.search_replace` | doc_text | 在在线文字文档中搜索或替换文本 | `url`\|`link_id`\|`id`, `action`, `find_text` |
| 8 | `wps.format_text` | doc_text | 设置在线文字文档的文本格式 | `url`\|`link_id`\|`id`, `scope`, `action` |
| 9 | `wps.read_table` | doc_table | 查询在线文字文档中的表格信息 | `url`\|`link_id`\|`id`, `action` |
| 10 | `wps.write_table` | doc_table | 在在线文字文档中创建或删除表格 | `url`\|`link_id`\|`id`, `action` |
| 11 | `wps.format_table` | doc_table | 设置在线文字文档中表格的格式 | `url`\|`link_id`\|`id`, `action`, `table_index` |
| 12 | `wps.read_image` | doc_image | 查询在线文字文档中的图片信息 | `url`\|`link_id`\|`id`, `action` |
| 13 | `wps.write_image` | doc_image | 在在线文字文档中插入或删除图片 | `url`\|`link_id`\|`id`, `action` |
| 14 | `wps.resize_image` | doc_image | 调整在线文字文档中图片的尺寸 | `url`\|`link_id`\|`id`, `action`, `index` |
| 15 | `wps.read_element` | doc_element | 查询在线文字文档中的元素（书签、目录、超链接、批注） | `url`\|`link_id`\|`id`, `type`, `action` |
| 16 | `wps.write_element` | doc_element | 在在线文字文档中创建、修改或删除元素 | `url`\|`link_id`\|`id`, `type`, `action` |
| 17 | `wps.read_info` | doc_info | 查询在线文字文档的属性信息 | `url`\|`link_id`\|`id`, `action` |
| 18 | `wps.write_info` | doc_info | 修改在线文字文档的属性 | `url`\|`link_id`\|`id`, `action` |
| 19 | `wps.set_list_style` | doc_info | 设置在线文字文档的列表或段落样式 | `url`\|`link_id`\|`id`, `scope`, `action` |

## Core Execute 使用指引

- 命令路由表与参数速查 → [execute.md](wps/execute.md)

## 典型用途

| 场景 | 说明 |
|------|------|
| 空白文档创建 | 新建在线文字后再进入后续编辑流程 |
| 文档导出 | 通过 `wps.export`、`wps.export_image`、`wps.query_export` 完成 |
| AP 生成 | 通过 `wps.export(format=ap)` 与 `wps.query_export(format=ap)` 完成 |
| 内容读写 | 通过 `wps.core_execute` → `getFullContent` / `modifyParagraphContent` 等 完成 |
| 查找替换 | 通过 `wps.core_execute` → `findContent` / `replaceContent` 等 完成 |
| 段落格式 | 通过 `wps.core_execute` → `modifyParagraphAlignment` / `modifyParagraphLineSpacing` 等 完成 |
| 字符样式 | 通过 `wps.core_execute` → `modifyParagraphFontStyle` / `modifyRangeHighlight` 等 完成 |
