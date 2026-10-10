# 演示文稿（pptx / wpp）工具完整参考文档

本文件包含金山文档 Skill 中**演示文稿**相关工具的完整 API 说明、详细调用示例、参数说明和返回值说明。

**适用范围**：本文说明在线演示（PPT）相关的 **`wpp.*`** 工具；按能力域在下表查阅参数与调用示例。

---

## 通用说明

### 演示文稿工具概述

- 若只是读取幻灯片正文/结构，优先使用 `read_file`（`format=kdc`），不必逐个调用 `wpp.*` 做只读查询。

下表按能力域列出参考文档（演示文稿、幻灯片、形状与图表、母版、导出、字体与配色等）。

| 能力域 | 参考文档 |
|--------|--------------------------------|
| 演示文稿 · 文件属性 | [presentation/properties-file.md](wpp/presentation/properties-file.md) |
| 演示文稿 · 显示与主题查询 | [presentation/properties-display.md](wpp/presentation/properties-display.md) |
| 演示文稿 · 节 | [presentation/sections.md](wpp/presentation/sections.md) |
| 演示文稿 · 标签 | [presentation/tags.md](wpp/presentation/tags.md) |
| 幻灯片 · 页序与增删 | [slide/items-structure.md](wpp/slide/items-structure.md) |
| 幻灯片 · 外部 PPTX 导入 | [slide.md](wpp/slide.md) |
| 幻灯片 · 单页属性 | [slide/items-slide.md](wpp/slide/items-slide.md) |
| 幻灯片 · 备注 | [slide/notes.md](wpp/slide/notes.md) |
| 幻灯片 · 标签 | [slide/tags.md](wpp/slide/tags.md) |
| 幻灯片 · 批注 | [slide/comments.md](wpp/slide/comments.md) |
| 幻灯片 · 动画 | [slide/animations.md](wpp/slide/animations.md) |
| 形状 · 列举与基础 | [shape/items-core.md](wpp/shape/items-core.md) |
| 形状 · 插入 | [shape/items-insert.md](wpp/shape/items-insert.md) |
| 形状 · 组合与几何 | [shape/items-group.md](wpp/shape/items-group.md) |
| 形状 · 文本 | [shape/texts.md](wpp/shape/texts.md) |
| 形状 · 表格 | [shape/tables.md](wpp/shape/tables.md) |
| 形状 · 图表数据 | [shape/charts-data.md](wpp/shape/charts-data.md) |
| 形状 · 图表版式 | [shape/charts-layout.md](wpp/shape/charts-layout.md) |
| 形状 · 标签 | [shape/tags.md](wpp/shape/tags.md) |
| 形状 · Web 形状 | [shape/webshapes.md](wpp/shape/webshapes.md) |
| 母版 · 主题与版式 | [master/properties-theme.md](wpp/master/properties-theme.md) |
| 母版 · 内容与动效 | [master/properties-content.md](wpp/master/properties-content.md) |
| 主题 · 预设字体与配色 | [theme.md](wpp/theme.md) |
| 导出 | [export.md](wpp/export.md) |

### 使用场景

| 场景 | 说明 |
|------|------|
| 工作汇报 / 对外宣讲 | 改版式与内容、批注协作、导出 PDF / 图片 |
| 培训课件 | 页序与版式、动画与备注、形状与图表 |
| 模板与品牌统一 | 母版/主题查询、全文或单页字体与配色 |
| 合并外部稿 | 从其他 PPTX 导入指定页 |

### 文档定位

- 除创建空白演示 / 纯任务查询外，定位参数 `url` / `link_id` / `file_id` **三选一**。

## 附录

### 错误响应

| 情况 | 说明或示例 |
|------|------------|
| 未登录 / 无权限 | 返回业务错误码或 `result` 非 `ok`，`msg` 含可读原因 |
| 内核执行失败 | 类似 `detail.res[].code` 非 `0`，或 `result` 为 `ExecuteFailed` 等（以实际为准） |
| HTTP 非 200 | 请求失败，检查鉴权（Cookie、Origin 等） |

### 响应格式

若返回带外层 `code` / `msg` / `data`，业务结果以 `data` 内、各工具「返回值说明」中的 JSON 为准。
