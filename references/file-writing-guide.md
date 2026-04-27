# 文件创建与写入指南

> 已有文件（用户提供了 `file_id` 或通过搜索/链接定位到文件）→ 跳过「类型选择」和 `create_file`，直接看各类型的「更新」路径。

#### 云盘位置参数（`drive_id` / `parent_id`）

`create_file` 与 `upload_file`（新建）在接口上 **`drive_id`、`parent_id` 为非必填**；**建议**在调用前按 `file-locating-guide.md` 尽量取得并显式传入，涉及指定目录、共享空间或可复现路径时不要省略。

#### 类型选择决策树

仅在需要新建文件时使用：

```
用户需要创建文档
├── 需要丰富排版/图文混排？ → otl（智能文档）首选
├── 需要表格/数据处理？
│   ├── 简单表格数据 → xlsx
│   ├── 需要多视图/字段管理/看板（智能表格产品形态）→ ksheet（智能表格）
│   └── 需要多数据表、关联、丰富字段类型与甘特/画册等多视图（多维表格产品形态）→ dbt（多维表格）
├── 需要生成 PDF？ → pdf
├── 需要兼容 Word？ → docx
├── 需要生成 PPT？ → pptx
└── 不确定 → otl（智能文档）默认推荐
```

#### 写入流程

**智能文档**（.otl）——**勿用 `upload_file`**：

- 新建：`create_file` → `otl.insert_content` 写入内容
- 更新：`otl.insert_content` 插入内容（`pos=begin` 从开头插入，`pos=end` 在末尾追加）

**文字文档**（.docx）：

- 新建：`create_file` → `upload_file(file_id, content_base64)` 写入内容
- 更新：优先调用 `wps.*` 基础能力做局部编辑；基础能力不支持或在“整篇重写/覆盖上传”场景使用 `upload_file(file_id, content_base64)` 全量覆盖
- Markdown 源内容须传 `content_format="markdown"`

**PDF**（.pdf）——新建无需 `create_file`：

- 新建：`upload_file(drive_id, parent_id, name="xxx.pdf", content_base64=...)`
- 更新：`upload_file(file_id, content_base64=...)` 全量覆盖

**Excel 表格**（.xlsx）：

- 新建：`create_file` → `sheet.update_range_data` 批量写入
- 更新：`sheet.update_range_data` 按范围写入

**智能表格**（.ksheet）：

- 新建：`sheet.create_airsheet_file` → `sheet.update_range_data` 批量写入
- 更新：`sheet.update_range_data` 按范围写入

**多维表格**（.dbt）：

- 新建：`create_file` → `dbsheet.create_sheet` → `dbsheet.create_fields` → `dbsheet.create_records`
- 更新：`dbsheet.update_records` / `dbsheet.create_records` 增改记录

**演示文稿**（.pptx）：

- 新建：`create_file` 或 `upload_file` 上传本地 pptx
- 主题生成型 AI PPT：优先走 `aippt.theme_questions` → `aippt.theme_deep_research` → `aippt.theme_outline` → 本地格式转换 → `aippt.theme_generate_html_pptx` → `upload_file`
