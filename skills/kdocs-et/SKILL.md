# 表格文档/智能表格（xlsx & ksheet）（工具路由索引）

本文件包含金山文档 Skill 中表格相关工具的完整 API 说明、详细调用示例、参数说明和返回值说明。

**适用范围**：支持云端 Excel 文件和智能表格的工作表管理、区域数据与公式读写、批量更新、跨表字段回填、排序筛选、条件格式、数据校验、图表、数据透视表、浮动图片、行高列宽及区域权限设置。`sheet.*` 工具同时适用于 Excel（.xlsx）和智能表格（.ksheet）。 当前渠道未启用 JSAPI。

**不适用范围**：云盘文件管理、通用创建上传、多维表格及不受支持的 JSAPI。智能表格禁止 JSAPI。遇到上述域外需求、需要借助其他组件定位文件或准备外部资源，或无法判断请求应由哪个组件处理时，[返回主 Skill](../../SKILL.md)安排，不自行切换子 Skill。

以下工具不可逆，调用前必须向用户确认；参数与边界以 `kdocs-cli help <tool>` 的工具卡 为准：

`sheet.delete_sheets`、`sheet.delete_range_data`、`sheet.delete_protection_ranges`、`sheet.delete_data_validations`、`sheet.delete_conditional_format_rules`、`sheet.delete_float_images`、`sheet.delete_filters`、`sheet.delete_pivot_table`

### 区域读取参数示例

调用 `sheet.get_range_data` 前，执行 `kdocs-cli help sheet.get_range_data` 获取参数与返回值说明。`worksheet_id` 使用 `sheet.get_sheets_info` 返回的 `sheetId`；四个坐标写在 `range` 对象内，字段名为 `rowFrom`、`rowTo`、`colFrom`、`colTo`，索引从 0 开始。

使用 worksheet_id 和 range 对象读取 A1:F11 区域（前 11 行、前 6 列）：

```json
{
  "file_id": "VsdfG0001234567",
  "worksheet_id": 3,
  "range": {
    "rowFrom": 0,
    "rowTo": 10,
    "colFrom": 0,
    "colTo": 5
  }
}
```

示例中的 `3` 应替换为目标工作表返回的 `sheetId`。

---

### 表格工具概述

表格工具专门用于操作金山文档中的在线表格，提供工作表信息的查询、范围数据的获取以及批量更新等功能。支持两种表格类型：

- **Excel（.xlsx）**：传统在线表格
- **智能表格（.ksheet）**：高级结构化表格


### ET 输出纪律（`sheet.*`）

面向 `.xlsx` / `.ksheet` 的云端写入，遵守以下纪律（填参前对表内点号名执行 `kdocs-cli help <tool>`）：

1. **公式优先**：可公式表达的计算写在单元格公式中（如 `=SUM(A2:A9)`），禁止在 Agent 侧算出数字再硬编码填入；仅当公式无法表达的一次性分析或固定标题可用常量。
2. **结果落在表里**：汇总、统计须体现在单元格；禁止只在对话里报数而不写回表格。
3. **批量写入**：禁止逐格串行调用；用 `sheet.range_data_batch_update`（或 `update_range_data` 兜底）一次提交多格/多行；大范围先 `sheet.get_range_data` 再改后写回。
4. **超级表 / 结构化引用**：写列合计前确认表名与列名（先 `sheet.get_range_data` 看清区域）；有超级表时列合计用 `TableName[列名]`，合计行写在表区域**外**，避免 `=SUM(Table[列])` 写在表内合计行导致循环引用；无表且数据会增长时可考虑整列引用。
5. **写后验证**：批量写入后用 `sheet.get_range_data` 回读关键格，不信任单次 `code: 0`。

## 表格选区更新工具差异

