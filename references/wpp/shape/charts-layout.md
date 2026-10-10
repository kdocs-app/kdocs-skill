# 形状 · 图表版式

> facet: shape.charts

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `wpp.shape.charts.axis` | 形状图表坐标轴：支持 查询/设置 | `slide_id`, `verb` |
| `wpp.shape.charts.chart_type` | 形状图表图表类型：支持 查询/设置 | `slide_id`, `verb` |
| `wpp.shape.charts.gridlines` | 形状图表网格线：支持 查询/设置 | `slide_id`, `verb` |
| `wpp.shape.charts.legend` | 形状图表图例：支持 查询/设置 | `slide_id`, `verb` |
| `wpp.shape.charts.title` | 形状图表标题：支持 查询/设置 | `slide_id`, `verb` |
| `wpp.shape.charts.trendline` | 形状图表趋势线：支持 查询/设置 | `slide_id`, `verb` |

---

## 1. wpp.shape.charts.axis

#### 功能说明

形状图表坐标轴：支持 查询/设置。

**幂等性**：是 — safe

#### 调用示例

查询图表坐标轴：

```json
{
  "file_id": "<FILE_ID>",
  "slide_id": 150995203,
  "verb": "query"
}
```

设置图表坐标轴：

```json
{
  "file_id": "<FILE_ID>",
  "slide_id": 150995203,
  "axis": {
    "axis_type": 1,
    "has_title": true,
    "title": "销售月份",
    "min_auto": true,
    "max_auto": true
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
      "axes": [
        {
          "axis_type": 1,
          "has_title": true,
          "max": 2.5,
          "max_auto": true,
          "min_auto": true,
          "title": "销售月份"
        },
        {
          "axis_type": 2,
          "max": 2.5,
          "max_auto": true,
          "min_auto": true
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

## 2. wpp.shape.charts.chart_type

#### 功能说明

形状图表图表类型：支持 查询/设置。

**幂等性**：是 — safe

#### 调用示例

查询图表类型：

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
      "has_legend": false,
      "has_title": false,
      "shape_id": 4,
      "xl_chart_type": 4
    }
  }
}
```


---

## 3. wpp.shape.charts.gridlines

#### 功能说明

形状图表网格线：支持 查询/设置。

**幂等性**：是 — safe

#### 调用示例

查询图表网格线：

```json
{
  "file_id": "<FILE_ID>",
  "slide_id": 150995203,
  "verb": "query"
}
```

设置图表网格线：

```json
{
  "file_id": "<FILE_ID>",
  "slide_id": 150995203,
  "gridlines": {
    "axis_type": 1,
    "major": true,
    "minor": false
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
      "gridlines": [
        {
          "axis_type": 1
        },
        {
          "axis_type": 2,
          "major": true
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

## 4. wpp.shape.charts.legend

#### 功能说明

形状图表图例：支持 查询/设置。

**幂等性**：是 — safe

#### 调用示例

查询图表图例：

```json
{
  "file_id": "<FILE_ID>",
  "slide_id": 150995203,
  "verb": "query"
}
```

设置图表图例位置：

```json
{
  "file_id": "<FILE_ID>",
  "slide_id": 150995203,
  "verb": "update",
  "shape_id": "<SHAPE_ID>",
  "legend": {
    "position": 3
  }
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
      "has_legend": false,
      "has_title": false,
      "shape_id": 4
    }
  }
}
```


---

## 5. wpp.shape.charts.title

#### 功能说明

形状图表标题：支持 查询/设置。

**幂等性**：是 — safe

#### 调用示例

查询图表标题：

```json
{
  "file_id": "<FILE_ID>",
  "slide_id": 150995203,
  "verb": "query"
}
```

设置图表标题：

```json
{
  "file_id": "<FILE_ID>",
  "slide_id": 150995203,
  "title": {
    "visible": true,
    "text": "季度销售趋势"
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
      "has_legend": false,
      "has_title": true,
      "shape_id": 4,
      "title_text": "季度销售趋势"
    }
  }
}
```


---

## 6. wpp.shape.charts.trendline

#### 功能说明

形状图表趋势线：支持 查询/设置。

**幂等性**：是 — safe

#### 调用示例

查询图表趋势线：

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
      "has_legend": false,
      "has_title": false,
      "shape_id": 4,
      "trendlines": [
        {
          "count": 6,
          "index": 1
        }
      ]
    }
  }
}
```
