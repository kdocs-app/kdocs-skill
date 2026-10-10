# 文本框与形状文案

> 文本框与带文案形状

**适用场景**：在锚点旁插入文本框或简单形状，写入框内文字并设置边框/填充/字形。

- 箭头、组合、置顶 → `wps-shapes-draw-compose.md`
**触发词**：插入文本框、边框样式、圆角矩形、框内文字

## 涉及工具

`wps.texts.search`、`wps.shapes.text_box`、`wps.shapes.text`、`wps.shapes.line_color`、`wps.shapes.line_dash`、`wps.shapes.basic`、`wps.shapes.fill_color`、`wps.shapes.font`、`wps.shapes.list`。见 `references/wps/shapes.*`。

## 执行流程

### 场景 1：锚点旁插入空文本框

适用：在指定字面旁插入空文本框。

**执行时由用户指定**：锚点字面；多命中取首处 begin/end。

1. `wps.texts.search` — 定位锚点。
2. `wps.shapes.text_box` — 传入 begin/end；勿写前 `wps.shapes.count`。
3. `wps.shapes.list` — 确认 type=17（TextBox）。

### 场景 2：文本框 + 文案 + 边框样式

适用：插入文本框、写入文案，并设边框颜色与线型（实线/虚线）。

**执行时由用户指定**：锚点或 `<document_selection>` 字面、框内文案、边框 RGB、`dash_style`（虚线时）。

1. `wps.texts.search` → `wps.shapes.text_box`（begin/end）。
2. `wps.shapes.text` update — 文案。
3. `wps.shapes.line_color` update — 边框色（勿 query）。
4. 若虚线：`wps.shapes.line_dash` update — 用 text_box 返回的 **source_index**（= shape_item），`dash_style` 按文档约定。
5. `wps.shapes.list` 回读；勿额外 `wps.shapes.text` query。

### 场景 3：插入形状并写入框内样式

适用：插入圆角矩形等形状，设填充色与框内文字、字色/加粗等。

**执行时由用户指定**：`shape_type`（如圆角矩形 5）、填充色、文案、`wps.shapes.font` 属性（颜色、加粗等）。

1. `wps.shapes.basic` — `shape_type`。
2. `wps.shapes.fill_color` update（勿 fill query）。
3. `wps.shapes.text` update — 文案。
4. `wps.shapes.font` update — 按用户设 Bold、ColorIndex/RGB 等（可多次写不同属性；勿 font query）。
5. `wps.shapes.list` 回读。
