# 形状 · 图表数据

> facet: shape.charts

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `wpp.shape.charts.data` | 形状图表插入空白页：支持 插入/查询/设置 | `height`, `left`, `slide_id`, `top`, `verb`, `width` |
| `wpp.shape.charts.data_labels` | 形状图表数据标签：支持 查询/设置 | `slide_id`, `verb` |
| `wpp.shape.charts.series_style` | 设置形状图表系列样式 | `slide_id` |

---

## 1. wpp.shape.charts.data

#### 功能说明

形状图表插入空白页：支持 插入/查询/设置。

**幂等性**：是 — safe

#### 调用示例

插入柱状图图表：

```json
{
  "file_id": "<FILE_ID>",
  "slide_id": 150995203,
  "left": 300,
  "top": 100,
  "width": 400,
  "height": 300,
  "chart_type": "COLUMN",
  "verb": "insert"
}
```

查询图表数据：

```json
{
  "file_id": "<FILE_ID>",
  "slide_id": 150995203,
  "verb": "query"
}
```

#### 参数说明

- `axis` (object, 可选): 坐标轴对象：{axis_type, has_title, title, min, max, min_auto, max_auto}
- `categories` (array, 可选): 分类（X 轴标签）
- `chart_type` (string, 可选): 图表类型：COLUMN / BAR / LINE / PIE / AREA / SCATTER / OTHER
- `data_labels` (object, 可选): 数据标签对象：{type}
- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `gridlines` (object, 可选): 网格线对象：{axis_type, major, minor}
- `height` (number, 必填): 高度
- `left` (number, 必填): 左边距
- `legend` (object, 可选): 图例对象：{visible, position}
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `series` (array, 可选): 系列数组：[{name, values[]}]
- `series_style` (object, 可选): 系列样式对象：{series_index, name, fill_rgb}
- `shape_id` (number, 可选): 形状 ID
- `slide_id` (number, 必填): 幻灯片 ID
- `title` (object, 可选): 标题对象：{visible, text}
- `top` (number, 必填): 上边距
- `trendline` (object, 可选): 趋势线对象：{series_index, type, display_equation, display_r_squared, name}
- `url` (string, 可选): 文档 URL；与 link_id、file_id 三选一
- `verb` (string, 必填): HTTP 动作：insert（插入） / query（查询） / update（设置）。可选值：`insert` / `query` / `update`
- `width` (number, 必填): 宽度
- `xl_chart_type` (number, 可选): XlChartType 整型（chart_type=OTHER 时直传）

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "data": {
      "has_legend": false,
      "has_title": false,
      "series": [
        {
          "fill_rgb": 12419407,
          "formula": "=SERIES(Sheet1!$B$1,Sheet1!$A$2:$A$4,Sheet1!$B$2:$B$4,1)",
          "index": 1
        }
      ],
      "shape_id": 4
    }
  }
}
```


---

## 2. wpp.shape.charts.data_labels

#### 功能说明

形状图表数据标签：支持 查询/设置。

**幂等性**：是 — safe

#### 调用示例

查询图表数据标签：

```json
{
  "file_id": "<FILE_ID>",
  "slide_id": 150995203,
  "verb": "query"
}
```

设置图表数据标签：

```json
{
  "file_id": "<FILE_ID>",
  "slide_id": 150995203,
  "data_labels": {
    "type": 1
  },
  "verb": "update",
  "shape_id": 4
}
```

#### 参数说明

- `axis` (object, 可选): 坐标轴对象：{axis_type, has_title, title, min, max, min_auto, max_auto}
- `categories` (array, 可选): 分类（X 轴标签）
- `chart_type` (string, 可选): 图表类型：COLUMN / BAR / LINE / PIE / AREA / SCATTER / OTHER
- `data_labels` (object, 可选): 数据标签对象：{type}
- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `gridlines` (object, 可选): 网格线对象：{axis_type, major, minor}
- `legend` (object, 可选): 图例对象：{visible, position}
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `series` (array, 可选): 系列数组：[{name, values[]}]
- `series_style` (object, 可选): 系列样式对象：{series_index, name, fill_rgb}
- `shape_id` (number, 可选): 形状 ID
- `slide_id` (number, 必填): 幻灯片 ID
- `title` (object, 可选): 标题对象：{visible, text}
- `trendline` (object, 可选): 趋势线对象：{series_index, type, display_equation, display_r_squared, name}
- `url` (string, 可选): 文档 URL；与 link_id、file_id 三选一
- `verb` (string, 必填): HTTP 动作：query（查询） / update（设置）。可选值：`query` / `update`
- `xl_chart_type` (number, 可选): XlChartType 整型（chart_type=OTHER 时直传）

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "data": {
      "data_labels": [
        {
          "index": 1
        }
      ],
      "has_legend": false,
      "has_title": false,
      "shape_id": 4
    }
  }
}
```


---

## 3. wpp.shape.charts.series_style

#### 功能说明

设置形状图表系列样式。

**幂等性**：是 — safe

#### 参数说明

- `axis` (object, 可选): 坐标轴对象：{axis_type, has_title, title, min, max, min_auto, max_auto}
- `categories` (array, 可选): 分类（X 轴标签）
- `chart_type` (string, 可选): 图表类型：COLUMN / BAR / LINE / PIE / AREA / SCATTER / OTHER
- `data_labels` (object, 可选): 数据标签对象：{type}
- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `gridlines` (object, 可选): 网格线对象：{axis_type, major, minor}
- `legend` (object, 可选): 图例对象：{visible, position}
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `series` (array, 可选): 系列数组：[{name, values[]}]
- `series_style` (object, 可选): 系列样式对象：{series_index, name, fill_rgb}；fill_rgb 为 int32 OLE BGR（如 16737792）或 #RRGGBB 十六进制字符串
- `shape_id` (number, 可选): 形状 ID
- `slide_id` (number, 必填): 幻灯片 ID
- `title` (object, 可选): 标题对象：{visible, text}
- `trendline` (object, 可选): 趋势线对象：{series_index, type, display_equation, display_r_squared, name}
- `url` (string, 可选): 文档 URL；与 link_id、file_id 三选一
- `xl_chart_type` (number, 可选): XlChartType 整型（chart_type=OTHER 时直传）

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {}
}
```
