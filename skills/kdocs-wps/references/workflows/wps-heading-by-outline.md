# 标题段样式（大纲级别）

> 标题段（大纲级别）批量样式

**适用场景**：已带 outline_level 1–9 的标题段：改色、居中、字符间距、主题强调色等。

- 按关键词离散改色/高亮 → `references/workflows/wps-multi-target-search-style.md`
**触发词**：标题改色、标题居中、标题字间距、主题色

## 涉及工具

`wps.texts.structure`、`wps.texts.format`、`wps.texts.font`、`wps.texts.font_color`、`wps.texts.theme_color`、`wps.texts.alignment`、`wps.texts.range`、`wps.texts.count`。
对上表各工具执行 `kdocs-cli help <tool>` 获取参数与约束。
## 执行流程

### 场景 1：定位全部标题段

适用：需要批量处理 outline_level 1–9 的段落。

**执行时由用户指定**：无（全文标题）；或 structure 每批 ≤100 `paragraph_indexes`。

1. **优先** `wps.texts.structure` 批量筛 outline_level 1–9；400100 则 **勿重试** `wps.texts.count`。
2. 降级：`wps.texts.format` 查 P1；仅当 P1 非标题才顺序查 P2…，**找到标题并处理即停**。
3. 勿仅用 `wps.texts.count` 探测全文段数。
> 执行本场景所列步骤前，先运行 `kdocs-cli help wps.texts.count wps.texts.format wps.texts.structure` 获取参数与约束。
