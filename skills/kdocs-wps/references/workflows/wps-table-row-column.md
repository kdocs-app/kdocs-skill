# 表格行级与列级格式

> 整行或整列的对齐、垂直对齐、字体

**适用场景**：同一行或同一列使用同一对齐、垂直对齐或字体。取值见对应工具的 example。

**触发词**：整行对齐、整列对齐、行字体、列字体、垂直居中

## 涉及工具

`wps.tables.row_alignment`、`wps.tables.column_alignment`、`wps.tables.row_vertical_alignment`、`wps.tables.column_vertical_alignment`、`wps.tables.row_font`、`wps.tables.column_font`。

调用接口前先执行以下命令获取接口参数及约束：
`kdocs-cli help wps.tables.row_alignment wps.tables.column_alignment wps.tables.row_vertical_alignment wps.tables.column_vertical_alignment wps.tables.row_font wps.tables.column_font`

## 执行流程

1. 整行同一水平对齐：一次 `wps.tables.row_alignment`。整列：一次 `wps.tables.column_alignment`。
2. 整行同一垂直对齐：一次 `wps.tables.row_vertical_alignment`。整列：一次 `wps.tables.column_vertical_alignment`。
3. 整行字体：一次 `wps.tables.row_font`。整列：一次 `wps.tables.column_font`。`font_style` 一次只传一个属性。
4. 写完后对一行或一列 query 一次。
5. 不要逐格调用 `wps.tables.cell_alignment`、`wps.tables.cell_vertical_alignment`、`wps.tables.cell_font`。
6. 不要按单元格列表批量 update。