| 对比项 | `sheet.update_range_data` | `sheet.range_data_batch_update` | 选择建议 |
|--------|---------------------------|----------------------------------|----------|
| 写入文本/公式 | 支持 | 支持 | 两者均可 |
| 设置格式/合并单元格 | 支持 | 支持 | 两者均可 |
| 插入图片 | 不支持 | 支持 | 涉及图片写入时优先使用 `sheet.range_data_batch_update` |
| 参数风格 | camelCase（如 `rangeData`） | 下划线参数（如 `range_data`） | 与上下游参数风格保持一致 |

> 提示：`sheet.range_data_batch_update` 为唯一优先入口；`sheet.update_range_data` 仅作兜底（batch 写入失败时使用）。

删行/列并让其余内容上移或左移 → `sheet.delete_range_data`（`shift_up` / `shift_left`）。只清格子、不移动其余单元格 → 用上面两个写入工具写空值，不要调用 `sheet.delete_range_data`。

---

### 写入/更新已有表格

批量改单元格、筛选、条件格式等 → 下文 `sheet.*` 与「工具组合速查」。精读区域用 `sheet.get_range_data`；概览首读可用 `read_file`（可选 `sheet_name`、`sheet_range`），`read_file` 参数约束参看[主 Skill「操作域路由」中的「读取」项](../../SKILL.md#操作域路由)。


### Excel vs 智能表格（ksheet）对比

| 特性 | Excel | 智能表格 ksheet |
|------|-------|----------------|
| 数据组织 | 传统行列表格 | 结构化字段+记录 |
| 视图 | 单一表格视图 | 多视图（表格/看板/日历/甘特图等） |
| 字段类型 | 通用单元格 | 丰富字段类型（单选/多选/日期/附件/关联等） |
| 适用场景 | 数据计算、报表、财务报表 | 项目管理、CRM、任务跟踪、库存管理 |
| 工作表/数据接口 | 使用 `sheet.*` 工具 | **同样使用 `sheet.*` 工具** |

### 使用场景

#### Excel 适用场景

| 场景 | 说明 |
|------|------|
| 数据记录 | 销售数据、财务报表 |
| 数据分析 | 结构化数据的读取与处理 |
| 报表汇总 | 多维度数据汇总 |
| 公式计算 | 需要复杂公式和数据透视 |

#### 智能表格 适用场景

| 场景 | 说明 |
|------|------|
| 项目管理 | 任务分配、进度跟踪 |
| CRM 管理 | 客户信息、跟进记录 |
| 资产管理 | 库存台账、设备管理 |
| 审批台账 | 合同风险排查台账等 |

### 类型选择建议

- 需要公式计算、数据透视 → 选 **Excel**
- 需要多视图、字段管理、看板展示 → 选 **ksheet**
- 需要做任务管理/项目跟踪 → 选 **ksheet**
- 需要做财务报表 → 选 **Excel**

> **注意**：`ksheet` 文件创建完成后，和 Excel 一样继续使用 `sheet.*` 接口做工作表管理与数据操作。
> **ksheet 与多维表格区分**：本子 Skill 处理 Excel 和智能表格；多维表格的数据表、关联记录、Webhook 等操作返回主 Skill。

### 表格增强能力

除基础的工作表管理与区域读写外，智能表格（.ksheet）和普通Excel（.xlsx）还支持以下增强能力：

- **区域权限**：可通过 `sheet.list_protection_ranges`、`sheet.create_protection_ranges`、`sheet.update_protection_ranges`、`sheet.delete_protection_ranges` 管理指定区域的访问或编辑限制
- **数据校验**：可通过 `sheet.get_data_validations`、`sheet.create_data_validations`、`sheet.update_data_validations`、`sheet.delete_data_validations` 配置下拉选项和输入约束
- **条件格式**：可通过 `sheet.get_conditional_format_rules`、`sheet.create_conditional_format_rules`、`sheet.update_conditional_format_rules`、`sheet.delete_conditional_format_rules` 管理自动高亮和格式规则
- **浮动图片**：可通过 `sheet.list_float_images`、`sheet.get_float_image`、`sheet.create_float_images`、`sheet.update_float_images`、`sheet.delete_float_images` 管理工作表中的浮动图片
- **筛选与工作表编排**：可通过 `sheet.get_filters`、`sheet.open_filters`、`sheet.update_filters`、`sheet.delete_filters` 管理筛选状态，并通过 `sheet.copy_worksheet`、`sheet.update_worksheet` 复制、重命名或移动工作表

> **补充说明**：上述增强能力可同时面向智能表格（.ksheet）和普通 Excel（.xlsx）。

> **工具填参**：下表每行 `<tool>` 选定后须 `kdocs-cli help <tool>`（可批量：`kdocs-cli help sheet.get_sheets_info sheet.add_sheet`）。

## 一、工作表管理

> 智能表格/Excel 工作表的创建、获取、复制、重命名、删除和冻结行列

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `sheet.get_sheets_info` | 获取工作表列表 | `url`, `link_id`, `file_id` |
| `sheet.add_sheet` | 新增工作表 | `url`, `link_id`, `file_id` |
| `sheet.update_sheet` | 更新工作表 | `url`, `link_id`, `file_id`, `worksheet_id` |
| `sheet.delete_sheets` | 删除工作表 | `url`, `link_id`, `file_id` |
| `sheet.copy_worksheet` | 复制工作表 | `url`, `link_id`, `file_id`, `worksheet_id` |
| `sheet.update_worksheet` | 更新工作表名称或位置 | `url`, `link_id`, `file_id`, `worksheet_id` |
| `sheet.freeze` | 设置冻结行列 | `url`, `link_id`, `file_id`, `worksheet_id`, `frozen_row_count`, `frozen_col_count` |

## 二、数据操作

> 区域数据读写、行列移动、单元格插入、合并区域查询与取消合并、样式清除、超链接及附件下载

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `sheet.get_range_data` | 获取选区数据 | `url`, `link_id`, `file_id`, `worksheet_id`, `range` |
| `sheet.range_data_batch_update` | 批量更新区域数据（下划线参数版本） | `url`, `link_id`, `file_id`, `worksheet_id`, `range_data` |
| `sheet.delete_range_data` | 删除选区并移位 | `url`, `link_id`, `file_id`, `worksheet_id`, `range_data` |
| `sheet.add_row` | 追加一行数据 | `url`, `link_id`, `file_id`, `worksheet_id` |
| `sheet.find_range_data` | 遍历筛选记录（支持分页与条件） | `url`, `link_id`, `file_id`, `worksheet_id`, `range` |
| `sheet.get_attachment_url` | 上传附件到文件 | `file_id`, `filename`, `url`, `content_base64`, `Content-Type` |
| `sheet.add_chart` | 在工作表中新增图表 | `url`, `link_id`, `file_id`, `worksheet_id`, `chartType`, `sourceAddress`, `rectAddress`, `title` |
| `sheet.range_auto_fill` | 用源区域自动填充目标区域 | `url`, `link_id`, `file_id`, `worksheet_id`, `sourceAddress`, `targetAddress` |
| `sheet.get_chart_information` | 获取工作表图表信息 | `url`, `link_id`, `file_id`, `worksheet_id` |
| `sheet.update_chart` | 修改图表属性 | `url`, `link_id`, `file_id`, `worksheet_id`, `index`, `updateType` |
| `sheet.create_pivot_table` | 创建数据透视表 | `url`, `link_id`, `file_id`, `worksheet_id`, `sourceAddress` |
| `sheet.merge_range` | 合并单元格区域 | `url`, `link_id`, `file_id`, `worksheet_id`, `range` |
| `sheet.range_sort` | 对区域进行原地排序 | `url`, `link_id`, `file_id`, `worksheet_id`, `range`, `key` |
| `sheet.update_pivot_table` | 修改数据透视表属性或执行透视表操作 | `url`, `link_id`, `file_id`, `worksheet_id`, `tableName`, `updateType` |
| `sheet.update_pivot_field` | 修改数据透视表字段属性或执行字段级操作 | `url`, `link_id`, `file_id`, `worksheet_id`, `tableName`, `fieldName`, `updateType` |
| `sheet.delete_pivot_table` | 删除整个数据透视表 | `url`, `link_id`, `file_id`, `worksheet_id`, `tableName` |
| `sheet.get_pivot_tables` | 获取数据透视表信息（支持透视表/字段/数据项三级下钻） | `url`, `link_id`, `file_id`, `worksheet_id` |
| `sheet.auto_fit` | 自适应列宽或行高 | `url`, `link_id`, `file_id`, `worksheet_id`, `range` |
| `sheet.get_typed_value` | 读取区域数据并保留单元格类型 | `url`, `link_id`, `file_id`, `worksheet_id`, `range` |
| `sheet.update_range_data` | 批量更新选区数据 | `url`, `link_id`, `file_id`, `rangeData` |
| `sheet.insert_rows_cols` | 插入行或列 | `url`, `link_id`, `file_id`, `worksheet_id`, `type` |
| `sheet.set_range_width_height` | 调整行高列宽 | `url`, `link_id`, `file_id`, `worksheet_id`, `range` |
| `sheet.clear_range_styles` | 清除区域样式 | `url`, `link_id`, `file_id`, `worksheet_id`, `ranges` |
| `sheet.get_merged_cells` | 查询合并单元格 | `url`, `link_id`, `file_id`, `worksheet_id` |
| `sheet.insert_cells` | 插入空白单元格 | `url`, `link_id`, `file_id`, `worksheet_id`, `range`, `direction` |
| `sheet.move_rows_cols` | 移动行或列 | `url`, `link_id`, `file_id`, `worksheet_id`, `dimension_type`, `move_range_from`, `move_range_to`, `target_pos` |
| `sheet.set_hyperlink` | 设置或清空超链接 | `url`, `link_id`, `file_id`, `worksheet_id`, `range`, `hyperlink_url` |
| `sheet.unmerge_range` | 取消单元格合并 | `url`, `link_id`, `file_id`, `worksheet_id`, `range` |

## 三、筛选

> 筛选状态的开启、获取、更新、删除

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `sheet.get_filters` | 获取筛选条件 | `url`, `link_id`, `file_id`, `worksheet_id` |
| `sheet.open_filters` | 开启工作表筛选 | `url`, `link_id`, `file_id`, `worksheet_id`, `expand_filter_range`, `range` |
| `sheet.update_filters` | 更新列筛选条件 | `url`, `link_id`, `file_id`, `worksheet_id`, `col`, `condition` |
| `sheet.delete_filters` | 删除工作表筛选 | `url`, `link_id`, `file_id`, `worksheet_id` |

## 四、条件格式

> 条件格式规则的增删改查

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `sheet.get_conditional_format_rules` | 获取条件格式规则 | `url`, `link_id`, `file_id`, `worksheet_id` |
| `sheet.create_conditional_format_rules` | 创建条件格式规则 | `url`, `link_id`, `file_id`, `worksheet_id`, `rule` |
| `sheet.update_conditional_format_rules` | 更新条件格式规则 | `url`, `link_id`, `file_id`, `worksheet_id`, `rule` |
| `sheet.delete_conditional_format_rules` | 删除条件格式规则 | `url`, `link_id`, `file_id`, `worksheet_id`, `ranges` |

## 五、数据校验

> 下拉选项与输入约束的增删改查

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `sheet.get_data_validations` | 获取数据校验规则 | `url`, `link_id`, `file_id`, `worksheet_id`, `col_from`, `col_to`, `row_from`, `row_to` |
| `sheet.create_data_validations` | 创建数据校验规则 | `url`, `link_id`, `file_id`, `worksheet_id`, `args`, `field_type`, `range` |
| `sheet.update_data_validations` | 更新数据校验规则 | `url`, `link_id`, `file_id`, `worksheet_id`, `args`, `field_type`, `range` |
| `sheet.delete_data_validations` | 删除数据校验规则 | `url`, `link_id`, `file_id`, `worksheet_id`, `range` |

## 六、区域权限

> 区域访问/编辑权限的增删改查

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `sheet.list_protection_ranges` | 获取区域权限列表 | `url`, `link_id`, `file_id`, `worksheet_id` |
| `sheet.create_protection_ranges` | 创建区域权限 | `url`, `link_id`, `file_id`, `sheets_protection_infos` |
| `sheet.update_protection_ranges` | 批量更新区域权限 | `url`, `link_id`, `file_id`, `sheets_protection_infos` |
| `sheet.delete_protection_ranges` | 批量删除区域权限 | `url`, `link_id`, `file_id`, `sheets_protection_infos` |

## 七、浮动图片

> 浮动图片的增删改查与详情

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `sheet.list_float_images` | 查询浮动图片列表 | `url`, `link_id`, `file_id`, `worksheet_id` |
| `sheet.get_float_image` | 获取单张浮动图片详情 | `url`, `link_id`, `file_id`, `worksheet_id`, `float_image_id` |
| `sheet.create_float_images` | 创建浮动图片 | `file_id`, `worksheet_id`, `sheet_id`, `tag`, `x_pos`, `y_pos` |
| `sheet.update_float_images` | 更新浮动图片位置或尺寸 | `url`, `link_id`, `file_id`, `worksheet_id`, `float_image_id` |
| `sheet.delete_float_images` | 删除浮动图片 | `url`, `link_id`, `file_id`, `worksheet_id`, `float_image_id` |

## 八、对象查询

> 查询工作表对象，按类型、名称和区域筛选

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `sheet.get_sheet_objects` | 查询工作表对象 | `url`, `link_id`, `file_id`, `worksheet_id` |

### `sheet.find_range_data` 与 `sheet.get_range_data`的区别

| 对比项 | `sheet.find_range_data` | `sheet.get_range_data` |
|--------|-------------------------|------------------------|
| 核心用途 | 先筛选再返回命中数据 | 直接读取固定矩形范围数据 |
| 是否支持筛选条件 | 支持（`filter.condition`） | 不支持 |
| 是否支持搜索 | 支持（`filter.search`） | 不支持 |
| 是否支持去重 | 支持（`filter.duplicates`） | 不支持 |
| 是否支持分页 | 支持（`page.page` / `page.page_size`） | 不支持（一次返回范围内数据） |
| 是否可返回总数 | 支持（`show_total`） | 不支持 |
| 典型场景 | 大范围检索、按条件过滤、统计候选结果 | 精准读取已知区域（如 A1:F100） |
| 推荐使用时机 | 你只知道业务条件（例如“状态=已完成且金额>1000”），不知道具体行号，或结果很多需要分页浏览时 | 你已经知道要读的确切区域（例如 A1:F100），目标是快速拿到原始区域数据时 |

快速决策：**先确认用户意图是否需要筛选/搜索/去重/分页**。需要就用 `sheet.find_range_data`；不需要且范围已知就用 `sheet.get_range_data`。

### 数据透视表

创建或配置数据透视表时，**先读** `references/workflows/pivot-table.md`，按其中步骤调用 `sheet.get_sheets_info` → `sheet.create_pivot_table` → `sheet.update_pivot_table` → `sheet.get_pivot_tables`。参数以 workflow 为准；**勿通读**整份 `SKILL.md` 工具表。某一步参数不明时，在 `SKILL.md` 上表选定工具后执行 `kdocs-cli help sheet.<tool>`。

## 工具组合速查

| 用户需求 | 推荐工具组合 |
|----------|-------------|
| 表格概览/首读 | `read_file`（可选 `sheet_name`、`sheet_range`） |
| 读表格（矩形区域，精读/校验） | `sheet.get_sheets_info` → `sheet.get_range_data` |
| 跨表按主键/列名回填字段 | 见 `references/workflows/sheet-cross-fill.md` |
| 创建/配置数据透视表 | 见 `references/workflows/pivot-table.md` |
| 写表格（批量改单元格） | `sheet.get_range_data`（可选对照）→ `sheet.range_data_batch_update` → `sheet.get_range_data` 验证 |
| 给某列配置下拉选项 | `sheet.get_data_validations`（可选）→ `sheet.create_data_validations` / `sheet.update_data_validations` |
| 管理条件格式高亮 | `sheet.get_conditional_format_rules` → `sheet.create_conditional_format_rules` / `sheet.update_conditional_format_rules` / `sheet.delete_conditional_format_rules` |
| 管理区域权限 | `sheet.list_protection_ranges` → `sheet.create_protection_ranges` / `sheet.update_protection_ranges` / `sheet.delete_protection_ranges` |
| 管理表格筛选 | `sheet.get_filters` → `sheet.open_filters` / `sheet.update_filters` / `sheet.delete_filters` |
| 插入并调整浮动图片 | `sheet.create_float_images` → `sheet.get_float_image` / `sheet.update_float_images` |
| 上传本地文件作为表格附件 | `sheet.get_attachment_url`（本地内容用 `content_base64`，公网文件用 `url`）→ 使用返回的 `object_id` |
| 本地图片写入单元格（不新开一体工具） | `upload_attachment`（`content_base64` 或本地读入）→ 取 `object_id` → `sheet.range_data_batch_update`（`op_type=cell_operation_type_picture`，`cell_pic_info.tag=sheet_pic_type_attachment`，`pic_content`=`object_id`，并传 `width`/`height`）。外链图用 `sheet_pic_type_url` + URL。勿仿 ET `insert_picture("local", path)` 一条命令 |

---

## 高频操作流程

| 流程 | 说明 | 详细参考 |
|------|------|----------|
| 数据透视表（创建与配置） | 在 .xlsx/.ksheet 上按固定步骤创建透视表、配置行列值字段并回读验证（透视主路径；勿通读 references/sheet/data.md） | `references/workflows/pivot-table.md` |
| 接龙转表格 | 识别接龙文本内容，自动提取并转为在线表格 | `references/workflows/jielong-to-table.md` |
| 跨表字段回填（按主键匹配） | 两张表格按主键列（如订单号、企业ID、SKU）匹配，将源表指定列写入目标表对应列；须先读表头按列名定位，写后按列名回读验证 | `references/workflows/sheet-cross-fill.md` |
| 表格美化与数据规范 | 读取表格数据，进行格式美化、数据规范化和样式调整，并通过条件格式、数据校验、区域权限固化规则 | `references/workflows/table-beautify.md` |

## 错误速查表

| 错误特征 | 原因 | 处理方式 |
|----------|------|----------|
| `400100` invalid op_type / 参数值不被识别 | 凭直觉猜测 op_type/opType 等枚举值，未查文档 | 执行 `kdocs-cli help <tool>`，从 description 中的对照表取正确值，修正后重试一次 |
| `400001` 参数缺失 / 格式错误 | 未确认必填参数或参数类型（如 opType 漏传、rangeData 传错） | 执行 `kdocs-cli help <tool>`，确认参数名、类型、必填性，补齐后重试 |
| 表格读不到或结构不明 | 未先取工作表列表 / 区域错误 | 先 `sheet.get_sheets_info`，再 `sheet.get_range_data` |
| 智能表格增强配置不生效 | 使用了普通 Excel，或未先读取现有配置结构 | 确认文件为 `.ksheet`，并先调用对应的 `get/list` 工具查看当前结构 |

---

## 附录

### 错误响应

| 情况 | 响应示例 |
|------|---------|
| 命令不支持 | `{"msg":"core not support","result":"unSupport"}` |
| 内核错误 | `{"errno":-1880935404,"msg":"Invalid request","result":"ExecuteFailed"}` |
| HTTP 状态非 200 | 请求本身失败，检查鉴权信息（Cookie/Origin） |
