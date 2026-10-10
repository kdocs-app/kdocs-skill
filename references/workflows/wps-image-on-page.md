# 图片插入与页面内调整

> 插图、缩放、裁剪、环绕、浮动定位

**适用场景**：段落后插入图片，按页缩放，裁剪（内嵌或浮动），版式（页宽比例、对齐、环绕、移位）。

- 形状组合/置顶 → `wps-shapes-draw-compose.md`
**触发词**：插图、缩放、裁剪、环绕、浮动定位

## 涉及工具

`wps.texts.search`、`wps.texts.range`、`wps.texts.count`、`wps.texts.page_number`、`wps.images.insert`、`wps.images.count`、`wps.images.list`、`wps.images.locate_paragraph`、`wps.images.resize`、`wps.images.crop`、`wps.images.crop_to_shape`、`wps.images.wrap`、`wps.images.move_to_paragraph`、`wps.images.info`、`wps.sections.page_setup`、`wps.shapes.list`、`wps.shapes.anchor`、`wps.shapes.props`、`wps.shapes.wrap_type`。见 `references/wps/images.*`、`shapes.*`。

## 执行流程

### 场景 1：段落后插入图片

适用：在某标题/段落文字**后**插入示意图（段落级内嵌图）。

**执行时由用户指定**：锚点字面、`position`（通常为 after）、图片 URL（经 upload 链）。

1. `wps.texts.search` — 锚点字面；须唯一命中（或按用户接受多命中策略）。
2. `wps.texts.count` + `wps.texts.range` — 映射 `paragraph_index`（勿线性扫全文）。
3. 本地/占位图：`upload_attachment` → `download_attachment` → `wps.images.insert`（`scope=paragraphs`，`paragraph_index`，`position=after`）；勿用无效占位 URL 直插。
4. `wps.images.locate_paragraph` 或 `wps.images.count` 回读确认。

### 场景 2：按页缩放图片

适用：将某一页（或页集）上全部图片按比例缩放。

**执行时由用户指定**：页 P 或页集、缩放比例（如 50% → 宽高 ×0.5）。

1. `wps.images.list` — index、paragraph_index、width/height。
2. 对每个候选 `wps.texts.page_number`（传 paragraph_index）筛目标页；list 已含 paragraph_index 时优先 page_number，勿逐图 locate_paragraph。
3. 对每个目标图 `wps.images.resize` — 新宽高 = 原尺寸×比例；以 list 最新 index 为准。
4. `wps.images.list` + 代表图 `wps.images.info` 回读（注意栅格吸附，见 `images.resize.md`）。

### 场景 3：按页裁剪与环绕（内嵌 / 浮动）

适用：指定页图片裁成正方形或其他裁剪；或设四周型等环绕。

**执行时由用户指定**：页集、裁剪规则、`wrap_type`（如四周型 0）。

**路径 A — 内嵌图**：`wps.images.count`>0 → `wps.images.crop` / `wps.images.crop_to_shape`（按用户）。

**路径 B — 浮动图**（count=0）：`wps.shapes.list` 筛 type=13/7；对每个候选 **交错** `wps.shapes.anchor`+`wps.texts.page_number`，命中目标页且该页尚未记录则记下 shape_item，**齐备后停止扫描**。
- 裁剪：用 list 的 width/height（勿 `wps.shapes.info`）；对称裁剪写 `wps.shapes.props` 的 Crop* 各一次；Crop* 不支持 query，四键 code=0 即 pass。
- **仅环绕**：`wps.shapes.wrap_type` update；勿 query；写后勿重复调。

### 场景 4：版式：页宽比例 + 对齐 + 环绕 + 移位

适用：单图或浮动图缩至页宽比例、右/左对齐、四周型环绕，或移到某段旁。

**执行时由用户指定**：相对页宽比例 N%、对齐侧、目标段字面、源图所在页 P。

**路径 A — 单内嵌图版式**：`wps.images.list` → `wps.sections.page_setup` → `wps.images.wrap` → `wps.shapes.list` → `wps.shapes.props` 设 Width/Height/Left（勿 wrap 后再 `wps.images.resize`，常 500002）。

**路径 B — 浮动图移位**：count=0 → `wps.shapes.list`；`wps.texts.search`+`wps.texts.range` 得目标段；anchor+page 定位图 → `wps.images.move_to_paragraph` / `wps.shapes.props` + `wps.images.wrap` 按用户落位。
