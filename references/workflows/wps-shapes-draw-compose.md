# 形状绘制、组合与叠放

> 形状插入、外观、按页删图、组合与叠放

**适用场景**：两锚点间插连线/箭头，新建形状后改填充或线宽，按页删除浮动图，同页多形状组合、置顶或左对齐，多形状居中等距后组合。

- 文本框与框内文案 → `wps-shapes-textbox.md`
- 内嵌图插入与裁剪 → `wps-image-on-page.md`
**触发词**：箭头、连接器、组合、置顶、删图、对齐、线宽、填充

## 涉及工具

`wps.texts.search`、`wps.texts.page_number`、`wps.images.count`、`wps.shapes.basic`、`wps.shapes.list`、`wps.shapes.anchor`、`wps.shapes.fill_color`、`wps.shapes.line_width`、`wps.shapes.data`、`wps.shapes.group`、`wps.shapes.z_order`、`wps.shapes.props`、`wps.shapes.align`、`wps.shapes.distribute`、`wps.shapes.find_by_text`。见 `references/wps/shapes.*`。

## 执行流程

### 场景 1：两锚点之间插入连接器

适用：在两段字面/选区**之间**插入箭头或连线（非段内单点插入）。

**执行时由用户指定**：锚点字面 A、锚点字面 B、`shape_type`（如向下箭头 36、右箭头 33）。

1. `wps.texts.search` — 定位 A 得 `end1`。
2. `wps.texts.search` — 定位 B 得 `begin2`。
3. `wps.shapes.basic` — **同时**传 `begin=end1` 与 `end=begin2`（仅 begin 易 500002）；勿 delete 重试、勿 `wps.texts.range` 探段。
4. `wps.shapes.list` — 确认新增形状；必要时 `wps.shapes.anchor` 介于两锚点段。

### 场景 2：插入形状后设置外观

适用：先插入形状（或已有形状），再改填充色、线宽、线色等视觉属性。

**执行时由用户指定**：可选页码 P 校验、`shape_type`、属性及值（如 `wps.shapes.fill_color` RGB；`wps.shapes.line_width` 磅值）。

1. `wps.shapes.basic` — 插入目标 `shape_type`（若尚未存在则跳过）。
2. 按用户要求 **单次 update**：`wps.shapes.fill_color` 或 `wps.shapes.line_width` 等（勿对同属性 query，常 400100）；写侧 code=0 即 pass。
3. `wps.shapes.list` + 可选 `wps.shapes.anchor` + `wps.texts.page_number` — 确认形状在目标页 P（用户要求按页时）。

### 场景 3：按页删除浮动图片

适用：删除指定页（或页集）上的全部图片；`wps.images.count=0` 时为浮动图。

**执行时由用户指定**：目标页集合（如 {4,7}）或「除某页外」策略。

1. `wps.images.count`：0 → `wps.shapes.list` 筛 type=7/13。
2. 对 list 中每个图片候选 **交错** `wps.shapes.anchor` + `wps.texts.page_number` 各一次；命中目标页则加入删除列表（勿先批量 anchor 再批量 page）。
3. 对每个目标 `wps.shapes.data` delete（**逆序** shape_item）。
4. `wps.shapes.list` 回读 count；勿 delete 后全量重扫 anchor/page（写成功 + count 减少即可）。

### 场景 4：定位并组合 / 叠放 / 页内对齐

适用：同页多个形状：组合、置于顶层、组合形左对齐等。

**执行时由用户指定**：页 P、定位方式、`wps.shapes.group` / `wps.shapes.z_order` / `wps.shapes.props.Left` 等。

**路径 A — 锚点旁组合**：`wps.texts.search` → `wps.texts.page_number` 确认页 P → `wps.shapes.list` 得 `shape_names` → `wps.shapes.group`。

**路径 B — 置顶**：`wps.shapes.list`；对目标页 P 用 anchor+page 或 `wps.texts.search` / `wps.shapes.find_by_text` → **唯一写** `wps.shapes.z_order`（`msoBringToFront`）→ 可选 query 回读 z。

**路径 C — 某页组合形左对齐**：`wps.shapes.list` 后交错 anchor+page 筛页 P → `wps.shapes.props` 设 `Left` 为 min(left)（`wps.shapes.align` 对组合形常无效）；见 `references/wps/shapes.align.md`。

### 场景 5：多形状对齐、等距与组合

适用：多个顶层形状水平居中、等距分布后组合。

**执行时由用户指定**：形状数量 N（常为 3）、`relative_to_page` 是否相对页面。

1. `wps.shapes.list` — 得 N 个顶层 `shape_names`。
2. `wps.shapes.align` — `align_cmd=1`（水平居中），`relative_to_page=true`（按用户）。
3. `wps.shapes.distribute` — `distribute_cmd=0`（水平等距）。
4. `wps.shapes.group` — N 个 shape_names。
5. `wps.shapes.list` 回读新 Group；勿 `wps.shapes.info` 读 Group 位置（常 400100）。
