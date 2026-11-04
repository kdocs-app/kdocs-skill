# 文本框与形状文案

> 文本框与带文案形状

**适用场景**：在锚点旁插入文本框或简单形状，写入框内文字并设置边框/填充/字形。

- 箭头、组合、置顶 → `references/workflows/wps-shapes-draw-compose.md`
**触发词**：插入文本框、边框样式、圆角矩形、框内文字

## 涉及工具

`wps.texts.search`、`wps.shapes.text_box`、`wps.shapes.text`、`wps.shapes.line_color`、`wps.shapes.line_dash`、`wps.shapes.basic`、`wps.shapes.fill_color`、`wps.shapes.font`、`wps.shapes.list`。
对上表各工具执行 `kdocs-cli help <tool>` 获取参数与约束。
## 执行流程

### 场景 1：锚点旁插入空文本框

适用：在指定字面旁插入空文本框。

**执行时由用户指定**：锚点字面；多命中取首处 begin/end。

1. `wps.texts.search` — 定位锚点。
2. `wps.shapes.text_box` — 传入 begin/end；勿写前 `wps.shapes.count`。
3. `wps.shapes.list` — 确认 type=17（TextBox）。
> 执行本场景所列步骤前，先运行 `kdocs-cli help wps.shapes.count wps.shapes.list wps.shapes.text_box wps.texts.search` 获取参数与约束。
### 场景 3：插入形状并写入框内样式

适用：插入圆角矩形等形状，设填充色与框内文字、字色/加粗等。

**执行时由用户指定**：`shape_type`（如圆角矩形 5）、填充色、文案、`wps.shapes.font` 属性（颜色、加粗等）。

1. `wps.shapes.basic` — `shape_type`。
2. `wps.shapes.fill_color` update（勿 fill query）。
3. `wps.shapes.text` update — 文案。
4. `wps.shapes.font` update — 按用户设 Bold、ColorIndex/RGB 等（可多次写不同属性；勿 font query）。
5. `wps.shapes.list` 回读。
> 执行本场景所列步骤前，先运行 `kdocs-cli help wps.shapes.basic wps.shapes.fill_color wps.shapes.font wps.shapes.list wps.shapes.text` 获取参数与约束。
