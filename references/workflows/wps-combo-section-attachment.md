# 分节组合（附件横向）

> 附件节横向、组合分节

**适用场景**：目录与附件等内容之间分节，并将附件节设为横向版式等跨步骤版式组合。

- 多步 breaks + sections 组合；锚点字面、节索引等**执行时由用户指定**
**触发词**：附件节横向、组合分节

## 涉及工具

`wps.sections.count`、`wps.sections.page_setup`、`wps.texts.search`、`wps.breaks.data`。见 `references/wps/` 对应说明。

## 执行流程

### 场景 1：组合分节+附件节横向

适用：用户要求在「**附件**」前插入分节符，并将**附件部分**改为**横向**页面（如「在附件前插入分节符，附件部分改成横向页面」）。

1. `wps.sections.count` query 一次 — 基线节数。
2. `wps.texts.search` 一次 — 锚点字面（如「附件」）；多命中取用户指定处。
3. `wps.breaks.data` 一次 — `break_type=2`（下一页分节）at search begin。
4. `wps.sections.count` query 一次 — 验证 count+1。
5. `wps.sections.page_setup` update 一次 — 新节 `section_index=2`（或 count 值），`Orientation=1`（横向）。
6. `wps.sections.page_setup` query **两次** — 回读 section 1 orientation=0、section 2 orientation=1。

> 。不要改 section 1 为横向。先读 `references/wps/breaks.data.md`、`references/wps/sections.page_setup.md`。
