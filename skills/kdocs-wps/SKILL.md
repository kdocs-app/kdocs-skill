# 在线文字（wps）（工具路由索引）

本文件包含金山文档 Skill 中在线文字（`wps.*`）工具的操作说明。该类工具面向在线编辑中的文字文档，适合已有文档的内容编辑、导出与原子操作。

**适用范围**：支持在线 Word 文档的段落与字符范围读写、查找替换、字体与段落格式、标题样式、列表编号、表格、图片、形状、页眉页脚、分节版式、书签、超链接、批注、修订、目录、水印及文档导出。

**不适用范围**：文件搜索、通用创建上传、分享、移动及其他组件操作。遇到上述域外需求、需要借助其他组件定位文件或准备外部资源，或无法判断请求应由哪个组件处理时，[返回主 Skill](../../SKILL.md)安排，不自行切换子 Skill。

---

## 通用说明

### 在线文字特点

- 面向在线文字文档，不是本地 `.docx` 文件直传接口
- 支持将已有在线文字文档导出为 DOCX、PDF、图片或 AP
- 提供结构化原子工具，对文档进行段落/区间级别的增删改查和格式设置等操作
- 若只是读取正文内容，按[主 Skill「操作域路由」中的「读取」项](../../SKILL.md#操作域路由)处理。

### 何时使用 `wps.*`
- 需要编辑已有在线文字文档的内容或格式
- 需要把在线文字导出为 DOCX、PDF、图片或 AP 文稿
- 需要对文档执行原子操作：读取/修改指定段落内容、查找替换、设置段落格式、设置字符格式等

### `wps.*` 工具调用说明

- 格式：服务名和工具分开，property 级命令保留点号
  例如：`kdocs-cli wps texts.range`、`kdocs-cli wps export`
- 文档定位：除创建空白文档 / 纯任务查询外，统一传 `url` / `link_id` / `file_id` **三选一**

#### 调用形态

```
kdocs-cli help wps.texts.format
kdocs-cli wps texts.format '{"file_id":"...","scope":"ranges"}'
kdocs-cli call wps.texts.format '{"file_id":"...","scope":"ranges"}'
```

## 导出能力总览

`wps.*` 中的导出能力对外拆分为三个工具：

- `wps.export`：导出 DOCX、创建 PDF 导出任务、发起 AP 导出流程
- `wps.export_image`：导出 PNG / JPEG 图片
- `wps.query_export`：统一查询异步导出结果

### 选择建议

- 需要拿到 `.docx` 下载地址：用 `wps.export`，传 `format=docx`
- 需要导出图片：用 `wps.export_image`，传 `format=png/jpeg`，定位 `url` / `link_id` / `file_id` 三选一
- 需要导出 PDF：先 `wps.export`，传 `format=pdf`；再按需用 `wps.query_export`
- 需要导出 AP：先 `wps.export`，传 `format=ap`；再用 `wps.query_export`（`format=ap` 时须传 export 返回的 `file_id`）

## 结构化文档工具

## 一、导出

> 异步导出为多种格式及状态轮询

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `wps.export` | 统一导出在线文字文档 | `url`, `link_id`, `file_id`, `format` |
| `wps.export_image` | 将在线文字导出为图片 | `url`, `link_id`, `file_id`, `format` |
| `wps.query_export` | 统一查询异步导出结果 | `format`, `task_id` |

## 二、书签操作

> 书签域相关操作。

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `wps.bookmarks.content` | 在线文字文档书签内容：支持查询/设置 | `file_id`, `link_id`, `url`, `verb` |
| `wps.bookmarks.count` | 查询书签的数量 | `file_id`, `link_id`, `url` |
| `wps.bookmarks.delete` | 删除书签 | `bookmark_name`, `file_id`, `link_id`, `url` |
| `wps.bookmarks.exists` | 查询书签的是否存在 | `file_id`, `link_id`, `url` |
| `wps.bookmarks.goto` | 跳转到书签 | `bookmark_name`, `file_id`, `link_id`, `url` |
| `wps.bookmarks.insert` | 在段落或区间插入书签 | `file_id`, `link_id`, `url`, `scope` |
| `wps.bookmarks.list` | 查询书签的列表 | `file_id`, `link_id`, `url` |
| `wps.bookmarks.rename` | 重命名书签 | `bookmark_name`, `file_id`, `link_id`, `new_bookmark_name`, `url` |

## 三、分隔符操作

> 分隔符域相关操作。

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `wps.breaks.data` | 在在线文字文档段落或区间插入分隔符 | `file_id`, `link_id`, `url`, `scope` |

## 四、题注操作

> 题注域相关操作。

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `wps.captions.data` | 插入题注 | `file_id`, `link_id`, `url` |
| `wps.captions.label` | 新增自定义题注标签 | `file_id`, `label`, `link_id`, `url` |

## 五、批注操作

> 批注域相关操作。

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `wps.comments.add` | 在在线文字文档中插入批注 | `file_id`, `link_id`, `text`, `url` |
| `wps.comments.author` | 按作者查询批注 | `file_id`, `link_id`, `url` |
| `wps.comments.count` | 查询批注数量 | `file_id`, `link_id`, `url` |
| `wps.comments.data` | 查询批注数据 | `file_id`, `link_id`, `url` |
| `wps.comments.delete` | 删除在线文字文档指定批注 | `file_id`, `link_id`, `url` |
| `wps.comments.delete_all` | 删除在线文字文档全部批注 | `file_id`, `link_id`, `url` |
| `wps.comments.list` | 查询批注列表 | `file_id`, `link_id`, `url` |
| `wps.comments.modify_text` | 修改在线文字文档批注文本 | `file_id`, `link_id`, `text`, `url` |
| `wps.comments.paragraph_insert` | 在在线文字文档指定段落插入批注 | `file_id`, `link_id`, `paragraph_index`, `text`, `url` |
| `wps.comments.range_insert` | 在在线文字文档字符区间插入批注 | `begin`, `end`, `file_id`, `link_id`, `text`, `url` |
| `wps.comments.reply` | 回复在线文字文档批注 | `file_id`, `link_id`, `text`, `url` |
| `wps.comments.resolve` | 解决在线文字文档批注 | `file_id`, `link_id`, `url` |
| `wps.comments.set_done` | 设置在线文字文档批注完成状态 | `file_id`, `link_id`, `url` |

- 批注索引（index）为 1-based，须先存在；可用 wps.comments.list 取真实 index 与选区（scope_begin/scope_end）。
- 新增批注定位：段落用 paragraph_index（1-based），字符区间用 begin/end（0-based，可由 wps.texts.search 返回的 ranges 回填）。

## 六、内容控件操作

> 内容控件域相关操作。

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `wps.content_controls.all` | 删除全部内容控件 | `file_id`, `link_id`, `url` |
| `wps.content_controls.checkbox` | 插入内容控件的复选框 | `file_id`, `link_id`, `url` |
| `wps.content_controls.count` | 查询内容控件数量 | `file_id`, `link_id`, `url` |
| `wps.content_controls.data` | 在线文字文档内容控件数据：支持 删除/查询 | `file_id`, `link_id`, `url`, `verb` |
| `wps.content_controls.date_picker` | 插入内容控件的日期选取器 | `file_id`, `link_id`, `url` |
| `wps.content_controls.dropdown` | 插入内容控件的下拉列表 | `file_id`, `link_id`, `url` |
| `wps.content_controls.dropdown_add` | 设置内容控件的下拉项 | `file_id`, `link_id`, `url` |
| `wps.content_controls.dropdown_remove` | 设置内容控件的下拉项 | `file_id`, `link_id`, `url` |
| `wps.content_controls.list` | 查询内容控件列表 | `file_id`, `link_id`, `url` |
| `wps.content_controls.plain_text` | 插入内容控件的纯文本 | `file_id`, `link_id`, `url` |
| `wps.content_controls.props` | 设置内容控件属性 | `file_id`, `link_id`, `url` |
| `wps.content_controls.rich_text` | 插入内容控件的富文本 | `file_id`, `link_id`, `url` |
| `wps.content_controls.tag` | 在线文字文档内容控件的标签查询：支持 删除/查询 | `file_id`, `link_id`, `url`, `verb` |
| `wps.content_controls.title` | 查询内容控件标题 | `file_id`, `link_id`, `url` |
| `wps.content_controls.value` | 设置内容控件值 | `file_id`, `link_id`, `url` |
| `wps.content_controls.value_by_tag` | 设置内容控件的标签设置值 | `file_id`, `link_id`, `url` |

## 七、域操作

> 文档域相关操作。

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `wps.fields.all` | 设置域的全部域 | `file_id`, `link_id`, `url` |
| `wps.fields.by_paragraph` | 按段落插入域 | `file_id`, `link_id`, `url` |
| `wps.fields.by_range` | 按区间插入域 | `file_id`, `link_id`, `url` |
| `wps.fields.count` | 查询域的域数量 | `file_id`, `link_id`, `url` |
| `wps.fields.data` | 在线文字文档域的域数据：支持 删除/查询/设置 | `file_id`, `link_id`, `url`, `verb` |
| `wps.fields.list` | 查询域的域列表 | `file_id`, `link_id`, `url` |
| `wps.fields.toggle_code` | 设置域的域代码 | `file_id`, `link_id`, `url` |
| `wps.fields.unlink` | 设置域的域链接 | `file_id`, `link_id`, `url` |

## 八、脚注尾注操作

> 脚注尾注域相关操作。

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `wps.footnote_endnotes.endnote_all` | 删除全部尾注 | `file_id`, `link_id`, `url` |
| `wps.footnote_endnotes.endnote_by_paragraph` | 按段落插入尾注 | `file_id`, `link_id`, `url` |
| `wps.footnote_endnotes.endnote_by_range` | 按区间插入尾注 | `file_id`, `link_id`, `url` |
| `wps.footnote_endnotes.endnote_count` | 查询尾注数量 | `file_id`, `link_id`, `url` |
| `wps.footnote_endnotes.endnote_data` | 在线文字文档尾注数据：支持 删除/查询 | `file_id`, `link_id`, `url`, `verb` |
| `wps.footnote_endnotes.endnote_list` | 查询尾注列表 | `file_id`, `link_id`, `url` |
| `wps.footnote_endnotes.endnote_text` | 设置尾注文本 | `file_id`, `link_id`, `url` |
| `wps.footnote_endnotes.footnote_all` | 删除全部脚注 | `file_id`, `link_id`, `url` |
| `wps.footnote_endnotes.footnote_by_paragraph` | 按段落插入脚注 | `file_id`, `link_id`, `url` |
| `wps.footnote_endnotes.footnote_by_range` | 按区间插入脚注 | `file_id`, `link_id`, `url` |
| `wps.footnote_endnotes.footnote_count` | 查询脚注数量 | `file_id`, `link_id`, `url` |
| `wps.footnote_endnotes.footnote_data` | 在线文字文档脚注数据：支持 删除/查询 | `file_id`, `link_id`, `url`, `verb` |
| `wps.footnote_endnotes.footnote_font` | 设置脚注/尾注的字体 | `file_id`, `link_id`, `url` |
| `wps.footnote_endnotes.footnote_list` | 查询脚注列表 | `file_id`, `link_id`, `url` |
| `wps.footnote_endnotes.footnote_text` | 设置脚注文本 | `file_id`, `link_id`, `url` |

## 九、公式操作

> 公式域相关操作。

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `wps.formulas.math` | 插入/查询在线文字文档公式 | `file_id`, `link_id`, `url`, `verb` |

## 十、页眉页脚操作

> 页眉页脚域相关操作。

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `wps.header_footers.content` | 设置页眉页脚内容 | `file_id`, `link_id`, `url` |
| `wps.header_footers.different_first` | 设置页眉页脚首页不同 | `file_id`, `link_id`, `url` |
| `wps.header_footers.different_odd_even` | 设置页眉页脚奇偶页不同 | `file_id`, `link_id`, `url` |
| `wps.header_footers.footer_alignment` | 在线文字文档页脚的对齐（查询/设置） | `file_id`, `link_id`, `url`, `verb` |
| `wps.header_footers.footer_content` | 在线文字文档页脚的内容：支持 查询/设置 | `file_id`, `link_id`, `url`, `verb` |
| `wps.header_footers.footer_font` | 设置页脚字体 | `file_id`, `link_id`, `url` |
| `wps.header_footers.footer_link_previous` | 设置页脚链接上一节 | `file_id`, `link_id`, `url` |
| `wps.header_footers.footer_page_number` | 设置页脚页码 | `file_id`, `link_id`, `url` |
| `wps.header_footers.footer_page_number_custom` | 在线文字文档页脚的自定义文字包裹页码（查询/设置） | `file_id`, `link_id`, `url`, `verb` |
| `wps.header_footers.footer_remove` | 设置页脚 | `file_id`, `link_id`, `url` |
| `wps.header_footers.header_alignment` | 在线文字文档页眉的对齐（查询/设置） | `file_id`, `link_id`, `url`, `verb` |
| `wps.header_footers.header_content` | 在线文字文档页眉的内容：支持 查询/设置 | `file_id`, `link_id`, `url`, `verb` |
| `wps.header_footers.header_font` | 设置页眉字体 | `file_id`, `link_id`, `url` |
| `wps.header_footers.header_footer_full` | 设置页眉页脚内容 | `file_id`, `link_id`, `url` |
| `wps.header_footers.header_link_previous` | 设置页眉链接上一节 | `file_id`, `link_id`, `url` |
| `wps.header_footers.header_page_number` | 设置页眉页码 | `file_id`, `link_id`, `url` |
| `wps.header_footers.header_page_number_custom` | 在线文字文档页眉的自定义文字包裹页码（查询/设置） | `file_id`, `link_id`, `url`, `verb` |
| `wps.header_footers.header_remove` | 设置页眉 | `file_id`, `link_id`, `url` |
| `wps.header_footers.page_number_style` | 设置页码样式 | `file_id`, `link_id`, `url` |
| `wps.header_footers.shape_alignment` | 设置页眉页脚形状对齐 | `file_id`, `link_id`, `url` |

## 十一、标题操作

> 标题域相关操作。

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `wps.headings.heading_table` | 按标题章节插入表格 | `file_id`, `link_id`, `url`, `heading_text`, `rows`, `cols`, `table_full_data` |
| `wps.headings.heading_delete` | 按标题删除整节 | `file_id`, `heading_text`, `link_id`, `url` |
| `wps.headings.heading_meta` | 按标题定位章节 | `file_id`, `heading_text`, `link_id`, `url` |
| `wps.headings.heading_text` | 按标题章节插入文本 | `file_id`, `heading_text`, `link_id`, `text`, `url` |
| `wps.headings.heading_text_batch` | 按标题章节批量插入文本 | `file_id`, `items`, `link_id`, `url` |

## 十二、超链接操作

> 超链接域相关操作。

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `wps.hyperlinks.address` | 更新超链接地址 | `file_id`, `index`, `link_id`, `url`, `value` |
| `wps.hyperlinks.all` | 删除全部超链接 | `file_id`, `link_id`, `url` |
| `wps.hyperlinks.count` | 查询超链接数量 | `file_id`, `link_id`, `url` |
| `wps.hyperlinks.data` | 在线文字文档超链接数据：支持删除/查询 | `file_id`, `link_id`, `url`, `verb` |
| `wps.hyperlinks.insert` | 在段落或区间插入超链接 | `file_id`, `link_id`, `url`, `scope` |
| `wps.hyperlinks.list` | 查询超链接列表 | `file_id`, `link_id`, `url` |

## 十三、图片操作

> 图片域相关操作。

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `wps.images.all` | 删除全部图片 | `file_id`, `link_id`, `url` |
| `wps.images.alternative_text` | 图片替代文字 | `file_id`, `link_id`, `url`, `verb` |
| `wps.images.border` | 设置图片的边框 | `file_id`, `index`, `link_id`, `url` |
| `wps.images.count` | 查询图片数量 | `file_id`, `link_id`, `url` |
| `wps.images.crop` | 设置图片的裁剪 | `file_id`, `index`, `link_id`, `url` |
| `wps.images.crop_to_shape` | 设置图片的按形状裁剪 | `file_id`, `index`, `link_id`, `url` |
| `wps.images.data` | 删除图片数据 | `file_id`, `link_id`, `url` |
| `wps.images.info` | 查询图片信息 | `file_id`, `link_id`, `url` |
| `wps.images.insert` | 在文档、段落或区间插入图片（file_path 为图片 URL） | `file_id`, `link_id`, `url`, `scope` |
| `wps.images.list` | 查询图片列表 | `file_id`, `link_id`, `url` |
| `wps.images.locate_paragraph` | 查询图片所在段落 | `file_id`, `link_id`, `url` |
| `wps.images.lock_aspect_ratio` | 锁定图片纵横比 | `file_id`, `index`, `link_id`, `lock_aspect_ratio`, `url` |
| `wps.images.move_to_paragraph` | 将图片移动到段落 | `file_id`, `index`, `link_id`, `url` |
| `wps.images.resize` | 设置图片尺寸 | `file_id`, `index`, `link_id`, `url` |
| `wps.images.scale` | 图片缩放百分比 | `file_id`, `index`, `link_id`, `scale_height`, `scale_width`, `url` |
| `wps.images.title` | 设置图片标题 | `file_id`, `index`, `link_id`, `title`, `url` |
| `wps.images.wrap` | 设置图片的环绕方式 | `file_id`, `index`, `link_id`, `url` |

## 十四、索引操作

> 索引域相关操作。

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `wps.indexes.entry` | 插入索引项 | `begin`, `end`, `file_id`, `index_entry`, `link_id`, `url` |

## 十五、列表操作

> 列表域相关操作。

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `wps.lists.apply_with_level` | 批量按级别为多个段落应用列表模板 | `file_id`, `items`, `link_id`, `url` |
| `wps.lists.blocks` | 枚举文档内列表块 | `file_id`, `link_id`, `url` |
| `wps.lists.custom_bullet` | 设置自定义项目符号 | `bullet_font_name`, `bullet_symbol`, `file_id`, `link_id`, `paragraph_index`, `url` |
| `wps.lists.data` | 在线文字文档段落或范围列表属性：支持 查询/设置 | `file_id`, `link_id`, `url`, `scope`, `verb` |
| `wps.lists.level_style` | 按级别定向设置列表编号样式 | `file_id`, `link_id`, `paragraph_index`, `url` |
| `wps.lists.multilevel` | 对字符区间应用大纲多级列表模板 | `begin`, `end`, `file_id`, `link_id`, `url` |
| `wps.lists.number_format` | 为段落设置自定义编号格式 | `file_id`, `link_id`, `number_format`, `number_style`, `paragraph_index`, `url` |
| `wps.lists.remove` | 移除列表格式 | `begin`, `end`, `file_id`, `link_id`, `url` |
| `wps.lists.restart` | 段落列表重新编号 | `file_id`, `link_id`, `paragraph_index`, `url` |

## 十六、OLE 对象操作

> OLE 对象域相关操作。

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `wps.ole.list` | 列出内嵌 OLE 对象 | `file_id`, `link_id`, `url` |
| `wps.ole.text_after` | 在内嵌 OLE 对象后插入文本 | `file_id`, `index`, `link_id`, `text`, `url` |

## 十七、文档保护操作

> 文档保护域相关操作。

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `wps.protection.disable` | 取消编辑限制 | `file_id`, `link_id`, `url` |
| `wps.protection.enable` | 限制编辑 | `file_id`, `link_id`, `protect_type`, `url` |
| `wps.protection.type` | 查询文档保护类型 | `file_id`, `link_id`, `url` |

## 十八、交叉引用操作

> 交叉引用域相关操作。

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `wps.references.insert` | 在指定字符区间插入书签交叉引用（仅支持 reference_type=2） | `begin`, `end`, `file_id`, `link_id`, `reference_type`, `url` |

## 十九、修订操作

> 修订域相关操作。

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `wps.revisions.accept` | 接受指定修订 | `file_id`, `link_id`, `url` |
| `wps.revisions.accept_all` | 在线文字文档接受全部修订：支持 查询/设置 | `file_id`, `link_id`, `url`, `verb` |
| `wps.revisions.accept_by_author` | 在线文字文档按作者接受修订：支持 查询/设置 | `file_id`, `link_id`, `url`, `verb` |
| `wps.revisions.all` | 查询全部修订 | `file_id`, `link_id`, `url` |
| `wps.revisions.by_author` | 按作者查询修订 | `file_id`, `link_id`, `url` |
| `wps.revisions.reject` | 拒绝指定修订 | `file_id`, `link_id`, `url` |
| `wps.revisions.reject_all` | 在线文字文档拒绝全部修订：支持 查询/设置 | `file_id`, `link_id`, `url`, `verb` |
| `wps.revisions.reject_by_author` | 在线文字文档按作者拒绝修订：支持 查询/设置 | `file_id`, `link_id`, `url`, `verb` |
| `wps.revisions.status` | 设置在线文字文档的修订模式状态 | `file_id`, `link_id`, `url`, `verb` |

## 二十、节操作

> 节域相关操作。

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `wps.sections.background_color` | 页面背景色查询/设置 | `file_id`, `link_id`, `url`, `verb` |
| `wps.sections.border` | 删除节的页面边框 | `file_id`, `link_id`, `section_index`, `url` |
| `wps.sections.border_art` | 设置节的页面边框艺术型 | `file_id`, `link_id`, `url` |
| `wps.sections.border_data` | 查询节的边框数据 | `file_id`, `link_id`, `url` |
| `wps.sections.border_distance` | 设置节的页面边框距正文距离 | `file_id`, `link_id`, `url` |
| `wps.sections.break` | 删除节的分节符 | `file_id`, `link_id`, `url` |
| `wps.sections.columns` | 删除节的分栏 | `file_id`, `link_id`, `section_index`, `url` |
| `wps.sections.count` | 查询节的节数量 | `file_id`, `link_id`, `url` |
| `wps.sections.grid_layout` | 设置节的网格排版 | `file_id`, `link_id`, `url` |
| `wps.sections.gutter` | 设置节的装订线 | `file_id`, `link_id`, `url` |
| `wps.sections.line_numbering` | 设置节的行号 | `file_id`, `link_id`, `url` |
| `wps.sections.page_setup` | 在线文字文档节的节页面设置：支持 查询/设置 | `file_id`, `link_id`, `url`, `verb` |
| `wps.sections.section_full` | 设置节的节页面设置 | `file_id`, `link_id`, `url`, `verb` |

## 二十一、形状操作

> 形状域相关操作。

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `wps.shapes.align` | 设置形状的对齐 | `file_id`, `link_id`, `url` |
| `wps.shapes.all` | 删除全部形状 | `file_id`, `link_id`, `url` |
| `wps.shapes.anchor` | 查询形状锚点段落 | `file_id`, `link_id`, `url` |
| `wps.shapes.basic` | 插入形状的基本形状 | `file_id`, `link_id`, `url` |
| `wps.shapes.count` | 查询形状数量 | `file_id`, `link_id`, `url` |
| `wps.shapes.data` | 删除形状数据 | `file_id`, `link_id`, `url` |
| `wps.shapes.delete_by_type` | 按类型删除形状 | `file_id`, `link_id`, `url` |
| `wps.shapes.distribute` | 设置形状的分布 | `file_id`, `link_id`, `url` |
| `wps.shapes.fill_color` | 在线文字文档形状填充颜色：支持 查询/设置 | `file_id`, `link_id`, `url`, `verb` |
| `wps.shapes.find_by_text` | 按文本查找形状 | `file_id`, `link_id`, `url` |
| `wps.shapes.font` | 设置在线文字文档形状内字体样式 | `file_id`, `link_id`, `url`, `verb` |
| `wps.shapes.group` | 组合形状 | `file_id`, `link_id`, `url` |
| `wps.shapes.info` | 查询形状信息 | `file_id`, `link_id`, `url` |
| `wps.shapes.line` | 插入形状的线条 | `file_id`, `link_id`, `url` |
| `wps.shapes.line_color` | 在线文字文档形状线条颜色：支持 查询/设置 | `file_id`, `link_id`, `url`, `verb` |
| `wps.shapes.line_dash` | 在线文字文档形状虚线样式：支持 查询/设置 | `file_id`, `link_id`, `url`, `verb` |
| `wps.shapes.line_width` | 在线文字文档形状线条宽度：支持 查询/设置 | `file_id`, `link_id`, `url`, `verb` |
| `wps.shapes.list` | 查询形状列表 | `file_id`, `link_id`, `url` |
| `wps.shapes.paragraph_font` | 设置形状内段落字体样式 | `file_id`, `link_id`, `url` |
| `wps.shapes.picture` | 插入形状的图片形状 | `file_id`, `link_id`, `url` |
| `wps.shapes.props` | 设置形状属性 | `file_id`, `link_id`, `url` |
| `wps.shapes.subrange_font` | 设置形状的文本子区间字体 | `file_id`, `link_id`, `url` |
| `wps.shapes.text` | 在线文字文档形状文本：支持 查询/设置 | `file_id`, `link_id`, `url`, `verb` |
| `wps.shapes.text_box` | 插入形状的文本框 | `file_id`, `link_id`, `url` |
| `wps.shapes.text_box_auto_size` | 设置形状的文本框自动调整大小 | `file_id`, `link_id`, `url` |
| `wps.shapes.text_box_link` | 设置文本框链接 | `file_id`, `link_id`, `url` |
| `wps.shapes.text_box_margins` | 设置形状的文本框内边距 | `file_id`, `link_id`, `url` |
| `wps.shapes.text_replace` | 查找并替换形状内文本 | `file_id`, `link_id`, `url` |
| `wps.shapes.ungroup` | 取消组合形状 | `file_id`, `link_id`, `url` |
| `wps.shapes.wrap_type` | 设置/查询形状环绕方式 | `file_id`, `link_id`, `url`, `verb` |
| `wps.shapes.z_order` | 设置/查询形状叠放次序 | `file_id`, `link_id`, `url`, `verb` |

## 二十二、样式操作

> 样式域相关操作。

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `wps.styles.data` | 为段落或区间套用已有样式 | `file_id`, `link_id`, `url`, `scope` |
| `wps.styles.definition` | 插入、更新或删除在线文字文档样式定义 | `file_id`, `link_id`, `url`, `verb` |
| `wps.styles.list` | 查询样式列表 | `file_id`, `link_id`, `url` |

## 二十三、表格操作

> 表格域相关操作。

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `wps.tables.alignment` | 设置在线文字文档表格的对齐 | `file_id`, `link_id`, `url`, `verb` |
| `wps.tables.all` | 删除全部表格 | `file_id`, `link_id`, `url` |
| `wps.tables.auto_fit` | 设置在线文字文档表格的自动调整 | `file_id`, `link_id`, `url`, `verb` |
| `wps.tables.borders` | 设置在线文字文档表格的边框 | `file_id`, `link_id`, `url`, `verb` |
| `wps.tables.cell_alignment` | 设置在线文字文档表格的单元格对齐 | `file_id`, `link_id`, `url`, `verb` |
| `wps.tables.cell_background` | 设置在线文字文档表格的单元格背景 | `file_id`, `link_id`, `url`, `verb` |
| `wps.tables.cell_border` | 设置在线文字文档表格的单元格边框 | `file_id`, `link_id`, `url`, `verb` |
| `wps.tables.cell_content` | 在线文字文档表格的单元格内容：支持 删除/查询/设置 | `file_id`, `link_id`, `url`, `verb` |
| `wps.tables.cell_content_append` | 设置表格的单元格追加内容 | `file_id`, `link_id`, `table_index`, `url` |
| `wps.tables.cell_font` | 在线文字文档表格的单元格字体：支持 查询/设置 | `file_id`, `link_id`, `url`, `verb` |
| `wps.tables.cell_margins` | 设置/查询表格单元格边距 | `file_id`, `link_id`, `url`, `verb` |
| `wps.tables.cell_merge` | 设置表格的合并单元格 | `file_id`, `link_id`, `table_index`, `url` |
| `wps.tables.cell_split` | 设置表格的拆分单元格 | `file_id`, `link_id`, `table_index`, `url` |
| `wps.tables.cell_split_to_cols` | 设置表格的单元格拆分为多列 | `file_id`, `link_id`, `table_index`, `url` |
| `wps.tables.cell_split_to_rows` | 设置表格的单元格拆分为多行 | `file_id`, `link_id`, `table_index`, `url` |
| `wps.tables.cell_vertical_alignment` | 设置在线文字文档表格的单元格垂直对齐 | `file_id`, `link_id`, `url`, `verb` |
| `wps.tables.column` | 删除表格列 | `file_id`, `link_id`, `url` |
| `wps.tables.column_alignment` | 设置在线文字文档表格的列对齐 | `file_id`, `link_id`, `url`, `verb` |
| `wps.tables.column_background` | 设置在线文字文档表格的列背景 | `file_id`, `link_id`, `url`, `verb` |
| `wps.tables.column_content` | 查询表格的列内容 | `file_id`, `link_id`, `url` |
| `wps.tables.column_font` | 在线文字文档表格的列字体：支持 查询/设置 | `file_id`, `link_id`, `url`, `verb` |
| `wps.tables.column_insert` | 插入列 | `file_id`, `link_id`, `table_index`, `url` |
| `wps.tables.column_merge` | 设置表格的合并列 | `file_id`, `link_id`, `table_index`, `url` |
| `wps.tables.column_vertical_alignment` | 设置在线文字文档表格的列垂直对齐 | `file_id`, `link_id`, `url`, `verb` |
| `wps.tables.column_width` | 设置表格的列宽 | `file_id`, `link_id`, `table_index`, `url` |
| `wps.tables.convert_from_text` | 区间文本转表格 | `begin`, `cols`, `end`, `file_id`, `link_id`, `url` |
| `wps.tables.convert_to_text` | 设置表格转文本 | `file_id`, `link_id`, `table_index`, `url` |
| `wps.tables.count` | 查询表格数量 | `file_id`, `link_id`, `url` |
| `wps.tables.create` | 在文档末尾新建空表格 | `file_id`, `link_id`, `url`, `rows`, `cols` |
| `wps.tables.data` | 删除表格数据 | `file_id`, `link_id`, `url` |
| `wps.tables.dimensions` | 查询表格行列尺寸 | `file_id`, `link_id`, `url` |
| `wps.tables.range` | 查询表格范围 | `file_id`, `link_id`, `url` |
| `wps.tables.repeat_header_row` | 设置在线文字文档表格的重复标题行 | `file_id`, `link_id`, `url`, `verb` |
| `wps.tables.row` | 删除表格行 | `file_id`, `link_id`, `url` |
| `wps.tables.row_alignment` | 设置在线文字文档表格的行对齐 | `file_id`, `link_id`, `url`, `verb` |
| `wps.tables.row_background` | 设置在线文字文档表格的行背景 | `file_id`, `link_id`, `url`, `verb` |
| `wps.tables.row_break_across_pages` | 在线文字文档表格的跨页断行 | `file_id`, `link_id`, `url`, `verb` |
| `wps.tables.row_content` | 查询表格的行内容 | `file_id`, `link_id`, `url` |
| `wps.tables.row_content_batch` | 设置表格的批量设置行内容 | `file_id`, `link_id`, `table_index`, `url` |
| `wps.tables.row_font` | 在线文字文档表格的行字体：支持 查询/设置 | `file_id`, `link_id`, `url`, `verb` |
| `wps.tables.row_height` | 设置表格的行高 | `file_id`, `link_id`, `table_index`, `url` |
| `wps.tables.row_height_rule` | 设置在线文字文档表格的行高规则 | `file_id`, `link_id`, `url`, `verb` |
| `wps.tables.row_insert` | 插入行 | `file_id`, `link_id`, `table_index`, `url` |
| `wps.tables.row_merge` | 设置表格的合并行 | `file_id`, `link_id`, `table_index`, `url` |
| `wps.tables.row_vertical_alignment` | 设置在线文字文档表格的行垂直对齐 | `file_id`, `link_id`, `url`, `verb` |
| `wps.tables.sort` | 设置表格排序 | `file_id`, `link_id`, `table_index`, `url` |
| `wps.tables.split` | 设置表格的拆分表格 | `file_id`, `link_id`, `table_index`, `url` |
| `wps.tables.table_full` | 一次插入完整表格 | `file_id`, `link_id`, `url`, `cols`, `rows`, `full_data` |

- tables 域无独立插入表格工具；创建表格走 wps.write_table（action: insert / paragraph_insert / range_insert）
- 多数 tables.* 工具需先有表格（wps.tables.count ≥ 1），否则报 table not found

## 二十四、文字文档文本操作

> WPS 文字文档中与段落、字符区间相关的文本格式与内容操作。

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `wps.texts.alignment` | 设置在线文字文档文本的段落对齐 | `file_id`, `link_id`, `url`, `scope`, `verb` |
| `wps.texts.border` | 在线文字文档文本的段落边框：支持 插入/设置/查询 | `file_id`, `link_id`, `url`, `verb` |
| `wps.texts.builtin_properties` | 在线文字文档文本的内置文档属性（标题、作者等）：支持 查询/设置 | `file_id`, `link_id`, `url`, `verb` |
| `wps.texts.char_border` | 设置文本的字符边框 | `file_id`, `link_id`, `url` |
| `wps.texts.char_case` | 设置文本的更改大小写 | `file_id`, `link_id`, `url` |
| `wps.texts.char_spacing` | 设置文本的字符间距与缩放 | `file_id`, `link_id`, `url` |
| `wps.texts.clear_format` | 设置在线文字文档文本的清除段落格式 | `file_id`, `link_id`, `url`, `scope` |
| `wps.texts.content` | 在线文字文档文本内容：支持查询/插入/设置/删除 | `file_id`, `link_id`, `url`, `scope`, `verb` |
| `wps.texts.count` | 查询段落数量 | `file_id`, `link_id`, `url` |
| `wps.texts.effect` | 设置文本的文字效果（阴影/隐藏等） | `file_id`, `link_id`, `url` |
| `wps.texts.emphasis_mark` | 在线文字文档文本的着重符号：支持 设置/查询 | `file_id`, `link_id`, `url`, `verb` |
| `wps.texts.enclosed_char` | 设置文本的带圈字符 | `file_id`, `link_id`, `url` |
| `wps.texts.font` | 在线文字文档文本的段落字体：支持 查询/设置 | `file_id`, `link_id`, `url`, `scope`, `verb` |
| `wps.texts.font_batch` | 设置在线文字文档文本的批量修改段落字体 | `file_id`, `link_id`, `url`, `scope` |
| `wps.texts.font_color` | 设置在线文字文档文本的段落字体颜色 | `file_id`, `link_id`, `url`, `scope` |
| `wps.texts.font_gradient` | 按字符区间设置文字双色渐变填充 | `begin`, `end`, `file_id`, `link_id`, `fore_color`, `back_color`, `url` |
| `wps.texts.format` | 在线文字文档文本的段落格式：支持 查询/设置 | `file_id`, `link_id`, `url`, `scope`, `verb` |
| `wps.texts.format_batch` | 设置在线文字文档文本的批量修改段落格式 | `file_id`, `link_id`, `url`, `scope` |
| `wps.texts.format_copy` | 将源段落格式复制到目标段落范围（格式刷） | `file_id`, `link_id`, `source_paragraph`, `target_end`, `target_start`, `url` |
| `wps.texts.heading` | 在线文字文档文本的标题段落：支持插入/设置 | `file_id`, `link_id`, `url`, `scope`, `verb` |
| `wps.texts.highlight` | 设置在线文字文档文本的段落高亮 | `file_id`, `link_id`, `url`, `scope` |
| `wps.texts.indent` | 设置在线文字文档文本的段落缩进 | `file_id`, `link_id`, `url`, `scope` |
| `wps.texts.info` | 查询文本的文档信息 | `file_id`, `link_id`, `url` |
| `wps.texts.line_spacing` | 设置在线文字文档文本的段落行距 | `file_id`, `link_id`, `url`, `scope` |
| `wps.texts.page_count` | 查询文本的页数 | `file_id`, `link_id`, `url` |
| `wps.texts.page_number` | 查询段落页码 | `file_id`, `link_id`, `paragraph_index`, `url` |
| `wps.texts.range` | 查询段落文本范围 | `file_id`, `link_id`, `paragraph_index`, `url` |
| `wps.texts.replace` | 在在线文字文档中查找并替换文本 | `file_id`, `link_id`, `url`, `scope` |
| `wps.texts.search` | 搜索文本 | `file_id`, `find_text`, `link_id`, `url` |
| `wps.texts.shading` | 在线文字文档文本的段落底纹：支持 插入/设置/查询 | `file_id`, `link_id`, `url`, `verb` |
| `wps.texts.structure` | 批量查询段落结构 | `file_id`, `link_id`, `paragraph_indexes`, `url` |
| `wps.texts.style` | 查询段落样式 | `file_id`, `link_id`, `paragraph_index`, `url` |
| `wps.texts.tab_stops` | 在线文字文档文本的制表位：支持 插入/设置 | `file_id`, `link_id`, `url`, `verb` |
| `wps.texts.tab_stops_clear` | 在线文字文档文本的制表位：支持 插入/设置 | `file_id`, `link_id`, `url`, `verb` |
| `wps.texts.theme_color` | 设置文本的主题色 | `file_id`, `link_id`, `paragraph_index`, `url` |
| `wps.texts.two_lines_in_one` | 设置文本的双行合一 | `file_id`, `link_id`, `url` |
| `wps.texts.underline_color` | 在线文字文档文本的下划线颜色：支持 设置/查询 | `file_id`, `link_id`, `url`, `verb` |
| `wps.texts.word_count` | 查询文本的字数统计 | `file_id`, `link_id`, `url` |

- 定位段落用 wps.texts.range（paragraph_index → begin/end）；按内容定位用 wps.texts.search（find_text → ranges）。
- 插入/修改前先定位：ranges 类操作回填 begin/end（0-based），paragraphs 类操作回填 paragraph_index（1-based）。

## 二十五、目录操作

> 目录域相关操作。

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `wps.tocs.all` | 删除全部目录 | `file_id`, `link_id`, `url` |
| `wps.tocs.count` | 查询目录数量 | `file_id`, `link_id`, `url` |
| `wps.tocs.data` | 查询、插入或删除在线文字文档目录 | `file_id`, `link_id`, `url`, `scope`, `verb` |
| `wps.tocs.figures` | 插入图表目录 | `file_id`, `link_id`, `url` |
| `wps.tocs.format` | 设置目录格式 | `file_id`, `link_id`, `url` |
| `wps.tocs.update` | 设置目录 | `file_id`, `link_id`, `url` |
| `wps.tocs.update_all` | 设置全部目录 | `file_id`, `link_id`, `url` |

## 二十六、水印操作

> 水印域相关操作。

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `wps.watermarks.data` | 删除水印 | `file_id`, `link_id`, `url` |
| `wps.watermarks.image` | 插入图片水印 | `file_id`, `file_path`, `link_id`, `url` |
| `wps.watermarks.query` | 查询文档水印 | `file_id`, `link_id`, `url` |
| `wps.watermarks.text` | 插入文本水印 | `file_id`, `link_id`, `text`, `url` |
| `wps.watermarks.text_italic` | 插入斜体文本水印 | `file_id`, `link_id`, `text`, `url` |

## 二十七、WordML/XML 插入

> WordML/XML 插入域相关操作。

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `wps.xmls.insert` | 按字符区间插入 WordML/OOXML | `file_id`, `link_id`, `url`, `xml` |

选定上表工具后，调用 `kdocs-cli help <tool>` 获取工具卡参数说明（`<tool>` 与表中点号名一致）。

### 图片类写操作

以下工具需传入**公网可访问的图片 URL**（服务端会拉取图片）：

| 工具 | 顶层参数 | 说明 |
|------|----------|------|
| `wps.watermarks.image` | `file_path` | 插入图片水印 |
| `wps.shapes.picture` | `file_path` | 插入浮动图片形状 |

## 典型用途

| 场景 | 说明 | 工作流 |
|------|------|--------|
| 文档导出 | 通过 `wps.export`、`wps.export_image`、`wps.query_export` | — |
| AP 生成 | 通过 `wps.export(format=ap)` 与 `wps.query_export(format=ap)` | — |
| 内容读写 | 通过 `wps.texts.content`、`wps.texts.range`、`wps.texts.count` 等 | — |
| 查找替换 | 通过 `wps.texts.search` / `wps.texts.replace` | — |
| 段落格式 | 通过 `wps.texts.alignment`、`wps.texts.indent`、`wps.texts.line_spacing` 等 | — |
| 字符样式（多目标） | 按关键词检索后批量改字体/高亮/下划线等 | `references/workflows/wps-multi-target-search-style.md` |
| 标题段（大纲） | 按 outline_level 识别标题段后改色/对齐/字距 | `references/workflows/wps-heading-by-outline.md` |
| 段落对齐缩进边框 | 对齐、缩进、行距、段边框与底纹 | `references/workflows/wps-paragraph-align-indent.md` |
| 节下正文 | 锚定小节后批量字号/对齐/行距 | `references/workflows/wps-section-body-format.md` |
| 列表编号 | 项目符号、数字编号、多级列表 | `references/workflows/wps-lists-and-indent.md` |
| 清除与字体替换 | 清除格式、内置样式、字体替换 | `references/workflows/wps-clear-and-replace-font.md` |
| 内置样式/ranges | 指定页套用样式、全文间距字号 | `references/workflows/wps-styles-apply-by-page.md` |
| 页眉页脚页码 | 页眉文字、页脚页码、罗马页码 | `references/workflows/wps-header-footer-pagination.md` |
| 分节版式 | 分页、分节、页边距、边框、水印 | `references/workflows/wps-section-break-layout.md` |
| 附件节横向 | 目录与附件分节组合 | `references/workflows/wps-combo-section-attachment.md` |
| 目录 | 插入/更新/生成目录 | `references/workflows/wps-toc-lifecycle.md` |
| 题注域公式 | 图片/表格题注、日期域、公式 | `references/workflows/wps-captions-and-fields.md` |
| 公文与书签 | 公文字体、书签增删 | `references/workflows/wps-official-and-bookmarks.md` |
| 复制粘贴格式刷 | 纯文本/保留格式/格式刷 | `references/workflows/wps-copy-paste-format.md` |
| 图片 | 插入、缩放、裁剪、环绕 | `references/workflows/wps-image-on-page.md` |
| 文本框形状 | 文本框与简单形状文案 | `references/workflows/wps-shapes-textbox.md` |
| 形状组合叠放 | 箭头、组合、置顶、删图 | `references/workflows/wps-shapes-draw-compose.md` |
| 表格行列格式 | 整行或整列的对齐、垂直对齐、字体 | `references/workflows/wps-table-row-column.md` |
