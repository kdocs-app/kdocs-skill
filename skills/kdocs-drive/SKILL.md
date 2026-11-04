# 文件管理（工具路由索引）

**适用范围**：支持云盘文件搜索定位、创建上传、读取下载、目录浏览、移动、复制、重命名、分享与协作者权限管理、标签收藏、历史版本查询、评论查询与发表、回收站恢复及网页剪藏，以及在线 Markdown 的创建、读取、覆盖更新、图片转存和导出。

**不适用范围**：其他组件的内容、格式与结构编辑及 AI 演示生成。创建后写入失败，保留文件并交回补写。遇到上述域外需求、需要借助其他组件定位文件或准备外部资源，或无法判断请求应由哪个组件处理时，[返回主 Skill](../../SKILL.md)安排，不自行切换子 Skill。

以下工具不可逆，调用前必须向用户确认；参数与边界以 `kdocs-cli help <tool>` 的工具卡 为准：

`cancel_share`、`cancel_collaborator_permissions`

> **通用返回字段**：所有接口均返回 `code`（状态码，0 表示成功）和 `msg`（人可阅读的文本信息）。

> **工具填参**：下表每行 `<tool>` 选定后须 `kdocs-cli help <tool>`（可批量：`kdocs-cli help drive.create_empty_file drive.list_document_comments`）。

## 操作域路由

先判定操作域，再打开对应章节或 reference。

| 操作域 | 触发场景 | 路由 |
|--------|---------|------|
| 云文档管理 | 搜索定位、创建上传、通用读取下载、分享移动、标签收藏、回收站、网页剪藏 | 见下方「云文档管理」 |
| 在线 Markdown | 新建/上传/覆盖/读取/导出在线 `.md`、图片转存为 `kdocsmd://` | `references/markdown.md` |

## 云文档管理

先按下方工具列表选型，再执行 `kdocs-cli help <tool>` 获取参数；高频创建示例见「读写主路径」。

| 操作 | 触发场景 | 路由 |
|------|---------|------|
| 创建/写入 | 新建并写入、上传本地文件、新建空白文档 | 见下方「创建/写入」 |
| 定位文件 | 搜索/按链接查文件标识或属性/浏览目录 | `references/file-locating-guide.md` |

### 创建/写入

| 用户意图 | 工具 | 适用后缀 / 说明 |
|----------|------|----------|
| 仅需空白文档 | `create_empty_file` | .doc .docx .otl .dbt（默认列） .xlsx .xls .ksheet .pptx .ppt |
| 已有正文或表格数据要写入 | `create_file_with_content` | .otl .docx .pdf（统计分析/归档报告除外） .xlsx .ksheet .dbt |
| 新建多维表且须自定义列 | `create_file_with_content` | .dbt（无业务数据也须 fields+records） |
| 本地 Markdown 新建为办公文档 | `create_file_with_content` | .otl .docx .pdf；原样 .md 才用 `upload_new_file` |
| 云 Word 转智能文档 | 见 `references/workflows/docx-to-otl.md` | .docx/.doc → 新建 .otl |
| 通过上传本地文件新建云文档 | `upload_new_file` | .doc .docx .xls .xlsx .ppt .pptx .pdf .md .txt .html .zip .png .jpg .jpeg .csv .json .dps .et .wps .gif |
| 从多维表/表格生成统计分析或归档 PDF | 尚无本地 PDF 时返回主 Skill，安排生成并预览；已有本地 PDF 时用 `upload_new_file` | .pdf |
| AI 生成 PPT | `aippt.execute_round` | .pptx |

后缀不确定时默认 `.otl`。指定文件夹时先按 `references/file-locating-guide.md` 取 `drive_id`、`parent_id`。

选定工具后，执行 `kdocs-cli help drive.create_empty_file drive.create_file_with_content drive.upload_new_file aippt.execute_round` 获取参数约束（对上表某一工具亦可单独 `kdocs-cli help <tool>`）。


要保留在线 `.md`（含图片转存）见 `references/markdown.md`。

## 读写主路径

本节仅提供 `read_file`、`create_file_with_content` 的最小示例。其他工具先从本子 Skill 的工具列表选型，再执行 `kdocs-cli help <tool>` 获取参数；调用方式由主 Skill「调用格式」统一说明。

### 读：`read_file`

`read_file` 是正文读取工具，定位参数 `url` / `link_id` / `file_id` 三选一。参数示例：

