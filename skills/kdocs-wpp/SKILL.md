# 演示文稿（pptx / wpp）（工具路由索引）

本文件为领域说明；工具参数以 `kdocs-cli help <tool>` 工具卡为准。

**适用范围**：支持幻灯片插入、复制、删除与页序调整，文字、图片、形状、表格和图表编辑，备注、批注、标签与动画操作，母版及主题属性查询，主题字体与配色设置，以及 PDF 和图片导出。

**不适用范围**：AI 演示生成、通用内容提取、上传及云盘文件管理。遇到上述域外需求、需要借助其他组件定位文件或准备外部资源，或无法判断请求应由哪个组件处理时，[返回主 Skill](../../SKILL.md)安排，不自行切换子 Skill。

---

## 通用说明

### 演示文稿工具概述

- 若只是读取幻灯片正文/结构，按[主 Skill「操作域路由」中的「读取」项](../../SKILL.md#操作域路由)处理。


### 使用场景

| 场景 | 说明 |
|------|------|
| 工作汇报 / 对外宣讲 | 改版式与内容、批注协作、导出 PDF / 图片 |
| 培训课件 | 页序与版式、动画与备注、形状与图表 |
| 模板与品牌统一 | 母版/主题查询、全文或单页字体与配色 |

### 文档定位

- 除创建空白演示 / 纯任务查询外，定位参数 `url` / `link_id` / `file_id` **三选一**。

> **工具填参**：下表每行 `<tool>` 选定后须 `kdocs-cli help <tool>`（可批量：`kdocs-cli help wpp.set_font_presentation wpp.set_font_slide`）。

## 一、演示文稿 · 文档/文件属性

> facet: presentation.properties

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `wpp.presentation.properties.builtin_properties` | 查询演示文稿的内置文档属性 |  |
| `wpp.presentation.properties.close` | 关闭演示文稿（关闭当前演示文稿会话） |  |
| `wpp.presentation.properties.custom_properties` | 查询演示文稿的自定义文档属性 |  |
| `wpp.presentation.properties.page_setup` | 查询演示文稿的页面设置（页面宽高与方向） |  |
| `wpp.presentation.properties.read_only` | 查询演示文稿的只读状态 |  |
| `wpp.presentation.properties.remarks` | 查询演示文稿的全部备注 |  |
| `wpp.presentation.properties.saved` | 查询演示文稿的已保存状态 |  |
| `wpp.presentation.properties.template_name` | 查询演示文稿的模板名称 |  |

## 二、演示文稿 · 显示与主题

> facet: presentation.properties

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `wpp.presentation.properties.color_scheme` | 查询演示文稿的配色方案 |  |
| `wpp.presentation.properties.designs` | 查询演示文稿的设计集合 |  |
| `wpp.presentation.properties.display_comments` | 演示文稿的显示批注：支持 查询/设置（是否在全稿范围显示批注） | `verb` |
| `wpp.presentation.properties.extra_colors` | 查询演示文稿的额外颜色 |  |
| `wpp.presentation.properties.fonts` | 查询演示文稿的字体集合 |  |
| `wpp.presentation.properties.line_break` | 查询演示文稿的换行字符 |  |
| `wpp.presentation.properties.section_properties` | 查询演示文稿的节属性 |  |

## 三、演示文稿 · 节

> facet: presentation.sections

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `wpp.presentation.sections.count` | 查询演示文稿分节分节总数 |  |
| `wpp.presentation.sections.data` | 演示文稿分节分节实体：支持 删除/插入 | `name`, `section_index`, `verb` |
| `wpp.presentation.sections.list` | 查询演示文稿分节分节列表 |  |
| `wpp.presentation.sections.name` | 演示文稿分节分节名称：支持 查询/设置 | `name`, `section_index`, `verb` |
| `wpp.presentation.sections.order` | 设置演示文稿分节分节顺序 | `section_index`, `to_pos` |
| `wpp.presentation.sections.slide_ids` | 查询演示文稿分节分节幻灯片列表 |  |

## 四、演示文稿 · 标签

> facet: presentation.tags

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `wpp.presentation.tags.add` | 添加演示文稿标签（演示文稿级标签，ADD 时可传 value） |  |
| `wpp.presentation.tags.delete` | 删除演示文稿标签 |  |
| `wpp.presentation.tags.tags` | 查询演示文稿标签：标签列表 |  |

## 五、幻灯片 · 页序与增删

> facet: slide.items

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `wpp.slide.items.batch_slides` | 批量插入多张幻灯片（通过 items[] 一次插入多张幻灯片 | `items` |
| `wpp.slide.items.count` | 查询幻灯片总数（返回演示文稿幻灯片页数） |  |
| `wpp.slide.items.delete` | 删除幻灯片（需 slide_id） | `slide_id` |
| `wpp.slide.items.duplicate` | 设置幻灯片：就地复制（需 slide_id，复制到相邻位置） | `slide_id` |
| `wpp.slide.items.insert` | 插入幻灯片：插入空白页 |  |
| `wpp.slide.items.list` | 查询幻灯片列表（返回全部幻灯片摘要列表） |  |
| `wpp.slide.items.move` | 设置幻灯片：移动位置（需 slide_id 与 to_position） | `slide_id`, `to_position` |

## 六、幻灯片 · 单页属性

> facet: slide.items

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `wpp.slide.items.background` | 查询幻灯片：背景（含填充类型与前景色；需 slide_id） | `slide_id` |
| `wpp.slide.items.datetime` | 设置幻灯片：datetime | `slide_id` |
| `wpp.slide.items.footer` | 设置幻灯片：footer | `slide_id` |
| `wpp.slide.items.layout` | 幻灯片：版式：支持 查询/设置（需 slide_id 与 layout 版式索引 | `layout`, `slide_id`, `verb` |
| `wpp.slide.items.master` | 查询幻灯片：关联母版 | `slide_id` |
| `wpp.slide.items.name` | 幻灯片：名称：支持 查询/设置（需 slide_id） | `name`, `slide_id`, `verb` |
| `wpp.slide.items.page_number` | 设置幻灯片：page number | `slide_id` |
| `wpp.slide.items.slide_id` | 查询幻灯片：幻灯片 ID | `slide_id` |
| `wpp.slide.items.transition` | 查询幻灯片：切换动画 | `slide_id` |

## 七、幻灯片 · 备注

> facet: slide.notes

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `wpp.slide.notes.append` | append幻灯片append | `slide_id` |
| `wpp.slide.notes.create` | 插入幻灯片create | `slide_id` |
| `wpp.slide.notes.notes` | 查询幻灯片备注页内容（需 slide_id） | `slide_id` |
| `wpp.slide.notes.set` | 设置幻灯片set | `slide_id` |

## 八、幻灯片 · 标签

> facet: slide.tags

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `wpp.slide.tags.add` | 添加幻灯片标签（需 slide_id、name，ADD 时可传 value） | `slide_id` |
| `wpp.slide.tags.delete` | 删除幻灯片标签（需 slide_id 与 name） | `slide_id` |
| `wpp.slide.tags.tags` | 查询幻灯片标签：标签列表（需 slide_id；可按 name 过滤） | `slide_id` |

## 九、幻灯片 · 批注

> facet: slide.comments

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `wpp.slide.comments.count` | 查询幻灯片评论评论总数 |  |
| `wpp.slide.comments.data` | 幻灯片评论评论实体：支持 删除/插入 | `author`, `author_initials`, `comment_index`, `left`, `slide_id`, `text`, `top`, `verb` |
| `wpp.slide.comments.list` | 查询幻灯片评论评论列表 |  |
| `wpp.slide.comments.reply` | 设置幻灯片评论回复评论 | `author`, `author_initials`, `comment_index`, `slide_id`, `text` |
| `wpp.slide.comments.thread` | 查询幻灯片评论回复线程 |  |

## 十、幻灯片 · 动画

> facet: slide.animations

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `wpp.slide.animations.data` | 幻灯片动画动画实体：支持 删除/插入 | `effect_id`, `effect_index`, `shape_id`, `slide_id`, `verb` |
| `wpp.slide.animations.effects` | 查询幻灯片动画动画效果列表 | `slide_id` |
| `wpp.slide.animations.order` | 设置幻灯片动画动画顺序 | `effect_index`, `slide_id`, `to_pos` |
| `wpp.slide.animations.timeline` | 查询幻灯片动画动画时间线 | `slide_id` |
| `wpp.slide.animations.timing` | 设置幻灯片动画动画时序 | `effect_index`, `slide_id`, `timing` |
| `wpp.slide.animations.trigger` | 设置幻灯片动画触发方式 | `effect_index`, `slide_id`, `trigger` |
| `wpp.slide.animations.types` | 查询幻灯片动画动画类型列表 | `slide_id` |

## 十一、形状 · 列举与基础

> facet: shape.items

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `wpp.shape.items.delete` | 删除指定形状（需 slide_id 与 shape_id） | `slide_id` |
| `wpp.shape.items.duplicate` | 设置形状：就地复制 | `slide_id` |
| `wpp.shape.items.item` | 查询形状：单条形状详情（需 slide_id 与 shape_id | `slide_id` |
| `wpp.shape.items.list` | 查询形状：形状列表 | `slide_id` |
| `wpp.shape.items.z_order` | 设置形状：叠放顺序 | `slide_id` |

## 十二、形状 · 插入

> facet: shape.items

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `wpp.shape.items.line` | 设置形状：描边（需 slide_id、shape_id | `slide_id` |
| `wpp.shape.items.lines` | 插入在指定幻灯片批量插入线条（批量 lines[]；需 slide_id） | `lines`, `slide_id` |
| `wpp.shape.items.picture` | 插入在指定幻灯片插入图片形状（单条平铺：image_url + 位置尺寸 | `image_url`, `slide_id` |
| `wpp.shape.items.tables` | 插入在指定幻灯片批量插入表格（批量 tables[]；需 slide_id） | `slide_id`, `tables` |
| `wpp.shape.items.textboxes` | 插入在指定幻灯片批量插入文本框（批量 textboxes[] | `slide_id`, `textboxes` |

## 十三、形状 · 组合与几何

> facet: shape.items

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `wpp.shape.items.fill` | 设置形状：填充（需 slide_id、shape_id | `slide_id` |
| `wpp.shape.items.geometry` | 设置形状：位置与尺寸（需 slide_id 与 shape_id | `slide_id` |
| `wpp.shape.items.group` | 设置形状：组合（组合或取消组合选中形状） | `slide_id` |
| `wpp.shape.items.group_children` | 查询形状：组合子形状（需 slide_id 与 shape_id | `slide_id` |
| `wpp.shape.items.name` | 设置形状：名称 | `name`, `slide_id` |
| `wpp.shape.items.reorder_in_group` | 设置形状：组内排序 | `child_shape_ids`, `slide_id` |
| `wpp.shape.items.shapes` | 插入在指定幻灯片批量插入选自图形（批量 shapes[]（auto_shape_ | `shapes`, `slide_id` |
| `wpp.shape.items.ungroup` | 设置形状：取消组合 | `slide_id` |
| `wpp.shape.items.update_group_props` | 设置形状：组合属性 | `props`, `slide_id` |

## 十四、形状 · 文本

> facet: shape.texts

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `wpp.shape.texts.append` | append形状append | `slide_id` |
| `wpp.shape.texts.find_replace` | 设置形状find replace | `slide_id` |
| `wpp.shape.texts.font` | 查询形状字体格式（需 slide_id、shape_id | `slide_id` |
| `wpp.shape.texts.font_paragraph` | 设置形状字体格式段落格式 | `slide_id` |
| `wpp.shape.texts.paragraph` | 查询形状段落格式（需 slide_id 与 shape_id） | `slide_id` |
| `wpp.shape.texts.text` | 形状文本内容：支持 查询/设置（需 slide_id 与 shape_id） | `slide_id`, `verb` |

## 十五、形状 · 表格

> facet: shape.tables

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `wpp.shape.tables.cell_merge` | 设置形状表格合并单元格 | `slide_id` |
| `wpp.shape.tables.cell_text` | 形状表格单元格文本：支持 查询/设置 | `slide_id`, `verb` |
| `wpp.shape.tables.cell_unmerge` | 设置形状表格取消单元格合并 | `slide_id` |
| `wpp.shape.tables.cells` | 查询形状表格单元格列表 | `slide_id` |
| `wpp.shape.tables.column` | 设置形状表格表格列 | `slide_id` |
| `wpp.shape.tables.dimensions` | 查询形状表格行列数 | `slide_id` |
| `wpp.shape.tables.merged_ranges` | 查询形状表格合并单元格区间 | `slide_id` |
| `wpp.shape.tables.row` | 设置形状表格表格行 | `slide_id` |

## 十六、形状 · 图表数据

> facet: shape.charts

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `wpp.shape.charts.data` | 形状图表插入空白页：支持 插入/查询/设置 | `height`, `left`, `slide_id`, `top`, `verb`, `width` |
| `wpp.shape.charts.data_labels` | 形状图表数据标签：支持 查询/设置 | `slide_id`, `verb` |
| `wpp.shape.charts.series_style` | 设置形状图表系列样式 | `slide_id` |

## 十七、形状 · 图表版式

> facet: shape.charts

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `wpp.shape.charts.axis` | 形状图表坐标轴：支持 查询/设置 | `slide_id`, `verb` |
| `wpp.shape.charts.chart_type` | 形状图表图表类型：支持 查询/设置 | `slide_id`, `verb` |
| `wpp.shape.charts.gridlines` | 形状图表网格线：支持 查询/设置 | `slide_id`, `verb` |
| `wpp.shape.charts.legend` | 形状图表图例：支持 查询/设置 | `slide_id`, `verb` |
| `wpp.shape.charts.title` | 形状图表标题：支持 查询/设置 | `slide_id`, `verb` |
| `wpp.shape.charts.trendline` | 形状图表趋势线：支持 查询/设置 | `slide_id`, `verb` |

## 十八、形状 · 标签

> facet: shape.tags

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `wpp.shape.tags.add` | 添加形状标签 | `slide_id` |
| `wpp.shape.tags.delete` | 删除形状标签 | `slide_id` |
| `wpp.shape.tags.tags` | 查询形状标签：标签列表（需 slide_id 与 shape_id） | `slide_id` |

## 十九、形状 · Web 形状

> facet: shape.webshapes

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `wpp.shape.webshapes.content` | 插入WebShape WebShape 内容（插入 WebShape | `height`, `slide_id`, `width` |

## 二十、母版 · 主题与版式

> facet: master.properties

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `wpp.master.properties.background` | 查询母版的背景（含填充类型与前景色 |  |
| `wpp.master.properties.color_scheme` | 查询母版的配色方案 |  |
| `wpp.master.properties.custom_layouts` | 查询母版的自定义版式（返回母版下全部自定义版式） |  |
| `wpp.master.properties.delete` | 删除母版（删除指定类型母版） |  |
| `wpp.master.properties.design` | 查询母版的设计 |  |
| `wpp.master.properties.name` | 母版的名称：支持 查询/设置（母版名称） | `verb` |
| `wpp.master.properties.size` | 查询母版的页面尺寸（幻灯片页面宽高） |  |
| `wpp.master.properties.theme` | 查询母版的主题 |  |

## 二十一、母版 · 内容与动效

> facet: master.properties

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `wpp.master.properties.headers_footers` | 查询母版的页眉页脚 |  |
| `wpp.master.properties.hyperlinks` | 查询母版的超链接 |  |
| `wpp.master.properties.shapes` | 查询母版的形状集合（母版上的占位形状） |  |
| `wpp.master.properties.text_styles` | 查询母版的文本样式 |  |
| `wpp.master.properties.timeline` | 查询母版的动画时间线 |  |
| `wpp.master.properties.transition` | 查询母版的切换动画 |  |

## 二十二、导出

> facet: export

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `wpp.export.export.image` | 导出在线演示文稿为图片 |  |
| `wpp.export.export.pdf` | 导出在线演示文稿为 PDF（v3 异步导出；创建任务或按 task_id 轮询） |  |

## 二十三、主题（字体与配色）

> 演示文稿或单页的字体与配色方案设置

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `wpp.set_font_presentation` | 全文更换字体 | `url`, `link_id`, `file_id` |
| `wpp.set_font_slide` | 单页更换字体 | `url`, `link_id`, `file_id`, `slide_idx` |
| `wpp.set_color_presentation` | 全文更换配色 | `url`, `link_id`, `file_id` |
| `wpp.set_color_slide` | 单页更换配色 | `url`, `link_id`, `file_id`, `slide_idx`, `theme_color_mode`, `color_scheme_id` |

### 字体与配色约束

| 项 | 规则 |
|----|------|
| 字体主题 | 支持 **16** 种方案名：`现代雅黑体`、`简约等线体`、`经典宋体`、`清秀姚体`、`经典黑体`、`现代黑体`、`简约灵秀黑体`、`科技云技术体`、`经典楷体`、`典雅气质体`、`简约中等线体`、`古朴小隶体`、`传统书宋二体`、`圆润体`、`可爱傲娇体`、`飘逸青云体`。`font_theme` 未传或非法 → **经典黑体** |
| 配色主题 | 支持 **16** 种方案名，每种配色对应 `theme_color_mode` 值如下表 |

#### 配色主题详表

| `color_theme` | `theme_color_mode` | 说明 |
|---------------|-------------------|------|
| 默认配色 | 0 | 恢复默认配色 |
| 落日红 | 1 | 浅色暖色系 |
| 蜜橘橙 | 1 | 浅色暖色系 |
| 琥珀黄 | 1 | 浅色暖色系 |
| 嫩芽绿 | 1 | 浅色冷色系 |
| 湖水青 | 1 | 浅色冷色系 |
| 晴空蓝 | 1 | 浅色冷色系 |
| 丁香紫 | 1 | 浅色冷色系 |
| 朱砂赤 | 3 | 深色暖色系 |
| 南瓜橙 | 3 | 深色暖色系 |
| 深麦黄 | 3 | 深色暖色系 |
| 深松绿 | 3 | 深色冷色系 |
| 深墨青 | 3 | 深色冷色系 |
| 深海蓝 | 3 | 深色冷色系 |
| 葡萄紫 | 3 | 深色冷色系 |
| 胭脂红 | 3 | 深色暖色系 |

`color_theme` 未传或非法 → **默认配色**（`theme_color_mode: 0`）

## 常用工作流

#### 演示文稿导出 PDF

```
wpp.export.export.pdf(file_id="xxx", from_slide=1, to_slide=9999)
        → 若返回 task_id：再调用 wpp.export.export.pdf(task_id="...", task_type="normal_export") 轮询
        → 成功时从 data 取 url/key 等下载信息；可选 from_slide / to_slide 限定页码范围
```
notes:
- 导出为异步任务；无 task_id 时按工具返回字段判断是否已直接完成
- 若返回带下载链接，获取后应尽快使用或返回给用户（可能有时效）

调用接口前先执行以下命令获取接口参数及约束：
`kdocs-cli help wpp.export.export.pdf`

## 附录

### 错误响应

| 情况 | 说明或示例 |
|------|------------|
| 未登录 / 无权限 | 返回业务错误码或 `result` 非 `ok`，`msg` 含可读原因 |
| 内核执行失败 | 类似 `detail.res[].code` 非 `0`，或 `result` 为 `ExecuteFailed` 等（以实际为准） |
| HTTP 非 200 | 请求失败，检查鉴权（Cookie、Origin 等） |

### 响应格式

若返回带外层 `code` / `msg` / `data`，业务结果以 `data` 内、各工具「返回值说明」中的 JSON 为准。
