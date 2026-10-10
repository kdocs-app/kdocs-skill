# wps.lists.data

#### 功能说明

在线文字文档段落或范围列表属性：支持 查询/设置。查询仅支持按段落（paragraphs/query），返回段落当前列表信息。

**幂等性**：是 — safe

> 标题下多要点圆点：首段 `gallery_type=1` + `template_index=1`，后续段 `is_continue=true` 续接同一列表；写后**一次** query 代表段即可。详见 `references/workflows/wps-lists-and-indent.md`（场景 1）。
> 连续段落数字编号：首段 `gallery_type=2`（编号），后续段 `is_continue=true` 续接 1、2、3…；写后**一次** query 代表段。详见 `references/workflows/wps-lists-and-indent.md`（场景 2）。

#### 支持功能

| 功能简述 | 详情 |
|--------------|------|
| 按段落查询列表信息 | [lists-data/paragraphs_query.md](lists-data/paragraphs_query.md) |
| 按段落设置列表属性 | [lists-data/paragraphs_update.md](lists-data/paragraphs_update.md) |
| 按字符区间设置列表属性 | [lists-data/ranges_update.md](lists-data/ranges_update.md) |