```json
{"url": "https://www.kdocs.cn/l/xxx"}
```

按已有定位信息选择一个参数即可：用户给完整链接传 `url`；已解析分享链接传 `link_id`；已拿到文件 ID 传 `file_id`。不要同时传多个定位参数。

首次请求返回 `status=pending` 时，携带该响应中的 `task_id` 和原读取参数再次请求，将返回文档全量内容：

```json
{"file_id": "file_xxx", "task_id": "task_xxx"}
```

`task_id` 必须来自本次响应，禁止猜测或跨请求复用。

校验新建的 `.otl` 时，显式传 `format=markdown` 读取正文，避免误把结构化内容当作 Markdown。

必须检查返回里的 `data.warnings`：表格可能只读默认工作表或首屏区域，warnings 会提示实际读取范围。

**智能文档（.otl）内嵌图片**：`read_file` 默认返回图片临时可下载 URL（约 10 分钟有效）；显式传 `enable_upload_medias=false` 或图片本身为空/不可转时，图片为 `![image]()` 空占位，不含可下载资源，不要据此调 `download_attachment`。

**在线 Markdown**：`read_file` 默认返回 Markdown 源码（含 `kdocsmd://` 图片协议）；`enable_upload_medias` **不会**把 `kdocsmd://` 转为 https。用户给链接 / file_id 直接读，仅有文件名时先 `search_files`。仅读取正文、不保存本地文件 → 见 `references/workflows/read-cloud-md.md`。

**在线 Markdown 图片**：

| 意图 | 怎么做 |
|------|--------|
| 本地文件上传为在线 Markdown，图片转为 `kdocsmd://` | `references/workflows/upload-md-raw.md` |
| 覆盖已有在线 Markdown，并转存图片 | `references/workflows/replace-cloud-md.md` |
| 对话/粘贴正文新建在线 Markdown，并转存图片 | `references/workflows/create-cloud-md.md` |
| 导出到本地（默认 https 临时链，可选 assets） | `references/workflows/download-cloud-md.md` |

规则详见 `references/markdown.md`。

**otl/docx/pdf → Markdown 文本**：`read_file(format=markdown)` 做格式转换提取；在线 Markdown 读源码不传 format（默认 markdown）。

### 写：`create_file_with_content`

`create_file_with_content` 是“新建并写入内容”的主路径，仅新建使用 `create_empty_file`。目标目录明确时只传 `parent_id`（文件夹 id）；未指定目录时盘和目录都可省略（默认个人云盘根）。参数示例：

```json
{
  "name": "周报",
  "file_extension": "otl",
  "content": "# 周报\n\n这里是正文"
}
```

| 后缀 | 关键参数 |
|------|----------|
| `.otl` `.docx` `.md` `.pdf` | `name` + `file_extension` + `content` |
| `.xlsx` `.ksheet` | `name` + `file_extension` + `sheet_name` + `rangeData` |
| `.dbt` | `name` + `file_extension` + `sheet_name` + `fields` + `records` |

创建表格时，`rangeData` 为对象数组，`formula` 为二维数组。`sheet.update_range_data` 的 `rangeData` 每项需包含 `opType` 和四个 camelCase 坐标，写值或公式时 `formula` 为字符串；两种结构不要混用。

```json
{
  "name": "台账",
  "file_extension": "xlsx",
  "sheet_name": "Sheet1",
  "rangeData": [{
    "row_from": 0,
    "row_to": 0,
    "col_from": 0,
    "col_to": 1,
    "formula": [["姓名", "部门"]]
  }]
}
```

