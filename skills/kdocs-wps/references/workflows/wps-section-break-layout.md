# 分节、分页与页面版式

> 分页、分节与页面版式

**适用场景**：锚点处分页或分节，调整页边距与纸张（如 A4）、页面边框、文字水印。

- 目录与附件节横向组合 → `references/workflows/wps-combo-section-attachment.md`
**触发词**：分页符、分节、页边距、A4、页面边框、文字水印

## 涉及工具

`wps.texts.search`、`wps.texts.range`、`wps.texts.content`、`wps.breaks.data`、`wps.sections.count`、`wps.sections.page_setup`、`wps.sections.border`、`wps.sections.border_data`、`wps.watermarks.text_italic`、`wps.watermarks.query`。见 `references/wps/` 对应说明。
## 执行流程

### 场景 1：分页符

适用：在标题或锚点段后插入分页符，与后续内容分页。

**执行时由用户指定**：标题/锚点字面。

1. `wps.texts.search` — 锚点字面。
2. `wps.texts.range` — `paragraph_index`（勿先 `wps.texts.format` 扫 P1）。
3. `wps.breaks.data` — `break_type=7`（分页符），scope=paragraphs。
4. `wps.texts.content` query — 下一段含 `\f`（form feed）。
> 执行本场景所列步骤前，先运行 `kdocs-cli help wps.breaks.data wps.texts.content wps.texts.format wps.texts.range wps.texts.search` 获取参数与约束。
### 场景 3：页边距（预设 / 自定义 / A4 组合）

适用：改常规边距、仅上下边距、或同时设 A4 与四边距。

**执行时由用户指定**：预设边距或厘米值、是否含 A4 `PageWidth`/`PageHeight`。

1. `wps.sections.page_setup` query — 当前边距/纸张。
2. **路径 A — 常规四边**：四次 update Top/Bottom=72，Left/Right=90（磅）。
3. **路径 B — 仅上下**：两次 update Top/Bottom（cm×72/2.54）。
4. **路径 C — A4 + 边距**：Orientation=0 → PageWidth/Height → 四边距各一次 update。
5. `wps.sections.page_setup` query — 回读验证。
> 执行本场景所列步骤前，先运行 `kdocs-cli help wps.sections.page_setup` 获取参数与约束。
### 场景 5：文字水印

适用：斜体文字水印。

**执行时由用户指定**：水印文案。

1. `wps.watermarks.text_italic` update — `text`。
2. `wps.watermarks.query` — 回读 count/italic。
> 执行本场景所列步骤前，先运行 `kdocs-cli help wps.watermarks.query wps.watermarks.text_italic` 获取参数与约束。
