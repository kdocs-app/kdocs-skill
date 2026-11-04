# 图片插入与页面内调整

> 插图、缩放、裁剪、环绕、浮动定位

**适用场景**：段落后插入图片，按页缩放，裁剪（内嵌或浮动），版式（页宽比例、对齐、环绕、移位）。

- 形状组合/置顶 → `references/workflows/wps-shapes-draw-compose.md`
**触发词**：插图、缩放、裁剪、环绕、浮动定位

## 涉及工具

`wps.texts.search`、`wps.texts.range`、`wps.texts.count`、`wps.texts.page_number`、`wps.images.insert`、`wps.images.count`、`wps.images.list`、`wps.images.locate_paragraph`、`wps.images.resize`、`wps.images.crop`、`wps.images.crop_to_shape`、`wps.images.wrap`、`wps.images.move_to_paragraph`、`wps.images.info`、`wps.sections.page_setup`、`wps.shapes.list`、`wps.shapes.anchor`、`wps.shapes.props`、`wps.shapes.wrap_type`。
对上表各工具执行 `kdocs-cli help <tool>` 获取参数与约束。
## 执行流程

### 场景 1：段落后插入图片

适用：在某标题/段落文字**后**插入示意图（段落级内嵌图）。

**执行时由用户指定**：锚点字面、`position`（通常为 after）、图片 URL（经 upload 链）。

1. `wps.texts.search` — 锚点字面；须唯一命中（或按用户接受多命中策略）。
2. `wps.texts.count` + `wps.texts.range` — 映射 `paragraph_index`（勿线性扫全文）。
3. 本地/占位图：`upload_attachment` → `download_attachment` → `wps.images.insert`（`scope=paragraphs`，`paragraph_index`，`position=after`）；勿用无效占位 URL 直插。
4. `wps.images.locate_paragraph` 或 `wps.images.count` 回读确认。
> 执行本场景所列步骤前，先运行 `kdocs-cli help wps.images.count wps.images.insert wps.images.locate_paragraph wps.texts.count wps.texts.range wps.texts.search` 获取参数与约束。
### 场景 3：按页裁剪与环绕（内嵌 / 浮动）

适用：指定页图片裁成正方形或其他裁剪；或设四周型等环绕。

**执行时由用户指定**：页集、裁剪规则、`wrap_type`（如四周型 0）。

**路径 A — 内嵌图**：`wps.images.count`>0 → `wps.images.crop` / `wps.images.crop_to_shape`（按用户）。

**路径 B — 浮动图**（count=0）：`wps.shapes.list` 筛 type=13/7；对每个候选 **交错** `wps.shapes.anchor`+`wps.texts.page_number`，命中目标页且该页尚未记录则记下 shape_item，**齐备后停止扫描**。
- 裁剪：用 list 的 width/height（勿 `wps.shapes.info`）；对称裁剪写 `wps.shapes.props` 的 Crop* 各一次；Crop* 不支持 query，四键 code=0 即 pass。
- **仅环绕**：`wps.shapes.wrap_type` update；勿 query；写后勿重复调。
> 执行本场景所列步骤前，先运行 `kdocs-cli help wps.images.count wps.images.crop wps.images.crop_to_shape wps.shapes.anchor wps.shapes.list wps.texts.page_number` 获取参数与约束。