**多维表关联与查找引用**：含 `Link` / `Lookup` / `OneWayLink`，或枚举列必须为 `SingleSelect` 时，禁止用 `create_file_with_content` 一次灌入业务 CSV / records。先用 `create_empty_file` 新建空表，再[返回主 Skill](../../SKILL.md#操作域路由)安排多维表子 Skill 按「关联与查找引用」创建字段并写入记录。仅含普通文本、数值、日期列且不要求单选枚举时，仍可直接创建并写入。

表格、多维表结构不确定时，先读 `create_file_with_content` reference。写入成功后优先展示返回里的 `data.link_url`。

**本地 Markdown 上传**：用户仅说「上传 md」且未要求保留 `.md` 时，走 `create_file_with_content`（`name` 不带后缀，默认 `file_extension=otl`；步骤见 `references/workflows/md-upload-to-otl.md`）；明确要求保留 `.md` 时走「本地文件上传为在线 Markdown」（`references/workflows/upload-md-raw.md`，含图片转 `kdocsmd://`）。对话内正文新建在线 Markdown → `references/workflows/create-cloud-md.md`。总规则见 `references/markdown.md`。

---

## 一、文档创建与上传

> 新建云文档、上传本地文件、网页剪藏与附件上传

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `drive.create_empty_file` | 新建空白在线文档 | `name`, `file_extension` |
| `drive.create_file_with_content` | 新建并写入内容 | `name`, `file_extension` |
| `drive.create_folder` | 在云盘下新建文件夹 | `name` |
| `drive.scrape_progress` | 查询网页剪藏任务进度 | `job_id` |
| `drive.scrape_url` | 网页剪藏，抓取网页内容并自动保存为智能文档 | `url` |
| `drive.upload_new_file` | 上传本地文件新建云文档 | `name`, `content_base64` |
| `drive.upload_replace_file` | 通过上传本地文件全量覆盖已有云文档 | `file_id`, `content_base64` |
| `drive.upload_attachment` | 向已有文档上传附件，支持 URL 或 Base64 | `file_id`, `filename`, `url`, `content_base64` |

## 二、文档读取与下载

> 列出目录、读取正文、获取下载信息与文件详情

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `drive.list_my_files` | 浏览我的云文档根目录 |  |
| `drive.find_folder_by_path` | 按路径查找已有文件夹 | `path` |
| `drive.list_files` | 获取指定文件夹下的子文件列表 | `parent_id` |
| `drive.download_file` | 获取文件下载信息 | `url`, `link_id`, `file_id` |
| `drive.download_attachment` | 获取文档附件的下载信息 | `url`, `link_id`, `file_id`, `attachment_id` |
| `drive.get_file_info` | 获取文件（夹）详细信息 | `url`, `link_id`, `file_id` |
| `drive.read_file` | 读取云文档正文为 Markdown/纯文本/结构化数据 | `url`, `link_id`, `file_id` |

## 三、文件组织

> 移动、重命名、复制以及同名检查

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `drive.move_file` | 批量移动文件(夹) | `file_ids`, `dst_parent_id` |
| `drive.rename_file` | 重命名文件（夹） | `file_id`, `dst_name` |
| `drive.copy_file` | 复制文件到指定目录（可跨盘） | `url`, `link_id`, `file_id`, `dst_parent_id` |
| `drive.check_file_name` | 检查目录下文件名是否已存在 | `name` |
| `drive.save_as_file` | 文件另存为到指定目录 | `url`, `link_id`, `file_id`, `dst_parent_id` |

## 四、分享

> 分享链接开启/权限/取消/查询，以及协作者查询与授权管理

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `drive.share_file` | 开启文件分享 | `url`, `link_id`, `file_id`, `scope` |
| `drive.set_share_permission` | 修改分享链接属性 | `url`, `link_id`, `file_id` |
| `drive.cancel_share` | 取消文件分享 | `url`, `link_id`, `file_id` |
| `drive.get_share_info` | 获取分享链接信息 | `url`, `link_id`, `file_id` |
| `drive.list_drive_roles` | 获取文档权限角色列表 | `drive_id` |
| `drive.list_document_collaborators` | 获取文档最近协作成员 | `url`, `link_id`, `file_id`, `limit` |
| `drive.search_document_collaborators` | 搜索文档协作者 | `url`, `link_id`, `file_id`, `key` |
| `drive.get_file_link` | 获取文件的云文档在线访问链接 | `url`, `link_id`, `file_id` |
| `drive.set_collaborator_permissions` | 批量设置协作者权限 | `url`, `link_id`, `file_id`, `role_id`, `subjects` |
| `drive.cancel_collaborator_permissions` | 批量取消协作者授权 | `url`, `link_id`, `file_id`, `subjects` |

## 五、搜索

> 跨云盘的文件搜索

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `drive.search_files` | 按关键词搜索云文档（文件/文件夹） | `page_size` |
| `drive.search_high_value_documents` | 根据你的描述，优先找出最相关、最近常用的云文档 | `query` |

## 六、标签

> 自定义标签的维护与对象打标

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `drive.list_labels` | 分页获取云盘自定义标签列表（可按归属者、标签类型筛选） | `page_size` |
| `drive.create_label` | 创建自定义标签 | `allotee_type`, `name` |
| `drive.get_label_meta` | 获取单个标签详情（含系统标签固定 ID） | `label_id` |
| `drive.get_label_objects` | 获取某标签下的对象列表（文件/云盘等） | `label_id`, `object_type`, `page_size` |
| `drive.batch_add_label_objects` | 批量为多个文档对象添加同一标签（打标签） | `label_id`, `objects` |
| `drive.batch_remove_label_objects` | 批量取消标签 | `label_id`, `objects` |
| `drive.batch_update_label_objects` | 批量更新标签下对象排序或属性 | `label_id`, `objects` |
| `drive.batch_update_labels` | 批量修改自定义标签名称或属性 | `labels` |

## 七、收藏

> 收藏列表与批量操作

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `drive.list_star_items` | 获取收藏（星标）列表 | `page_size` |
| `drive.batch_create_star_items` | 批量添加收藏 | `objects` |
| `drive.batch_delete_star_items` | 批量移除收藏 | `objects` |

## 八、最近与回收站

> 最近访问、回收站列表与还原

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `drive.list_latest_items` | 获取最近访问文档列表 | `page_size` |
| `drive.list_deleted_files` | 获取回收站文件列表 | `page_size` |
| `drive.restore_deleted_file` | 将回收站文件还原到原位置 | `file_id` |

## 九、评论

> 创建与查询评论

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `drive.list_document_comments` | 获取文档全文评论列表 | `url`, `link_id`, `file_id`, `origin_id` |
| `drive.create_document_comment` | 发表全文评论或回复 | `url`, `link_id`, `file_id`, `content` |
| `drive.get_file_inline_comments` | 获取文档正文划选批注列表 | `url`, `link_id`, `file_id` |

## 十、团队文档库

> 枚举团队文档库列表、获取单个文档库详情

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `drive.list_doclibs` | 获取团队文档库列表 |  |
| `drive.get_doclib_meta` | 获取单个团队文档库详情 | `drive_id` |

## 十一、文件历史版本

> 查询文件历史版本列表、获取指定版本下载信息

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `drive.list_file_versions` | 获取文件历史版本列表 | `url`, `link_id`, `file_id` |
| `drive.get_file_version_download` | 获取文件历史版本的下载信息 | `url`, `link_id`, `file_id`, `version_num` |

---

## 高频操作流程

| 流程 | 说明 | 详细参考 |
|------|------|----------|
| 网页剪藏 | 抓取网页内容并自动保存为智能文档 | `references/workflows/web-scrape.md` |
| 搜索-读取-汇报撰写 | 搜索多份文档、提取信息、汇总撰写新报告 | `references/workflows/search-read-report.md` |
| 定期读取与播报 | 定期读取指定文档，提取关键信息生成摘要 | `references/workflows/periodic-read-summary.md` |
| 文档内容转 Markdown | 将云文档内容提取为 Markdown 格式文本 | `references/workflows/read-to-markdown.md` |
| 智能分类整理 | 列出目录，按内容或指定维度分类创建文件夹并归档 | `references/workflows/smart-classify.md` |
| 读取在线 Markdown | 读取在线 Markdown 文件正文（Markdown 源码） | `references/workflows/read-cloud-md.md` |
| 整夹批量移动 | 将一个文件夹下的所有文件批量移动到另一个文件夹 | `references/workflows/batch-move.md` |
| 精准搜索与风险排查 | 在特定目录批量搜索文档，逐一读取分析，汇总到新文档 | `references/workflows/precise-search-analysis.md` |
| 在线 Markdown 导出到本地 | 将在线 Markdown 导出为本地文件；图片默认内联 HTTPS 临时链，也可保存到本地 assets 目录 | `references/workflows/download-cloud-md.md` |
| 回收站查看与恢复 | 查看回收站文件并恢复误删文件到原位置 | `references/workflows/recycle-restore.md` |
| 本地 Markdown 上传为在线文档（保留 .md） | 将本地 .md 上传为在线 Markdown，并把图片转存为 kdocsmd://（不转为 otl/docx） | `references/workflows/upload-md-raw.md` |
| 对话内正文新建在线 Markdown | 将对话中的 Markdown 正文新建为在线 Markdown，并把图片转存为 kdocsmd:// | `references/workflows/create-cloud-md.md` |
| 本地 Markdown 上传为办公文档 | 本地 .md 转为云盘办公文档（默认 otl；亦可 docx/pdf） | `references/workflows/md-upload-to-otl.md` |
| 云 Word 转智能文档 | 云盘 .docx/.doc 正文转为 .otl，清洗 Word 域代码后再写入 | `references/workflows/docx-to-otl.md` |
| 本地 Markdown 覆盖在线 Markdown | 用本地 .md 全量覆盖已有在线 Markdown，并把图片转存为 kdocsmd:// | `references/workflows/replace-cloud-md.md` |
| 在线 Markdown 总结与问答 | 读取在线 Markdown 正文后，总结要点或按用户问题作答 | `references/workflows/summarize-cloud-md.md` |
| 标签列表、打标与按标检索 | 列出标签、批量给文档打标，或按标签检索文件 | `references/workflows/label-tag-search.md` |
| 标签归类与检索 | 用自定义标签整理文件并按标签浏览 | `references/workflows/label-classify.md` |
| 批量提取发票信息 | 批量读取 PDF 发票，提取结构化字段并整合输出 | `references/workflows/invoice-extract.md` |

## 错误速查

| 错误特征 | 原因 | 处理方式 |
|----------|------|----------|
| 搜索无结果 | 关键词过精确 / 索引延迟 | 缩短关键词 / 等待 3-5 秒重试 |
| 读取内容为空 | 文件无内容或格式不支持 | 确认文件非空且后缀正确 |
| 不支持的文件类型 / `Unsupported file type .xxx` | 该格式不在当前工具支持范围内 | 立即停止：不要换 file_id/链接、不要改用其它读取或下载工具重试；直接告知用户该格式不支持 |
| 创建文件失败 | 文件名后缀不正确 | 检查后缀：`.otl` / `.docx` / `.xlsx` / `.ksheet` / `.dbt` / `.pdf` / `.pptx` |
| 移动文件失败 | 目标文件夹不存在 | 先搜索确认或创建文件夹 |

## 附录

### A. 通用文件信息结构（FileInfo）

`create_empty_file`、`upload_new_file`、`upload_replace_file`（步骤三）、`rename_file`、`list_files` 等接口返回的文件信息共用以下结构。响应字段表中 `array[FileInfo]` 即引用此结构：
| 字段 | 类型 | 说明 |
|------|------|------|
| `id` | string | 文件 ID |
| `name` | string | 文件名 |
| `type` | string | 文件类型：`file` / `folder` / `shortcut` |
| `size` | integer | 文件大小 |
| `parent_id` | string | 父目录 ID |
| `drive_id` | string | 驱动盘 ID |
| `version` | integer | 文件版本 |
| `ctime` | integer | 文件创建时间（时间戳，秒） |
| `mtime` | integer | 文件修改时间（时间戳，秒） |
| `shared` | boolean | 是否开启分享（link.status = 'open' 时为 true） |
| `link_id` | string | 分享 ID |
| `link_url` | string | 分享链接 URL |
| `created_by` | object | 文件创建者信息 |
| `created_by.id` | string | 身份 ID |
| `created_by.name` | string | 用户或应用的名称 |
| `created_by.type` | string | 身份类型：`user` / `sp` / `unknown` |
| `created_by.avatar` | string | 头像 |
| `created_by.company_id` | string | 身份所归属的公司 |
| `modified_by` | object | 文件修改者信息（结构同 created_by） |
| `ext_attrs` | array[object] | 文件扩展属性（`with_ext_attrs=true` 时返回），每项含 name 和 value |
| `drive` | object | 文件驱动盘信息（`with_drive=true` 时返回）

**`permission` 对象**（`with_permission=true` 时返回）：

| 字段 | 类型 | 说明 |
|------|------|------|
| `comment` | boolean | 评论 |
| `copy` | boolean | 复制 |
| `copy_content` | boolean | 内容复制 |
| `delete` | boolean | 文件删除 |
| `download` | boolean | 下载 |
| `history` | boolean | 历史版本（仅公网支持） |
| `list` | boolean | 列表 |
| `move` | boolean | 文件移动 |
| `new_empty` | boolean | 新建 |
| `perm_ctl` | boolean | 权限管理 |
| `preview` | boolean | 预览 |
| `print` | boolean | 打印 |
| `rename` | boolean | 文件重命名 |
| `saveas` | boolean | 另存为（仅公网支持） |
| `secret` | boolean | 安全文档（仅公网支持） |
| `share` | boolean | 分享 |
| `update` | boolean | 编辑/更新 |
| `upload` | boolean | 上传：手动上传新版本
