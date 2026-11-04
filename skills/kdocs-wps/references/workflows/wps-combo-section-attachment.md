# 分节组合（附件横向）

> 附件节横向、组合分节

**适用场景**：目录与附件等内容之间分节，并将附件节设为横向版式等跨步骤版式组合。

- 多步 breaks + sections 组合；锚点字面、节索引等**执行时由用户指定**
**触发词**：附件节横向、组合分节

## 涉及工具

`wps.sections.count`、`wps.sections.page_setup`、`wps.texts.search`、`wps.breaks.data`。见 `references/wps/` 对应说明。

调用接口前先执行以下命令获取接口参数及约束：
`kdocs-cli help wps.sections.count wps.sections.page_setup wps.texts.search wps.breaks.data`
