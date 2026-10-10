# 仪表盘

## 1. dbsheet.dashboard_copy

#### 功能说明

复制仪表盘

#### 调用约束

- **前置检查**：`dashboard_id` 来自 `dbsheet.dashboard_list`。
- **后置验证**：dashboard_list 确认副本已创建

**幂等性**：否 — 重复调用可能产生多个副本，先确认是否已成功

#### 调用示例

复制：

```json
{
  "file_id": "string",
  "dashboard_id": 2,
  "body": {
    "name": "副本-仪表盘"
  }
}
```

#### 参数说明

- `url` (string, 三选一必填: `url` / `link_id` / `file_id`): 文档 URL
- `link_id` (string, 三选一必填: `url` / `link_id` / `file_id`): 分享链接 ID
- `file_id` (string, 三选一必填: `url` / `link_id` / `file_id`): 文件 ID
- `dashboard_id` (integer, 必填): 源仪表盘 ID
- `body` (object, 必填): JSON 请求体，须含 name（新仪表盘名称）

**body 根级必填**

| 字段 | 类型 | 说明 |
|------|------|------|
| `name` | string | 新仪表盘名称 |

其它可选字段见 copy-dashboard 文档。

#### 返回值说明

```json
{
  "result": "ok",
  "detail": {}
}

```

| 字段 | 类型 | 说明 |
|------|------|------|
| `result` | string | ok 表示成功 |
| `detail` | object | 新仪表盘 id 等 |


---

## 2. dbsheet.dashboard_list

#### 功能说明

**必填 query**：无。

#### 调用示例

列出仪表盘：

```json
{
  "file_id": "string"
}
```

#### 参数说明

- `url` (string, 三选一必填: `url` / `link_id` / `file_id`): 文档 URL
- `link_id` (string, 三选一必填: `url` / `link_id` / `file_id`): 分享链接 ID
- `file_id` (string, 三选一必填: `url` / `link_id` / `file_id`): 文件 ID

#### 返回值说明

```json
{
  "result": "ok",
  "detail": {}
}

```

| 字段 | 类型 | 说明 |
|------|------|------|
| `result` | string | ok 表示成功 |
| `detail` | object | dashboards 数组 |


---

## 3. dbsheet.dashboard_create

#### 功能说明

在多维表格文档中创建仪表盘（看板）。返回 `detail.sheet.id` 即 dashboard_id，后续所有仪表盘工具
（dashboard_update / dashboard_component_* / dashboard_chart_*）都用它定位仪表盘。
注意 dashboard_id 是仪表盘的 sheet id（sheetType=xlDbDashBoardSheet），不要与数据表 sheet_id 混用。

#### 调用约束

- **后置验证**：dashboard_chart_list 确认仪表盘可访问

**幂等性**：否 — 重复调用会创建多个仪表盘，先确认是否已成功

#### 调用示例

创建销售看板：

```json
{
  "file_id": "string",
  "name": "销售看板",
  "icon": "📊"
}
```

#### 参数说明

- `url` (string, 三选一必填: `url` / `link_id` / `file_id`): 文档 URL
- `link_id` (string, 三选一必填: `url` / `link_id` / `file_id`): 分享链接 ID
- `file_id` (string, 三选一必填: `url` / `link_id` / `file_id`): 文件 ID
- `name` (string, 必填): 仪表盘名称
- `description` (string, 可选): 仪表盘说明
- `icon` (string, 可选): 图标（emoji）

#### 返回值说明

```json
{
  "detail": {
    "sheet": {
      "id": 2,
      "name": "仪表盘",
      "description": "仪表盘说明",
      "icon": "⌚",
      "sheetType": "xlDbDashBoardSheet"
    }
  },
  "result": "ok"
}

```

| 字段 | 类型 | 说明 |
|------|------|------|
| `detail.sheet.id` | integer | 仪表盘 ID（dashboard_id），后续工具的定位参数 |
| `detail.sheet.name` | string | 仪表盘名称 |
| `detail.sheet.sheetType` | string | 固定 xlDbDashBoardSheet |
| `result` | string | ok 表示成功 |


---

## 4. dbsheet.dashboard_update

#### 功能说明

更新仪表盘的名称、描述或图标。三个字段均为选填，至少传一个。
dashboard_id 来自 `dbsheet.dashboard_create` 返回的 `detail.sheet.id` 或 `dbsheet.dashboard_list`。

**幂等性**：是

#### 调用示例

重命名仪表盘：

```json
{
  "file_id": "string",
  "dashboard_id": 2,
  "name": "经营看板"
}
```

#### 参数说明

- `url` (string, 三选一必填: `url` / `link_id` / `file_id`): 文档 URL
- `link_id` (string, 三选一必填: `url` / `link_id` / `file_id`): 分享链接 ID
- `file_id` (string, 三选一必填: `url` / `link_id` / `file_id`): 文件 ID
- `dashboard_id` (integer, 必填): 仪表盘 ID（detail.sheet.id，非数据表 sheet_id）
- `name` (string, 可选): 新名称
- `description` (string, 可选): 新说明
- `icon` (string, 可选): 图标（emoji）

#### 返回值说明

```json
{
  "detail": {
    "sheet": { "id": 2, "name": "仪表盘2", "sheetType": "xlDbDashBoardSheet" }
  },
  "result": "ok"
}

```

| 字段 | 类型 | 说明 |
|------|------|------|
| `detail.sheet.id` | integer | 仪表盘 ID |
| `detail.sheet.name` | string | 更新后名称 |
| `result` | string | ok 表示成功 |


---

## 5. dbsheet.dashboard_delete

#### 功能说明

删除指定仪表盘（含其中所有图表组件），**不可恢复**。
dashboard_id 来自 `dbsheet.dashboard_list`；如只想清空图表、保留仪表盘，改用 `dbsheet.dashboard_clear`。

#### 调用约束

- **前置检查**：`dashboard_id` 来自 `dbsheet.dashboard_list`，核对名称后再删；不得自行捏造 ID。
- **用户确认**：删除仪表盘不可恢复，必须向用户确认仪表盘名称和 ID

**幂等性**：是

#### 调用示例

删除仪表盘：

```json
{
  "file_id": "string",
  "dashboard_id": 6
}
```

#### 参数说明

- `url` (string, 三选一必填: `url` / `link_id` / `file_id`): 文档 URL
- `link_id` (string, 三选一必填: `url` / `link_id` / `file_id`): 分享链接 ID
- `file_id` (string, 三选一必填: `url` / `link_id` / `file_id`): 文件 ID
- `dashboard_id` (integer, 必填): 要删除的仪表盘 ID

#### 返回值说明

```json
{
  "detail": { "sheet": { "id": 6 } },
  "result": "ok"
}

```

| 字段 | 类型 | 说明 |
|------|------|------|
| `detail.sheet.id` | integer | 被删除的仪表盘 ID |
| `result` | string | ok 表示成功 |


---

## 6. dbsheet.dashboard_clear

#### 功能说明

删除仪表盘中**所有**图表组件，仪表盘本身保留，**不可恢复**。
只想删单个组件用 `dbsheet.dashboard_component_delete`。

#### 调用约束

- **前置检查**：可先 `dbsheet.dashboard_component_list` 展示将被删除的组件清单。
- **用户确认**：清空所有图表不可恢复，必须向用户确认

**幂等性**：是

#### 调用示例

清空仪表盘内所有图表：

```json
{
  "file_id": "string",
  "dashboard_id": 6
}
```

#### 参数说明

- `url` (string, 三选一必填: `url` / `link_id` / `file_id`): 文档 URL
- `link_id` (string, 三选一必填: `url` / `link_id` / `file_id`): 分享链接 ID
- `file_id` (string, 三选一必填: `url` / `link_id` / `file_id`): 文件 ID
- `dashboard_id` (integer, 必填): 要清空的仪表盘 ID

#### 返回值说明

```json
{
  "detail": { "sheet": { "id": 6 } },
  "result": "ok"
}

```

| 字段 | 类型 | 说明 |
|------|------|------|
| `detail.sheet.id` | integer | 被清空的仪表盘 ID |
| `result` | string | ok 表示成功 |


---

## 7. dbsheet.dashboard_component_list

#### 功能说明

查询仪表盘所有部件（WebExtension），目前仅返回 db 图表。
返回的 `webExtensionKey` 是组件增删改的定位参数；`statisticsModule` 是组件当前生效的统计配置；
`properties` 是组件的前端属性表（样式/标题等），**改样式前必须先读它取现值**。

**幂等性**：是

#### 调用示例

列出仪表盘组件：

```json
{
  "file_id": "string",
  "dashboard_id": 6
}
```

#### 参数说明

- `url` (string, 三选一必填: `url` / `link_id` / `file_id`): 文档 URL
- `link_id` (string, 三选一必填: `url` / `link_id` / `file_id`): 分享链接 ID
- `file_id` (string, 三选一必填: `url` / `link_id` / `file_id`): 文件 ID
- `dashboard_id` (integer, 必填): 仪表盘 ID

#### 返回值说明

```json
{
  "detail": {
    "webExtensions": [
      {
        "webExtensionKey": "{E65A9EA9-D0D0-451D-988F-F46B1FD20EAD}",
        "webExtensionType": "DbDataSource",
        "dataSourceType": "dbTable",
        "statisticsModule": {
          "id": "B",
          "dataSourceSheetId": 1,
          "dimensionFieldId": "D",
          "groupFieldId": "A",
          "sortByRow": true,
          "ascending": true,
          "onlyCountDimension": false,
          "filter": {
            "conditions": [
              { "fieldId": "D", "criteria": { "op": "Intersected", "values": [ { "type": "Date", "year": 2026, "mon": 9, "mday": 22 } ] } }
            ],
            "mode": "And"
          }
        },
        "properties": {
          "chart-info": "{\"layout\":{\"x\":0,\"y\":0,\"w\":4,\"h\":3},\"type\":\"BasicBar\",\"colors\":{\"name\":\"default\",\"value\":[\"hsl(218,100%,58%)\"],\"version\":\"1\"},\"splitLine\":true,\"stackType\":\"noStack\",\"showLabel\":true,\"showLegend\":true,\"legendPosition\":\"bottom\"}",
          "allLabelStyle": "{\"labelContent\":[\"value\"],\"barLabelPosition\":\"inside\",\"fontStyle\":{\"fontSize\":30,\"bold\":false,\"italic\":false}}",
          "legendStyle": "{\"legendPosition\":\"bottom\",\"fontStyle\":{\"fontSize\":12,\"bold\":true,\"italic\":false}}",
          "yAxisStyle": "{\"show\":true,\"leftYAxis\":{\"unit\":1,\"format\":{\"type\":0,\"config\":{}},\"showLabel\":true,\"showAxisLine\":false}}",
          "ext-base-name": "月度销售额",
          "cardBgColor": "#EFB310",
          "cardBgOpacity": "100",
          "cardBgBlur": "0"
        }
      }
    ]
  },
  "result": "ok"
}

```

| 字段 | 类型 | 说明 |
|------|------|------|
| `detail.webExtensions[].webExtensionKey` | string | 部件 key（{GUID}），组件 update/delete 的入参 |
| `detail.webExtensions[].webExtensionType` | string | DbDataSource=图表组件 |
| `detail.webExtensions[].dataSourceType` | string | dbTable=数据源为数据表 |
| `detail.webExtensions[].statisticsModule` | object | 统计配置（维度/分组/排序/筛选）。**按需出现**：未设置的字段不返回，如设了筛选才返回 filter |
| `detail.webExtensions[].properties` | object | 前端属性表 Map<string,string>：chart-info（图表本体 JSON 串）、allLabelStyle（数据标签）、legendStyle（图例）、yAxisStyle（纵轴）、ext-base-name（标题）、cardBgColor/cardBgOpacity/cardBgBlur（背景色）等。key 对照表见 dbsheet.dashboard_component_update |
| `detail.webExtensions[].enableMultiDataSource` | boolean | 是否启用多数据源 |
| `detail.webExtensions[].multiStatisticsModules` | array | 多数据源时的统计配置数组（结构同 statisticsModule） |
| `detail.webExtensions[].statisticsModule.filter` | object | 筛选条件 {conditions:[{fieldId, criteria:{op, values}}], mode:"And"|"Or"} |
| `result` | string | ok 表示成功 |

**读-改-写样式（必须按此顺序）**

`properties` 里的独立属性（`chart-info` / `allLabelStyle` / `yAxisStyle` / `legendStyle`）是**整串 JSON**：

1. 本工具读出 `properties["<属性名>"]`；
2. 解析 JSON、只改目标子键；
3. 序列化回字符串（需转义），用 `dbsheet.dashboard_component_update` 的 `property_modify_list` 整串回写。

只传子键会丢掉同属性内的其它子键（如只传 `fontStyle` 会清掉 `labelContent`/`barLabelPosition`）。

**筛选/统计核对**：`statisticsModule` 只返回已设置的字段；筛选数值变化用 `dbsheet.dashboard_chart_data` 验证。

**不可回读的属性**：系列颜色（`chartStyle`）、`statisticSettingAddList` 写后读不到。
⚠️ 明细数据显示的字段是**面板专属**设置（公开 API 读写均不可用，见 `dashboard_component_update` 陷阱 3）：实测在面板上改过它之后，本工具返回的组件载荷**逐字节一致**（没有任何明细字段相关的键），别指望从这里核对它。


---

## 8. dbsheet.dashboard_component_create

#### 功能说明

在仪表盘中创建图表组件（WebExtension）。批量创建多个组件时**逐个调用**本工具，用 `chart_info.layout`
自行排布（12 列网格，典型 `w=4,h=3`；一行放 3 个 w=4 图表）。
创建前必须先 `dbsheet.get_schema` 拿到数据源表 ID（dataSourceSheetId）与字段 ID（fieldId），
禁止凭字段名捏造 fieldId。创建后用 `dbsheet.dashboard_chart_data` 回读统计结果核对数值。

#### 调用约束

- **前置检查**：`dashboard_id` 来自 `dbsheet.dashboard_create`/`dashboard_list`；`dataSourceSheetId` 与 `fieldId` 来自 `dbsheet.get_schema`，禁止捏造。
- **后置验证**：`dbsheet.dashboard_component_list` 或 `dashboard_chart_list` 确认组件已落盘；数值用 `dashboard_chart_data` 核对

**幂等性**：否 — 重复调用会产生多个重复组件，先 dashboard_component_list 确认

> statisticsModuleSetting 与 chart_info 内的 fieldId 都是字段 ID（如 "B"、"E"），不是字段名。
> 图表类型为 Value（统计数字）/ Progress（进度图）时 statisticsModuleSetting 不传 dimensionFieldId（进度图在「数值 + 自定义值」时数据源仅用于明细）。

#### 调用示例

创建月度销售额柱状图：

```json
{
  "file_id": "string",
  "dashboard_id": 6,
  "statistics_module_setting": {
    "dataSourceSheetId": 1,
    "dimensionFieldId": "E",
    "onlyCountDimension": false,
    "statisticSettingAddList": [
      {
        "fieldId": "B",
        "statisticOptions": "sum"
      }
    ]
  },
  "chart_info": {
    "layout": {
      "x": 4,
      "y": 0,
      "w": 4,
      "h": 3,
      "minW": 2,
      "minH": 2
    },
    "name": "月度销售额",
    "type": "BasicBar",
    "showLabel": true,
    "showLegend": true
  }
}
```

创建总销售额指标卡：

```json
{
  "file_id": "string",
  "dashboard_id": 6,
  "statistics_module_setting": {
    "dataSourceSheetId": 1,
    "onlyCountDimension": true
  },
  "chart_info": {
    "layout": {
      "x": 0,
      "y": 0,
      "w": 4,
      "h": 3,
      "minW": 2,
      "minH": 2
    },
    "name": "总销售额",
    "type": "Value"
  }
}
```

#### 参数说明

- `url` (string, 三选一必填: `url` / `link_id` / `file_id`): 文档 URL
- `link_id` (string, 三选一必填: `url` / `link_id` / `file_id`): 分享链接 ID
- `file_id` (string, 三选一必填: `url` / `link_id` / `file_id`): 文件 ID
- `dashboard_id` (integer, 必填): 仪表盘 ID
- `web_extension_type` (string, 可选): 部件类型，图表类组件固定 DbDataSource；默认值：`DbDataSource`
- `data_source_type` (string, 可选): 数据源类型，数据源为数据表时固定 dbTable；默认值：`dbTable`
- `statistics_module_setting` (object, 可选): 统计配置（附录 A：维度/分组/统计字段/筛选/显示范围）
- `chart_info` (object, 可选): 图表前端属性（附录 B）：type/name/layout/showLabel/showLegend/pieType/stackType/lineEffect/colors；自动包装为 chart-info property
- `property_modify_list` (array, 可选): 高级：前端属性整包 [{key, value}]，value 为 JSON 字符串；与 chart_info 二选一
- `plugin_config` (object, 可选): 插件类组件配置（预留；插件组件暂不支持）

**附录 A：statisticsModuleSetting 常用字段**

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `dataSourceSheetId` | integer | 是 | 数据源数据表 ID（来自 `get_schema`） |
| `dimensionFieldId` | string | 否 | 维度字段 ID；图表类型为**统计数字（Value）时不传** |
| `dimensionSplitMultiple` | boolean | 否 | 维度多选项是否拆分统计，默认 false |
| `onlyCountDimension` | boolean | 否 | 是否仅统计记录总数，默认 true |
| `ascending` | boolean | 否 | 排序方式：true 正序 / false 倒序 |
| `sortByRow` | boolean | 否 | 排序依据：true 横轴 / false 纵轴 |
| `groupConditionFieldId` | string | 否 | 分组字段 ID |
| `groupUnit` | string | 否 | 分组类型，支持 `Text` / `Day` / `Month` / `Year` |
| `groupLimitNum` / `statsLimitNum` | integer | 否 | 分组/统计字段数量上限，默认 2 / 10 |
| `onlyTopNDimensions` | integer | 否 | 显示范围：TopN，**只有柱状图 / 条形图有**，面板取值 1~20（越界会被夹到 1/20），`0` / 不传 = 全部数据。⚠️ **实测公开 API 写不进**（柱状图实测：写 `5`、与 `retainEqualNthDimension:true` 同写、先改排序依据再写，以及 `0` / `1` / `20` / `21`，全部返回成功但**回读不到、组件无变化**；`""` / `null` 报 `400100`）→ 显示范围只能在面板改 |
| `retainEqualNthDimension` | boolean | 否 | TopN 是否包含并列数据（⚠️ 与 `onlyTopNDimensions` 同：实测写不进，返回成功但回读不到）|
| `statisticSettingAddList` | array | 否 | 统计字段 `[{fieldId, statisticOptions, plotType?, yAxisIndex?}]`，statisticOptions 枚举见附录 C；**柱线图（组合图）**用 `plotType`（`Bar`/`Line`）决定该系列画柱还是画线、`yAxisIndex`（`0` 左轴 / `1` 右轴）决定归属轴（实测生效） |
| `filterCriteriasAddList` | array | 否 | 筛选 `[{fieldId, criteria:{op, values}}]`；`op` 见 `dashboard_component_update` 的 param_detail；`values[].type` 仅支持 `Text` / `Date`（`{year,mon,mday}`，`mon` 1-based）/ `DynamicSimple`（`{dynamicType}`） |
| `dataDetailVisibleFields` | object | 否 | ⚠️ **无效键（别写）**：明细数据显示的字段是**面板专属**设置，公开 API 既写不了也读不到（实测传对象 / 字符串 / 数字一律返回成功但组件无任何变化；旧文档名 `visibleFields` 同样无效） |
| `separators` | object | 否 | 词云分词符 `{builtinSeparators, userDefinedSeparators}` |
| `chartStyle` | object | 否 | `{chartElementType:"Series", identifier:"{}", style:"{\"color\":\"#1A9F5A\"}"}`，**style 为 JSON 字符串需转义** |

**附录 B：chart_info（图表前端属性）**

```json
{
  "layout": {"x": 0, "y": 0, "w": 4, "h": 3, "minW": 2, "minH": 2},
  "name": "图表名称",
  "type": "BasicBar",
  "showLabel": true,
  "showLegend": true,
  "stackType": "noStack",
  "lineEffect": "smooth",
  "pieType": "pie",
  "colors": {"name": "c1", "value": ["#3DC5BA", "#417FF9"]}
}
```

| 字段 | 适用图表 | 说明 |
|------|---------|------|
| `layout` | 全部 | 12 列网格布局，典型 `w=4,h=3` |
| `type` | 全部 | 图表类型枚举（附录 D） |
| `name` | 全部 | 图表名称；实际存为独立属性 `ext-base-name` |
| `stackType` | 柱状/条形/折线/**面积** | `noStack` 不堆叠 / `stack` 堆叠 / `percentStack` 百分比堆叠（**面积图专属**）；面板**没有独立「堆叠」开关**，改堆叠 = 在「类型」里选**堆积**变体（堆积柱状图 / 堆积条形图 / **堆积折线图** / 堆积面积图 / 百分比堆积面积图） |
| `lineEffect` | 折线 / 面积 | 折线样式（面板「效果元素 > 效果」）：`"straight"` 直线 / `"smooth"` 曲线 / `"step"` 阶梯；**值是字符串**，如 `"step"`（旧记的 `"null"` 与直线同效，面板取值为 `"straight"`） |
| `pieType` | 饼图 | `pie` 饼形 / `ring` 环形 |
| `colors` | 全部 | 配色组 `{"name":"c1","value":["#…",…]}`；`value` 即调色板，可传自定义色值数组 |
| `splitLine` | 柱状/折线等 | 网格线开关（boolean） |
| `blankType` | 折线 / 面积（面板可改）/ 柱状等（仅存值） | 空值显示方式，**折线图与面积图的「空单元格显示为」写的就是它**（浏览器实测：零值=`2`默认 / 空距=`1` / 用直线连接数据点=`3`；面积图上把「空距」切回「零值」，变的也只有这一个键） |
| `funnelType` | 漏斗 | `"base"` 普通漏斗 / `"convert"` 转化漏斗（面板「类型」；实测切过去只改这一个键，转化漏斗会多出「转化层标签」区块） |
| `funnelDataSum` | 漏斗 | 「累计是否包含下层数据」：`true` = 面板**包含**（默认）/ `false` = **不包含**（两档：button `value="true"/"false"`，实测回读值一一对应） |
| `shape` | 雷达 / **词云** | 雷达形状：`"circle"` 圆形（默认）/ `"polygon"` 多边形（面板「自定义配置 > 效果元素 > 形状」；实测切换只改这一个键）；**词云形状**：`cloud` 云朵（默认）/ `heart` 心形 / `light` 灯泡 / `toast` 气泡 / `good` 点赞 / `notGood` 点踩 / `up` 向上 / `down` 向下，面板「自定义配置 > 词云形状」（词云组件实测回读 `shape:"cloud"`；取值由面板切换实测得到） |
| `actualTextNum` | 词云 | 词云的「展示个数」滑块（词云组件实测存在；**滑块量程 = 0 ~ 该字段去重后的关键词条数**——实测该组件面板显示 `0 —— 2`，因为「情感分析」只有「差评 / 空白」2 个去重关键词；而组件里存的仍是默认值 `50`，图上也只渲染这 2 个词，即**量程以字段去重条数为上限**，详见附录 E 词云图行） |
| `chartStyle` | 雷达 | 雷达样式：`"stroke"` = 面板「类型 → **雷达图**」（默认）/ `"fill"` = 「类型 → **填充雷达图**」（实测切换只改这一个键） |
| `valueAxisConfig` | 雷达 | 雷达图的「数值轴」`{show, unit, format:{type,config}, showLabel, showAxisLine, fontStyle, min?, max?, interval?}`。⚠️ **雷达图不写 `yAxisStyle`**（实测：面板「数值轴 → 显示轴线」切隐藏后，回读变的是 `chart-info.valueAxisConfig.showAxisLine`） |
| `categoryAxisConfig` | 雷达 | 雷达图的「类别轴」`{show, fontStyle}`（面板**只给文本样式**，没有单位/最小最大值等） |
| `labelContent` / `labelPosition` / `labelFontStyle` | 雷达 | 雷达图的数据标签**平铺在 chart-info 里**（直角坐标图用的是独立属性 `allLabelStyle`）。`labelPosition` = `top` 顶部 / `bottom` 底部 / **`inside` 内部**（三档实测回读） |
| `xAxisBaseField` / `yAxisBaseField` | 矩阵气泡图 | 「横轴 / 纵轴」所用字段 ID（实测：`"B"` = 填报内容 = 面板「横轴」、`"C"` = 数量 = 面板「纵轴」） |
| `minBubbleSize` / `maxBubbleSize` | 矩阵气泡图 | 气泡大小 = 面板「自定义配置 > 图表元素 > 气泡大小」的**最小/最大气泡半径**（0~999 整数；实测 8→9 回读 `minBubbleSize:9`） |
| `labelNumberUnit` / `labelNumberFormat` / `labelDecimalPlaces` / `labelShowThousandSeparator` / `labelCurrencySymbol` | 矩阵气泡图 | 数据标签的数字格式（面板「显示单位 / 数字格式 / 小数位数 / 千位分隔符 / 货币符号」）：`labelNumberUnit` 与 `leftYAxis.unit` **同一套量级**（`1`=无 … `1000000000000`=兆）、`labelNumberFormat` 与 `leftYAxis.format.type` **同一套枚举**（`0` 来自源数据 / `1` 数值 / `2` 百分比 / `3` 货币；实测把「数字格式」选「数值」→ 0→1） |
| `showLegend` | 直角坐标图 / 漏斗图 / 雷达图等 | 图例开关，**三类图都吃这个键**（漏斗图 A/B 实测：`chart-info.showLegend` 置 `false` 重载后图例消失、置 `true` 重载后图例出现）。⚠️ **漏斗图与雷达图**在切换「图例」时还会**同时**在属性表里新建顶层 `legendStyle`（其中只有 `legendPosition`，**没有** `visible`；见附录 E） |
| `valueLabel` / `formatType` | 统计数字 | 面板「统计值说明」→ `valueLabel`（纯文本）；「数字格式」→ `formatType`（**7 档**：`Num` / `Percent` / `RMB` / `Dollar` / `Euro` / `TenThousand` / `TenThousandYuan`）。⚠️ 「小数位数」「千位分隔符」**不在这里**——写的是独立属性 `valueFormatStyle`（见附录 E 统计数字行） |
| `progressType` / `dataType` / `targetValue` / `currentValue` / `dateRange` / `currentDate` / `statisticFormat` / `statisticDecimalPlaces` / `percentDecimalPlaces` / `showThousandSeparator` / `progressColor` | 进度图 | 面板「图表类型」→ `progressType`（`line` 条形 / `semicircle` 半环 / `circle` 环形）；「类型」→ `dataType`（`numberType`/`dateType`）；「目标值 / 当前值」→ `targetValue`（默认 100）/ `currentValue`（默认 50），`targetValueType`/`currentValueType` = `custom`/`select`；日期态 → `dateRange`（`thisMonth`/`thisQuarter`/`thisYear`/`custom`）+ `customDateRange` + `currentDate`（`today`/`yesterday`/`custom`）；数字格式 → `statisticFormat`（**9 档**，比统计数字多 `JPY`/`GBP`）+ `statisticDecimalPlaces` + `percentDecimalPlaces`（缺省 `Decimal2`）+ `showThousandSeparator`；「主题色」→ `progressColor`（**色板下标 0 起，或色值字符串**）。全部实测回读 |

图表的**样式类属性**（纵轴 `yAxisStyle`、数据标签 `allLabelStyle`、图例 `legendStyle`、组件背景色 `cardBgColor` 等）
不在 `chart_info` 内，改它们请用 `dbsheet.dashboard_component_update` 的 `property_modify_list`（key 对照表见该工具 `param_detail`）。

⚠️ **例外：雷达图**——它的「数值轴」「类别轴」「数据标签」「图例」「网格线」**全都写在 `chart-info` 里**
（`valueAxisConfig` / `categoryAxisConfig` / `labelContent`+`labelPosition`+`labelFontStyle` / `showLegend` / `showSplitLine`+`showAxisLine`+`showSplitArea`）；
雷达图组件的 `properties` 里**没有** `yAxisStyle`、`allLabelStyle`（实测只有 `chart-info` + `ext-base-name` 两个属性键）。

**附录 E：图表类型专属字段（已实测：柱状图 / 条形图 / 折线图 / 面积图 / 饼状图 / 柱线图 / 漏斗图 / 雷达图 / 矩阵气泡图 / 桑基图 / 共现网络图 / 词云图 / 统计数字 / 进度图）**

| 图表类型 | 数值轴设置 | 类型专属字段 | 面板**没有**的控件 | 数据标签内容 |
|---|---|---|---|---|
| 柱状图 BasicBar | `yAxisStyle`（面板叫「纵轴」） | `chart-info.stackType`（堆叠；面板经「类型 → 堆积柱状图」选）；数据标签「标签位置」= `allLabelStyle.barLabelPosition`（4 档，取值见通用规则）；支持「显示范围」(TopN)，但**按当前统计配置条件出现**（见通用规则） | 折线样式、空值显示方式 | `value` / `categoryName` / `seriesName` |
| 条形图 HorizBar | **同样写 `yAxisStyle`**（面板叫「横轴」；实测：改「自定义配置 > 图表元素 > 横轴 > 显示单位」，回读变的是 `yAxisStyle.leftYAxis.unit`，`xAxisStyle` 不动） | `chart-info.stackType`（堆叠；面板经「类型 → 堆积条形图」选）；数据标签「标签位置」= **`allLabelStyle.horizBarLabelPosition`**（4 档，键名与柱状图不同）；支持「显示范围」(TopN)，同样**条件出现**（见通用规则） | 折线样式、空值显示方式 | `value` / `categoryName` / `seriesName` |
| 折线图 BasicLine | `yAxisStyle`（面板叫「纵轴」） | `chart-info.stackType`（堆叠；面板经「类型 → **堆积折线图**」选）、`chart-info.lineEffect`（折线样式，面板「效果元素 > 效果」）、`chart-info.blankType`（空值显示方式，面板「空单元格显示为」在**基础配置**：零值=2 / 空距=1 / 用直线连接数据点=3）；数据标签「标签位置」= `allLabelStyle.lineLabelPosition`（**3 档**，键名与柱状图不同） | 显示范围 TopN（**代码里就没有这个控件**） | `value` / `categoryName` / `seriesName` |
| 面积图 Area | `yAxisStyle`（面板叫「纵轴」） | `chart-info.stackType`（**三档**：`noStack` 基础面积图 / `stack` 堆积面积图 / `percentStack` 百分比堆积面积图，面板经「类型」选，三档全部实测回读）、`chart-info.lineEffect`（折线样式，面板「效果元素 > 效果」，与折线图同键同值：`straight`/`smooth`/`step`）、`chart-info.blankType`（空值显示方式，面板「空单元格显示为」在**基础配置**，与折线图同键同值：零值=2 / 空距=1 / 用直线连接数据点=3；`emptyCellDisplayMode` **不在面积图组件上**）；数据标签「标签位置」= **`allLabelStyle.areaLabelPosition`**（**3 档**：顶部 `top` / 居中 `inside` / 底部 `bottom`，三档实测回读） | 显示范围 TopN（**配置代码里就没有这个控件**，实测把「排序依据」改成纵轴也不出现） | `value` / `categoryName` / `seriesName` |
| 饼状图 BasicPie | **无坐标轴**（不写 `yAxisStyle`） | `chart-info.pieType`（`pie` 饼形 / `ring` 环形，面板经「类型 → 饼图/圆环图」选）、数据标签位置/内容用**饼图专属键** `allLabelStyle.pieLabelPosition`（`outside`/`inside`）与 `allLabelStyle.pieLabelContent` | 网格线、系列颜色（面板叫**「扇区颜色」**）、堆叠、显示范围 TopN（**代码里就没有**）、分组（图例）（维度叫「扇区分组」） | `value` / `categoryName` / **`percent`**（百分比，饼图专属） |
| 漏斗图 Funnel | **无坐标轴、无网格线**（不写 `yAxisStyle`）；维度在面板叫「**漏斗层**」，下方还有「**漏斗层顺序**」拖拽列表 | `chart-info.funnelType`（`base` 普通漏斗 / `convert` 转化漏斗）、`chart-info.funnelDataSum`（累计是否包含下层数据）；标签设置**全部**在 `allLabelStyle` 的**分区块对象**里：`{funnelLayer:{show,labelContent,fontStyle}, funnelTransLayer:{…}（仅转化漏斗）, funnelTotalTransRate:{…}}`；配色面板叫「**漏斗颜色**」（按层逐项） | 分组（图例）、排序依据/排序方式（改用「漏斗层顺序」）、显示范围 TopN、数据标签位置 | `funnelLayer` 名称/值（`categoryName`/`value`）、`funnelTransLayer` 名称/百分比、`funnelTotalTransRate` 名称/百分比 |
| 柱线图 BarLine | **双轴写在同一个 `yAxisStyle` 里**：主轴=`yAxisStyle.leftYAxis`、次轴=`yAxisStyle.rightYAxis`（面板「自定义配置 > 图表元素 > 纵轴 > **轴类型 主轴/次轴**」切换；基础配置里的统计字段分列「统计值（左坐标轴）」「统计值（右坐标轴）」两栏）；**外层还须同时带扁平字段**，只写 `leftYAxis` 或只写扁平层都不生效 | 每个统计字段带 `plotType`（`Bar`/`Line`）+ `yAxisIndex`（`0` 主轴/`1` 次轴），写在 `statisticSettingAddList` 项里 | 堆叠、显示范围 TopN（**代码里就没有**）、**分组（图例）**（基础配置无此控件，实测排序依据改成纵轴也不出现）、数据标签的**「标签位置」**（柱线图的数据标签只有 标签内容 / 标签样式） | `value` / `categoryName` / `seriesName`（直角坐标图面板都提供，**非柱线图专属**） |
| 雷达图 Radar | **不写 `yAxisStyle`**：面板「数值轴」= `chart-info.valueAxisConfig`、「类别轴」= `chart-info.categoryAxisConfig` | `chart-info.shape`（`circle` 圆形 / `polygon` 多边形，面板「效果元素 > 形状」）、`chart-info.chartStyle`（`stroke` 雷达图 / `fill` 填充雷达图，面板「类型」）、数据标签用**平铺键** `showLabel`+`labelContent`+`labelPosition`+`labelFontStyle`、「网格线/刻度线/网格区域」= 平铺的 `showSplitLine`/`showAxisLine`/`showSplitArea`；基础配置的「**类别轴**」是**多字段 + 每字段聚合**（求和 / 计数 / 最大值 / 最小值 / 平均值 / 去重计数，可增删），「**数值轴（系列）**」是单字段 | 分组（图例）、排序依据/排序方式、显示范围 TopN、空单元格显示为、`allLabelStyle`、纵轴「显示单位」 | `value` / `categoryName` / `seriesName`（实测下拉：系列名称 / 分类名称 / 值） |
| 矩阵气泡图 MatrixBubble | **面板没有坐标轴样式区块**（属性里虽然有 `yAxisStyle`，但面板不写它，见陷阱 6） | `chart-info.xAxisBaseField` / `yAxisBaseField`（横轴 / 纵轴字段 ID）、`chart-info.minBubbleSize` / `maxBubbleSize`（气泡大小）、`chart-info.splitLine`（面板「自定义配置 > 网格线」**单个开关**，实测 true→false）、数据标签数字格式 `labelNumberUnit` / `labelNumberFormat` / `labelDecimalPlaces` / `labelShowThousandSeparator` / `labelCurrencySymbol`；**图例位置写在独立属性 `legendStyle`**（`chart-info` 里没有 `legendPosition`） | 类型（只有 1 种）、分组（图例）、排序依据/排序方式、显示范围、空单元格显示为、数据标签「标签内容 / 标签位置」、坐标轴样式区块 | 只有**数字格式**：显示单位（无 / 百 / 千 / 万 / 十万 / 百万 / 千万 / 亿 / 十亿 / 兆）+ 数字格式（来自源数据 / 数值 / 百分比 / 货币）+ 小数位数 + 千位分隔符 + 货币符号（`¥` / `€`） |
| 桑基图 Sankey | **无坐标轴、无网格线**（不写 `yAxisStyle`）；「**节点类别**」是**两个字段**：源节点= `statisticsModule.dimensionFieldId`、目标节点= `groupFieldId`（实测把第二个字段从「数量」切到「日期」→ `groupFieldId:"C"→"D"`，第一个字段对应 `dimensionFieldId`） | 「**节点高度(值)**」= 记录总数 / 指定字段（实测切「指定字段」→ `onlyCountDimension:true→false`，面板同时多出「字段 + 聚合」两个下拉 = `statisticSettingAddList`）；「**排序方式**」= 默认 / 自定义拖拽（实测切「自定义拖拽」→ `statisticsModule.sortType:"Custom"` **且** `sortByRow:true→false`；切回「默认」→ `sortType:"SortByHorizontalAxis"` + `sortByRow:true`）；数据标签是**平铺键** `chart-info.showLabel` / `labelContent` / `labelFontStyle`（同雷达图写法，实测关掉「数据标签」→ `showLabel:true→false`） | 类型、分组（图例）、排序依据/排序方式（改用上面那个「排序方式」）、显示范围、空单元格显示为、坐标轴样式的子项（最小值/最大值/显示单位等）、图例样式、网格线 / 刻度线 / 网格区域 | `value` / **`nodeCategory`**（节点类别；实测 `allLabelStyle.labelContent:["value","nodeCategory"]`，`chart-info.labelContent` 同值） |
| 共现网络图 CoGraph | **无坐标轴**（不写 `yAxisStyle`）；「**类型**」= 环形布局 / 自由布局（实测 → `chart-info.layoutType` 的 `"circular"` / `"cola"`，两个选项在下拉里是**图标缩略图**、没有文字，选中后下拉框才显示名字）；「**节点类别**」= 两个字段：`chart-info.firstNodeCategoryField` / `secondNodeCategoryField`（实测 `"B"` = 填报内容、`"C"` = 数量；statistics 侧对应 `dimensionFieldId` / `groupFieldId`） | 「**连接线(边)**」= 记录总数 / 指定字段（即 `statisticsModule.onlyCountDimension`，与桑基图「节点高度(值)」同款）；「**节点大小**」的「最小/最大节点半径」= `chart-info.minNodeSize` / `maxNodeSize`（默认 8 / 20）；「**图例**」开关 = `chart-info.showLegend`，**图例位置同时写两处**（`chart-info.legendPosition` 与独立属性 `legendStyle.legendPosition`，实测切「底部」两处都变 `"bottom"`）；数据标签是**平铺键** `showLabel` / `labelContent` / `labelFontStyle`（同桑基图；「标签内容」= 值 / 节点类别） | 分组（图例）（面板只有图例开关+位置，没有分组字段）、排序依据/排序方式、显示范围、空单元格显示为、坐标轴样式、网格线、系列颜色（面板叫「**节点颜色**」，按节点逐项） | `value` / `nodeCategory`（实测 `chart-info.labelContent:["value","nodeCategory"]`） |
| 词云图 WordCloud | **无坐标轴、无网格线**（不写 `yAxisStyle`）；「**关键词字段**」= `statisticsModule.dimensionFieldId`（实测面板选「情感分析」→ `"J"`）；「**统计方式**」有三档 = 关键词次数（默认）/ 指定其他字段 / 关键词次数（按分割符） | ① 「统计方式」**只落在一个布尔位上**：`statisticsModule.onlyCountDimension`（关键词次数 = `true`，另外两档 = `false`；实测 true→false，切回「关键词次数」又变回 true）；② 「**词云形状**」= `chart-info.shape`（实测 `cloud`→`toast`→还原 `cloud`）；③ 「**展示个数**」滑块 = `chart-info.actualTextNum`（默认 `50`；实测拖到中段 → `50`→`1`，词云立刻只剩 1 个词，滑块量程 0~2 = 该字段去重后的关键词个数）；④ 「指定其他字段」多出的「字段 + 聚合」两个下拉、「按分割符」多出的「**分割符号**」（斜杠 / 分号 / 逗号 / 空格 / 其他[单字符输入]）**在 API 侧读不到**（切换前后回读 `properties` / `statisticsModule`，只有 `onlyCountDimension` 变了；改完刷新页面 UI 状态仍在） | 分组（图例）、排序依据/排序方式、显示范围、空单元格显示为、坐标轴样式、网格线、图例、**图表元素 / 数据标签**（词云图的自定义配置只有「色彩元素」「词云形状」两块） | 面板没有「数据标签」区块（词云图不渲染数据标签）；形状名 = 云朵 `cloud` / 心形 `heart` / 灯泡 `light` / 气泡 `toast` / 点赞 `good` / 点踩 `notGood` / 向上 `up` / 向下 `down` |
| 统计数字 Value | **无坐标轴、无数据标签**（不写 `yAxisStyle`，也没有 `allLabelStyle`）；**没有「类型」下拉**（无变体） | `chart-info.valueLabel`（面板「统计值说明」，纯文本，默认空）、`chart-info.formatType`（面板「数字格式」，**7 档**：`Num` 数字 / `Percent` 百分比 / `RMB` 人民币 / `Dollar` 美元 / `Euro` 欧元 / `TenThousand` 单位:万 / `TenThousandYuan` 单位:万元）。⚠️ `Percent` 是**值 ×100 再加 %**（实测 28 → `2800%`），别当成"加个百分号"、**独立属性** `valueFormatStyle`（面板「小数位数」+「千位分隔符」→ `{"decimalPlaces":"Integer\|Decimal1\|Decimal2","showThousandSeparator":true\|false}`，**不在 `chart-info` 里**）、`chart-info.colors`（面板「主题色」→ `{name,value:[6 个 hsl],refIdx,version}`，`refIdx` = 色板下标）；「统计值（纵轴）」= 记录总数 / 指定字段 → `statisticsModule.onlyCountDimension`（指定字段时多出 `statisticSettingAddList`） | 类型（无变体）、排序依据、排序方式、分组、显示范围、空单元格显示为、数据标签、图例、网格线、坐标轴 | 无（整卡 = 大号数字 + 说明文字） |
| 进度图 Progress | **无坐标轴、无数据标签**（不写 `yAxisStyle`，也没有 `allLabelStyle`）；**类型 = 数值 / 日期**两态，日期态下数据相关控件几乎全隐藏 | `chart-info.progressType`（面板「图表类型」：`line` 条形 / `semicircle` 半环 / `circle` 环形）、`chart-info.dataType`（面板「类型」：`numberType` 数值 / `dateType` 日期）、数值态 `chart-info.targetValueType` + `targetValue`（面板「目标值」，默认 100）与 `chart-info.currentValueType` + `currentValue`（面板「当前值」，`custom` 自定义值 / `select` 选择字段，默认 50）、日期态 `chart-info.dateRange`（`thisMonth` 本月 / `thisQuarter` 本季度 / `thisYear` 今年 / `custom` 自定义）+ `customDateRange:["YYYY-MM-DD","YYYY-MM-DD"]` + `currentDate`（`today` 今日 / `yesterday` 昨日 / `custom` 自定义）、数字格式 `chart-info.statisticFormat`（**9 档**：`Num`/`Percent`/`RMB`/`JPY`/`Dollar`/`Euro`/`GBP`/`TenThousand`/`TenThousandYuan`）+ `statisticDecimalPlaces` + `showThousandSeparator`、百分比 `chart-info.percentDecimalPlaces`、「主题色」= `chart-info.progressColor`（**两态**：色板**第一行 6 色**写 **0 起下标**，顺序与 `chart-info.colors.value` **一致**——实测点第 3 个色块 → `progressColor:2` → 进度条渲染紫 `#9670FF` = `colors.value[2]`，默认 `0` = 蓝 `#2977FF`；**第二行 8 个「标准色」**写的是**色值字符串**——实测点红块 → `progressColor:"#EB5451"`。见通用规则） | 排序依据、排序方式、分组、显示范围、空单元格显示为、数据标签、图例、网格线、坐标轴；**日期态下**「数据源 / 数据范围 / 目标值 / 当前值 / 明细数据显示的字段」全部不渲染 | 无（整卡 = 进度图形 + 当前/目标或日期范围文字） |

**通用规则**

- **排序统一用 `sortByRow` + `ascending`**（`sortByRow:true` 按类目轴 / `false` 按值轴；实测饼图同样生效，扇区顺序会变）——饼图面板把这两档叫 **「扇区名称」/「扇区数值」**（值仍是 `SortByHorizontalAxis` / `SortByVerticalAxis`），但它**没有**「分组（图例）」和「显示范围」；
- **轴样式一律写 `yAxisStyle`**：条形图的数值轴虽是横轴，存储键仍是 `yAxisStyle`；写 `xAxisStyle` 属于无效键（能存下但渲染忽略）；
- **堆叠经「类型」选择**：面板没有独立「堆叠」开关——柱状图选「堆积柱状图」、条形图选「堆积条形图」、**折线图选「堆积折线图」**（折线因此也有 `stackType`，不是没有）；饼图/柱线图没有堆积变体；
- **「类型」下拉的全量变体（实测，分组图标网格）**：柱状图 基础柱状图/堆积柱状图 · 折线图 基础折线图/**堆积折线图** · 条形图 基础条形图/堆积条形图 · 饼状图 **饼图/圆环图** · 面积图 基础面积图/堆积面积图/**百分比堆积面积图** · 柱线图（仅 1 种） · 雷达图 雷达图/填充雷达图 · 漏斗图 普通漏斗/转化漏斗 · 词云图（仅 1 种）；同一变体在同一下拉里按图族分组，选变体即切换 `chart-info.type` 或对应开关（如饼图/圆环图 → `pieType`，面积图三档 → `stackType` = `noStack` / `stack` / `percentStack`）；
- **「类型」变体以面板的图标网格为准（实测）**：**必须以「在面板切到该变体后回读 `chart-info`」为准**——实测把折线图切到第 2 个变体（堆积折线图）后，`chart-info.type` 仍是 `"BasicLine"`，只有 `stackType` 由 `"noStack"` 变成 `"stack"`（切回第 1 个变体再变回 `"noStack"`）。各图族变体数（实测）：柱状图 2 / **折线图 2** / 条形图 2 / 饼状图 2 / **面积图 3** / 柱线图 1 / 雷达图 2 / 漏斗图 2 / 词云图 1 ——因此「百分比堆积」(`percentStack`) 只有**面积图**能在面板选到（柱状图 / 条形图的面板未暴露该变体）。
- **数据标签内容面板选项**：直角坐标图（柱状/条形/折线/柱线）都是 系列名称 / 分类名称 / 值（`seriesName` / `categoryName` / `value`）；饼图是 分类名称 / 值 / 百分比；
- **数据标签位置面板选项**（各类型不同，别互相照搬）：柱状图 **4 档 居中 / 顶部 / 外部 / 底部**；条形图 **同样 4 档 居中 / 顶部 / 外部 / 底部**；折线图 **3 档 顶部 / 居中 / 底部**（**没有「外部」**）；饼图 **2 档 外部 / 内部**（存储键是 `pieLabelPosition`，见上）；**柱线图根本没有「标签位置」控件**（只有 标签内容 / 标签样式）；
- **「标签位置」的存储键按图族各自独立（实测 5 个键），取值不能按文案望文生义**（均实测，切换后回读 `allLabelStyle`，已恢复原值）：**柱状图** = `barLabelPosition`（居中 `inside` / 顶部 `insideTop` / **外部 `top`** / 底部 `insideBottom`）；**条形图** = **`horizBarLabelPosition`**（居中 `inside` / 顶部 `insideRight` / 外部 `outside` / 底部 `insideLeft`）；**折线图** = **`lineLabelPosition`**（顶部 `top` / 居中 `inside` / 底部 `bottom`）；**面积图** = **`areaLabelPosition`**（顶部 `top` / 居中 `inside` / 底部 `bottom`，三档实测回读）；饼图 = `pieLabelPosition`。⚠️ 给条形图 / 折线图写 `barLabelPosition` 是**无效键**（这两个图族各自只认自己的键；实测折线图属性里遗留的 `barLabelPosition:"outside"` 面板不会去改它）；⚠️ 折线图遇到枚举外的值会**回落到第一档「顶部」**（实测把条形图遗留的 `insideLeft` 留在折线图上，面板显示「顶部」）；⚠️ 标签位置的取值与文案是**错位**的：柱状图的「外部」存 `top`、「顶部」存 `insideTop`，条形图的「顶部」存 `insideRight`、「底部」存 `insideLeft` —— 不要按字面猜值（面积图实测另有独立键 `areaLabelPosition`，别按同族关系类推）；⚠️ 这些键名以「改前回读 `properties`、改后再回读」为准，最可靠；
- **图例位置面板选项**：顶部 / 底部 / 左侧 / 右侧（**四档**，不是只有上下）；
- **「显示范围」(TopN)：只有柱状图 / 条形图有，而且在面板上是「条件出现」**。类型层面：柱状图（BarChart）与条形图（HorizBarChart）的配置代码里有 TopN，折线图（LineChart）/ 饼状图（PieChart）/ 柱线图（BarLineChart）**完全没有** TopN 代码 → 后三者别写 `onlyTopNDimensions`；条件层面：柱状图/条形图的控件仅在「分组（图例）为空 + 合并同类别开启 + **排序依据=纵轴**（按统计值排序）+ 统计值要么是记录总数、要么指定字段 ≤1 个」时渲染，任一不满足控件就从「基础配置」消失。**实测（柱状图）**：分组=无 & 排序依据=横轴 → 无「显示范围」；把「排序依据」改成**纵轴** → 立刻出现（`全部数据`/`自定义范围`）；再把分组设为某字段 → 又消失。取值：`全部数据`=不限制（等价 0 / 不传）、`自定义范围`=**1~20**（越界会被夹到边界值）；⚠️ **实测该值写不进 API**：在柱状图上写 `onlyTopNDimensions` = `5` / `0` / `1` / `20` / `21` 都返回成功但组件载荷里始终没有这个键（`""` / `null` 报 `400100`），想设显示范围只能让用户在面板上选（同 `dashboard_component_update` 陷阱 26）。
- **不要按其他组件的当前值推断"默认值"**（看到某组件图例在底部，可能是被显式设过）；未设 `legendStyle` 时图例默认在**顶部**（折线图/条形图实测）。改样式前先读 `properties` 取现值；
- **「空单元格显示为」（只有折线图 / 面积图有，在基础配置）实际写的是 `chart-info.blankType`**：面板「**零值**」=`2`（默认）/ 「**空距**」=`1` / 「**用直线连接数据点**」=`3`（折线图三档全部实测回读：空距→1、零值→2、用直线连接数据点→3）。⚠️ **`emptyCellDisplayMode` 只存在于折线图组件**（值形如 `"Zero"`；**面积图组件里没有这个键**，它是历史实验写进折线图的残留），且**面板不会写它**——把「空单元格显示为」三档轮流切一遍，变的始终是 `chart-info.blankType`，`emptyCellDisplayMode` 原样不动 → **要控制空值显示请写 `blankType`，别写 `emptyCellDisplayMode`**；饼图/柱状图等面板没有此控件；
- **漏斗图专属（实测）**：① 维度在面板叫「**漏斗层**」，下面还有一个「**漏斗层顺序**」拖拽列表（列出各层，可拖动改序）——顺序**在 API 侧读不到**（`properties`/`statisticsModule` 里没有对应字段，只看到 `sortType:"Custom"`），要改顺序只能拖面板；② 基础配置没有「排序依据/排序方式/分组（图例）/显示范围」，自定义配置没有坐标轴、网格线、数据标签位置；③ 类型 = `chart-info.funnelType`（`base` 普通漏斗 / `convert` 转化漏斗），转化漏斗会多出「转化层标签」区块；④ 「累计是否包含下层数据」= `chart-info.funnelDataSum`（包含=`true` 默认 / 不包含=`false`）；⑤ **标签配置写成 `allLabelStyle` 的「分区块对象」**（与直角坐标图的扁平结构完全不同）：`funnelLayer`（漏斗层标签，选项 名称/值）、`funnelTransLayer`（转化层标签，选项 名称/百分比，**仅转化漏斗有**）、`funnelTotalTransRate`（总转化率，选项 名称/百分比），每块形如 `{"show":true,"labelContent":["categoryName","value"],"fontStyle":{…}}`；⑥ 配色面板叫「**漏斗颜色**」（按层逐项，不是「系列颜色」）；⑦ **分区块结构已有落盘证据（实测）**：新建漏斗组件后逐个关标签开关，属性表**追加**出 `allLabelStyle`——关「漏斗层标签」→ `{"funnelLayer":{"show":false,"labelContent":["categoryName","value"],"fontStyle":{…}}}`；关「总转化率」→ 再追加 `"funnelTotalTransRate":{"show":false,"labelContent":["categoryName","percent"],"fontStyle":{…}}`；把「类型」切到**转化漏斗**后关「转化层标签」→ 再追加 `"funnelTransLayer":{"show":false,"labelContent":["percent"],"fontStyle":{…}}`。⚠️ 三个子块的默认 `labelContent` **并不一致**：漏斗层 `["categoryName","value"]`、总转化率 `["categoryName","percent"]`、**转化层只有 `["percent"]`**；写入是**按块合并**语义，改一块别把另两块覆盖掉；⑧ ⚠️ 漏斗图**吃 `chart-info.showLegend`**（旧文档「漏斗不吃该键、走 `legendStyle.visible`」的说法**已推翻**）：面板「图例」开关写 `chart-info.showLegend` 并同时新建顶层 `legendStyle:{"legendPosition":"top"}`；A/B 实测把 `chart-info.showLegend` 置 `false` 重载后**图例消失**、置 `true` 重载后**图例出现**，且 `legendStyle` 里**没有** `visible` 键；
- **雷达图专属（实测）**：① 「数值轴」**不写 `yAxisStyle`**，而是 `chart-info.valueAxisConfig`（面板项：最小值 / 最大值 / 显示轴线 / 显示标签 / 数字格式 / 文本样式——**没有「显示单位」**；实测关掉「显示轴线」→ `valueAxisConfig.showAxisLine` 变 false）；② 「类别轴」= `chart-info.categoryAxisConfig`（面板只给**文本样式**）；③ 数据标签**不走 `allLabelStyle`**，是平铺的 `showLabel` / `labelContent` / `labelPosition` / `labelFontStyle`（实测：「数据标签」开关 → `showLabel` false→true；「标签位置」三档 = `top` 顶部 / `bottom` 底部 / **`inside` 内部**）；④ 「自定义配置 > 其他」的 **网格线 / 刻度线 / 网格区域** = 平铺的 `showSplitLine` / `showAxisLine` / `showSplitArea`（写 `gridElements` 不生效；实测关掉「网格区域」→ `showSplitArea` true→false）；⑤ 「形状」= `chart-info.shape`（`circle`/`polygon`）；「类型 → 雷达图 / 填充雷达图」= `chart-info.chartStyle`（`stroke`/`fill`）；⑥ 基础配置的「**类别轴**」是**多字段 + 每字段聚合**（选项：求和 / 计数 / 最大值 / 最小值 / 平均值 / 去重计数），「**数值轴（系列）**」是**单字段**——⚠️ 存储上 `statisticsModule.dimensionFieldId` 对应的是面板的**「数值轴（系列）」**（实测 `dimensionFieldId:"B"` = 字段「填报内容」，**不是**「类别轴」），而「类别轴」那一串字段**读不回来**（`dashboard_component_list` 不返回）；⑦ 基础配置**没有** 分组（图例）/ 排序依据 / 排序方式 / 显示范围 / 空单元格显示为；自定义配置**没有** 网格线以外的坐标轴区块，数据标签也没有「标签位置外置」以外的写法；⑧ 「图例」开关写 `chart-info.showLegend`，并会**在属性表里新建顶层 `legendStyle`**（`{"legendPosition":"top","fontStyle":{…}}`；该键写进去就删不掉，见陷阱 2）；⑨ **`mapType` / `sortBy` / 嵌套 `chart-info.legendStyle` 三者已实测**：面板**没有**对应控件，但 API **可写、且渲染真的会变**——A/B 实测写入 `{"mapType":"recordCategory","sortBy":"recordOrder"}` 并重载 → 雷达由「同心环 + 数值轴刻度」变成「按记录分布的多轴形态」；删掉这两键重载 → 还原 ⇒ 属有效键，不是残留脏数据（`mapType`：`fieldCategory` 按字段分类（默认）/ `recordCategory` 按记录分类；`sortBy`：`categoryValue`（默认）/ `recordOrder`）。**嵌套的 `chart-info.legendStyle` 由「图例位置」写入**（实测 顶部→底部：`chart-info.legendPosition:"bottom"` + `chart-info.legendStyle:{"legendPosition":"bottom"}` + 顶层属性 `legendStyle:{"legendPosition":"bottom"}` 三处同步更新）；只拨「图例」开关时只写 `chart-info.showLegend` + 新建顶层 `legendStyle`，**不会**产生嵌套键；
- **矩阵气泡图专属（实测）**：① 轴的语义和其它图不同——「**横轴**」= 类别字段、「**纵轴**」= 数值字段，两者下方各有一个**值列表**（可拖动排序，列表最多展示 20 个选项）；存储键是 `chart-info.xAxisBaseField` / `yAxisBaseField`（实测 `"B"` = 填报内容 = 横轴、`"C"` = 数量 = 纵轴），statistics 侧对应 `statisticsModule.dimensionFieldId`（横轴）/ `groupFieldId`（纵轴）；② 「**气泡值**」= 记录总数 / 指定字段（实测 `statisticsModule.onlyCountDimension:true`）；③ 基础配置**没有** 类型 / 分组（图例）/ 排序依据 / 排序方式 / 显示范围 / 空单元格显示为；④ 自定义配置 = 色彩元素（主题色 / 组件背景色 / 系列颜色[按纵轴数值逐项]）→ 图表元素（数据标签 / 图例 / **气泡大小**）→ **网格线**；⑤ 「气泡大小」的「最小/最大气泡半径」= `chart-info.minBubbleSize` / `maxBubbleSize`（实测 8→9 回读 `minBubbleSize:9`）；⑥ 「网格线」是**单个开关** → 写 `chart-info.splitLine`（实测 true→false），**没有**「刻度线/网格区域」子项；⑦ 数据标签**没有**「标签内容 / 标签位置」，只有「标签样式 / 显示单位 / 数字格式」（选 数值/百分比/货币 时多出「小数位数」「千位分隔符」，货币再多一个符号 `¥`/`€`）——写 `chart-info.showLabel` + `labelNumberUnit` / `labelNumberFormat` / `labelDecimalPlaces` / `labelShowThousandSeparator` / `labelCurrencySymbol`，同时会在属性表里**新建 `allLabelStyle`**（`{"fontStyle":{…}}`，键删不掉）；⑧ 「图例」开关写 `chart-info.showLegend`，而**图例位置写在独立属性 `legendStyle`**（`{"legendPosition":"top",…}`）——`chart-info` 里**没有** `legendPosition`；⑨ 属性里虽然有 `yAxisStyle`，但面板**没有**坐标轴样式区块（同陷阱 6，写了也不生效）；
- **桑基图专属（实测）**：① 基础配置 = 标题 / 数据源 / 数据范围 / **节点类别**（两个字段下拉）/ **节点高度(值)**（记录总数 | 指定字段）/ **排序方式**（默认 | 自定义拖拽）/ 明细数据显示的字段——**没有**「类型」下拉、分组（图例）、显示范围、空单元格显示为；② 自定义配置 = 色彩元素（主题色 / 组件背景色）→ **节点颜色**（按节点逐项，一个节点一个颜色下拉）→ 图表元素（数据标签：**标签内容 = 值 / 节点类别**，加标签样式）——**没有** 网格线 / 图例 / 效果元素 / 坐标轴区块；③ 数据标签**平铺在 `chart-info` 里**（`showLabel` / `labelContent` / `labelFontStyle`，与雷达图同一写法），实测「数据标签」开关 → `chart-info.showLabel` true↔false；属性表里同时还有 `allLabelStyle:{labelContent,fontStyle}`（两处都会被写）；④ 面板的「**排序方式**」只有 「默认」/「自定义拖拽」两档（**不是**其它图的「排序依据 + 排序方式」），实测落到 `statisticsModule.sortType`（`SortByHorizontalAxis`/`Custom`）与 `sortByRow`（true/false）；⑤ 统计配置与数据标签的明细**不落盘到可读字段**；⑥ 组件 `properties` 只有 `chart-info` + `ext-base-name`（+ 写过的 `allLabelStyle`），**没有** `yAxisStyle` / `legendStyle`；配色面板叫「**节点颜色**」（按节点逐项，与漏斗「漏斗颜色」、饼图「扇区颜色」同类）。
- **共现网络图专属（实测）**：① 基础配置 = 标题 / 数据源 / 数据范围 / **类型**（环形布局 / 自由布局）/ **节点类别**（**两个字段**）/ **连接线(边)**（记录总数 | 指定字段）/ 明细数据显示的字段——**没有** 分组（图例）、排序、显示范围、空单元格显示为；② 自定义配置 = 色彩元素（主题色 / 组件背景色）→ **节点颜色**（按节点逐项）→ 图表元素（**数据标签** 开关 + 标签内容[值 / 节点类别] + 标签样式；**图例** 开关 + 图例位置[顶部 / 底部 / 左侧 / 右侧] + 文本样式；**节点大小** 最小节点半径 / 最大节点半径）；③ 实测映射：「类型」写入 `chart-info.layoutType`（`circular` 环形布局 / `cola` 自由布局；`circular` 已由组件回读交叉验证，`cola` 亦经实测确认，且该图默认配置为 `Circular`）；「节点类别」写入 `chart-info.firstNodeCategoryField` / `secondNodeCategoryField`；「连接线(边)」写入 `statisticsModule.onlyCountDimension`；「节点大小」= `chart-info.minNodeSize` / `maxNodeSize`；④ 数据标签**平铺在 `chart-info` 里**（`showLabel` / `labelContent` / `labelFontStyle`），实测打开开关 → `showLabel:false→true`，**且会在属性表里新建顶层 `allLabelStyle`**（又一个陷阱 2 的坑）；⑤ ⚠️ 图例位置**写两处**：`chart-info.legendPosition` + 独立属性 `legendStyle.legendPosition`（实测切「底部」两处同时变）——与矩阵气泡图相反（矩阵气泡图的 `chart-info` 里**没有** `legendPosition`）；⑥ 组件 `properties` 里**没有** `yAxisStyle`。
- **词云图专属（实测）**：① 基础配置 = 标题 / 数据源 / 数据范围 / **类型**（词云族只有「词云图」1 个变体，下拉里同时可切换到其它图族）/ **关键词字段**（候选过滤**已实测**：该表 23 个字段里入选 19 个——**滤掉** 关联类（`Object`：简历助手 / 电子发票识别）、富文本（`Note`）、图片和附件（`Attachment`）；**复选框（Checkbox）、单选（情感分析）、多选、日期、数字、文本、联系人、评分都保留**——即普通复选框字段**会**出现在下拉里（实测新建一个复选框字段后确实出现在候选列表末尾），旧文档「复选框被滤掉」的说法**已推翻**；**时间类字段只在多数据源（带 `moduleId`）时才排除**，这正是单数据源下「日期」仍在候选里的原因）/ **统计方式**（关键词次数 | 指定其他字段 | 关键词次数（按分割符））/ **分割符号**（仅「按分割符」时出现：斜杠 / 分号 / 逗号 / 空格 / 其他 + 单字符输入框）/ **展示个数**（滑块）/ **词频表**（「查询词频表」按钮弹出「查看词频」弹窗，两列 = 关键词 / 词频统计）/ 明细数据显示的字段——**没有** 分组（图例）、排序依据、排序方式、显示范围、空单元格显示为；② 自定义配置 = 色彩元素（主题色 / 组件背景色）→ **词云形状**（下拉选择，8 种：云朵 / 心形 / 灯泡 / 气泡 / 点赞 / 点踩 / 向上 / 向下）——**没有** 图表元素、数据标签、图例、网格线、坐标轴区块；③ 能回读的写入只有三处：`statisticsModule.onlyCountDimension`（统计方式）、`chart-info.shape`（词云形状）、`chart-info.actualTextNum`（展示个数）；「统计方式」的「字段 + 聚合」明细与「分割符号」明细**不落盘到可读字段**；④ 「**多项内容拆开统计**」复选框**已实测**：只有当「关键词字段」= 支持拆分的字段（`supportSplitOption`，如多选「智能分类」）**且**「统计方式」= 关键词次数 / 指定其他字段时，才会出现在关键词字段下拉框**正下方**；勾选后**整页重载仍保持勾选**（确实随组件配置持久化），但**API 读不到**——`dashboard_component_list` 的 `statisticsModule` / `properties` 都查不到（该开关不落盘到可读字段）⇒ **不要指望用 API 复现或校验这个开关**。
- **统计数字（Value）专属（实测）**：① 基础配置 = 标题 / 数据源 / 数据范围 / **统计值说明**（写 `chart-info.valueLabel`）/ **统计值（纵轴）**（记录总数 | 指定字段 → `statisticsModule.onlyCountDimension`）/ 明细数据显示的字段——**没有**「类型」下拉、排序、分组、显示范围、空单元格显示为；② 自定义配置 = 色彩元素（主题色 / 组件背景色）→ **统计元素**（数字格式 / 小数位数 / 千位分隔符）——**没有** 数据标签、图例、网格线、坐标轴；③ ⚠️ **两套数字格式键名要分清**：「数字格式」写 `chart-info.formatType`（7 档），而「小数位数 + 千位分隔符」写**独立属性 `valueFormatStyle`**（实测把「小数位数」改成 1位小数 + 「千位分隔符」改成 显示 → 属性表新增 `valueFormatStyle:{"decimalPlaces":"Decimal1","showThousandSeparator":true}`，`chart-info` 里**没有**这两个键；卡片渲染同步变成 `28.0`）；⚠️ 反过来把 `valueFormatStyle` 写进 `chart-info` **也能生效**（实测 `chart-info.valueFormatStyle.decimalPlaces="Decimal2"` → 卡片显示 `28.00`），但面板的规范位置是独立属性，**优先写属性**；④ 「主题色」写 `chart-info.colors`（`{name,value,refIdx,version}`，`refIdx` 是色板下标；实测选第 3 个色块 → `refIdx:2`）；⑤ 「组件背景色」仍是三键一组 `cardBgColor` / `cardBgOpacity` / `cardBgBlur`（实测选色后 = `#2B93CC` / `100` / `0`）；⑥ 用 API 新建的 Value 组件 `chart-info` 里只有 `layout`/`name`/`type`，`valueLabel`/`formatType`/`valueFormatStyle` **都是面板点出来的**——别指望创建时自动带默认值。
- **进度图（Progress）专属（实测）**：① 基础配置 = 标题 / **类型**（数值 | 日期）/ 数据源 / 数据范围 / **图表类型**（条形 / 半环 / 环形 三个图标变体，悬停有说明文字）/ **目标值** / **当前值**（自定义值 | 选择字段）/ 明细数据显示的字段；选「日期」后只留 标题 / 类型 / 图表类型 / **日期范围**（本月 | 本季度 | 今年 | 自定义）/ **当前日期**（今日 | 昨日 | 自定义），其余项**不渲染**；② 自定义配置 = 色彩元素（主题色 / 组件背景色）→ **当前/目标值**（数字格式 / 小数点位数 / 千位分隔符）→ **百分比设置**（小数点位数）——**没有** 数据标签、图例、网格线、坐标轴；③ 实测键名：「图表类型」→ `chart-info.progressType`（`line`/`semicircle`/`circle`）；「类型」→ `chart-info.dataType`（`numberType`/`dateType`）；「目标值 / 当前值」→ `chart-info.targetValue`（默认 100）/ `chart-info.currentValue`（默认 50）；「日期范围」→ `chart-info.dateRange`（`thisMonth`/`thisQuarter`/`thisYear`/`custom`），选「自定义」会多出日期区间选择器并写 `chart-info.customDateRange:["2026-09-15","2026-09-29"]`；「当前日期」→ `chart-info.currentDate`（`today`/`yesterday`/`custom`）；④ ⚠️ **进度图的数字格式键名与统计数字完全不同**：「数字格式」→ `chart-info.statisticFormat`（**9 档**，比统计数字多 `JPY` 日元与 `GBP` 英镑）、「当前/目标值 小数点位数」→ `chart-info.statisticDecimalPlaces`、「百分比设置 小数点位数」→ `chart-info.percentDecimalPlaces`、「千位分隔符」→ `chart-info.showThousandSeparator`（**全都平铺在 `chart-info` 里**，不是统计数字那种独立 `valueFormatStyle` 属性）；⑤ 「主题色」→ `chart-info.progressColor`，**两态**：色板**第一行 6 色**写 **0 起下标**，顺序与 `chart-info.colors.value` **一致**（实测点第 3 个色块 → `progressColor:2`，进度条渲染为紫色 `#9670FF` = `colors.value[2]`；默认 `0` = 蓝 `#2977FF`，与"第 N 个色块 = 下标 N-1"吻合）；**第二行 8 个「标准色」**写的是**色值字符串**（实测点红块 → `progressColor:"#EB5451"`）——两行语义不同，选完色请回读确认；不是 `colors`/`theme_style`；⑥ 默认值：`statisticFormat` 缺省 = `Num`、`statisticDecimalPlaces` 缺省 = `Integer`、`percentDecimalPlaces` 缺省 = `Decimal2`（百分比显示 2 位小数）、`showThousandSeparator` 缺省 = false、`progressType` 缺省 = 条形；日期态直接按「当前日期在所选区间内的占比」自动算（例：区间=本月、今天=9/22 → 73%），**不需要**配数据源；⑦ 老组件上可能残留旧版写法 `chart-info.valueFormatStyle:{decimalPlaces}`（面板已改为写平铺的 `statisticDecimalPlaces`）——两者并存时以面板新键为准，别照抄旧键。
- **饼图的数据标签用「专属键」**（浏览器实测已定位）：标签位置写 **`allLabelStyle.pieLabelPosition`**（`"outside"` 外部 / `"inside"` 内部），标签内容写 **`allLabelStyle.pieLabelContent`**（如 `["categoryName","value"]`）；在面板把「标签位置」由 外部 切到 内部 后回读，`allLabelStyle` 新增的就是这两个键，老键 `barLabelPosition`/`position` 原样不动、对饼图**无效** → 给饼图写位置别再写 `barLabelPosition`；渲染实测：写 `pieLabelPosition:"inside"` 后标签确实移进扇区内部（外部时带引导线在外侧）；

**附录 F：默认值速查表（未手动改动时的默认值）**

用法：用 API 建出来的组件，`chart-info` 里**只存 `layout` / `name` / `type`**；下面这些默认值**不落盘到组件**（面板打开时按默认值渲染）。
所以「面板上看到的值 ≠ 组件里存的值」——**想对齐面板效果就必须显式写入**（`chart_info` 或 `property_modify_list`），只有回读到的键才算真正落盘。

| `chart-info.type` | 组件 | 默认键值（组件里未显式写入时生效） |
|------|------|------|
| `BasicBar` | 柱状图 | `splitLine:true` · `blankType:0`（零值）· `stackType:"noStack"` · `showLabel:true` · `showLegend:true` |
| `HorizBar` | 条形图 | 同柱状图 |
| `BasicLine` | 折线图 | 同柱状图 + `lineEffect:null`（**不是** `"smooth"`，要平滑得显式写） |
| `Area` | 面积图 | 未出现在已抓取的默认表里，但同族键都在——现网用面板建的组件实测存 `splitLine:true` · `blankType:2`（零值）· `lineEffect:"null"`（**存的是字符串 `"null"`，不是 JSON null**）· `stackType:"noStack"` · `showLabel:true` · `showLegend:true` |
| `BasicPie` | 饼图 / 环形图 | `pieType:"pie"` · `showLabel:true` · `showLegend:true` |
| `BarLine` | 柱线组合图 | `splitLine:true` · `showLabel:true` · `showLegend:true` · `shape:"cloud"` |
| `Funnel` | 漏斗图 | `funnelType:"convert"`（**默认就是转化漏斗**）· `funnelDataSum:true` · `showLabel:true` · `showLegend:false` |
| `Radar` | 雷达图 | `shape:"circle"` · `chartStyle:"stroke"` · `showLabel:false` · `showLegend:true` · `legendPosition:"top"` · `sortBy:"categoryValue"` · `showSplitLine:true` · `showSplitArea:true` · `showAxisLine:true` · `labelContent:["value"]` · `labelPosition:"top"` · `valueAxisConfig:{show:true,unit:1,showLabel:true,showAxisLine:true}` · `categoryAxisConfig:{show:true}` |
| `WordCloud` | 词云图 | `shape:"cloud"` · `actualTextNum:50` |
| `MatrixBubble` | 矩阵气泡图 | `splitLine:true` · `showLabel:true` · `showLegend:true` · `legendPosition:"top"` · `labelNumberUnit:1` · `labelNumberFormat:0` · `labelDecimalPlaces:0` · `labelShowThousandSeparator:false` · `labelCurrencySymbol:"¥"` |
| `Sankey` | 桑基图 | 未取到独立默认表；现网用面板建的组件只落了**平铺的数据标签键** `showLabel:true` · `labelContent:["value","nodeCategory"]` · `labelFontStyle:{…}`（没有 `splitLine`/`showLegend` 等） |
| `CoGraph` | 共现网络图 | `layoutType:"circular"` · `showLabel:false` · `showLegend:true` · `legendPosition:"top"` · `labelContent:["value","nodeCategory"]` · `minNodeSize:8` · `maxNodeSize:20` |
| `Value` | 统计数字 | `valueLabel:"记录总数"` · `formatType:"Num"` |
| `Progress` | 进度图 | `progressType:"line"` · `dataType:"numberType"` · `targetValueType:"custom"` · `targetValue:100` · `currentValueType:"custom"` · `currentValue:50` · `dateRange:"thisMonth"` · `currentDate:"today"` · `percentDecimalPlaces:"Decimal2"` · `progressColor:0` |

**通用样式默认**（面板补到 `properties` 独立属性上的那几块）：

| 属性 | 默认值 |
|------|------|
| `allLabelStyle` | `{labelContent:["value"], pieLabelContent:["categoryName","value","percent"], barLabelPosition:"top", lineLabelPosition:"top", pieLabelPosition:"outside", horizBarLabelPosition:"outside", fontStyle:{fontSize:12,bold:false,italic:false}, funnelLayer:{show:true,labelContent:["categoryName","value"]}, funnelTransLayer:{show:true,labelContent:["percent"]}, funnelTotalTransRate:{show:true,labelContent:["categoryName","percent"]}}` |
| `legendStyle` | `{legendPosition:"top", fontStyle:{fontSize:12,bold:false,italic:false}}` |
| `yAxisStyle` | `{show:true, leftYAxis:{fontStyle:{fontSize:12,bold:false,italic:false}, unit:1(无), unitText:"", format:{type:0,config:{}}, isLogScale:false, angle:<水平>, showLabel:true, showAxisLine:false}, rightYAxis:<同 leftYAxis>}` |
| `valueFormatStyle`（统计数字） | `{decimalPlaces:"Integer", showThousandSeparator:undefined}` |
| 动画 / 缩放 | `animation:"auto"` · `animationDuration:1000` · `animationThreshold:2000` · `fontScale:1` |

**两个数字枚举的取值**（避免猜错）：显示单位 `None=1` / `Hundred=100` / `Thousand=1000` / `TenThousand=1e4` / `HundredThousand=1e5` / `Million=1e6` / `TenMillion=1e7` / `Billion=1e8` / `TenBillion=1e9` / `Trillion=1e12`（**「无」是 1，不是 0**，所以现网组件里的 `unit:1` 就是「无」而不是「百」）；数字格式 `Source=0` / `Number=1` / `Percent=2` / `Currency=3`（矩阵气泡图组件实测 `labelNumberUnit:1` + `labelNumberFormat:0` = 无单位 + 来自源数据）。
**附录 C：统计选项枚举（statisticOptions）**

**实测可用（8 个）**：`count` 计数 · `sum` 求和 · `avg` 平均值 · `max` 最大值 · `min` 最小值 ·
`uniqueCount` 唯一计数 · `minTime` / `maxTime` 最小/最大时间。

⚠️ **实测不可用（10 个，写入一律 `400100 第三方服务错误：Internal error / Unknown enum`）**：
`filledCount` 填充计数 · `unfilledCount` 未填充计数 · `filledPercentage` 已填写百分比 · `uniquePercentage` 唯一百分比 ·
`checkedCount` 选中计数 · `uncheckoutCount` 未选中计数 · `checkedPercentage` 选中百分比 · `uncheckedPercentage` 未选中百分比 ·
`dayRange` / `monthRange` 日/月范围。实测连「数值字段 + `filledCount`」「**复选框字段** + `checkedCount`」都被拒
（各复测 2~3 次，同批对照写入成功 ⇒ 不是偶发故障，就是未实现）——复选框字段实际只能用 `count` / `uniqueCount`。

**可用范围按字段类型分档**（实测矩阵，完整支持表见附录 G）：

| 字段类型 | 可用统计选项 |
|---------|-------------|
| 数值类：数值 / 百分比 / 货币 / 进度条 / 评分 / 数值公式 | `count` `sum` `avg` `max` `min` `uniqueCount` |
| 日期时间类：日期 / 时间 / 创建时间 | `count` `uniqueCount` `minTime` `maxTime` |
| 文本·编号·电话·邮箱：文本 / 自动编号 / 电话 / 邮箱 | `count` `uniqueCount` |
| 选项·联系人·复选框：单选 / 多选 / 联系人 / 复选框 | `count` `uniqueCount` |
| 富文本 / 附件 / 关联 | 无（任意选项均 `400100`） |

**附录 D：图表类型枚举（chart_info.type）**

| 值 | 组件 | 值 | 组件 |
|----|------|----|------|
| `BasicBar` | 柱状图 | `Funnel` | 漏斗图 |
| `BasicLine` | 折线图 | `Radar` | 雷达图 |
| `HorizBar` | 条形图 | `Area` | 面积图 |
| `BasicPie` | 饼图/环形图 | `MatrixBubble` | 矩阵气泡图 |
| `BarLine` | 柱线组合图 | `Sankey` | 桑基图 |
| `CoGraph` | 共现网络图 | `WordCloud` | 词云图 |
| `Value` | 统计数字（指标卡） | `Progress` | 进度图 |

以上 14 种均为正式图表类型；雷达图、桑基图、共现网络图、词云图**已支持**（不是插件类）。

**附录 G：字段类型支持矩阵（维度 / 分组 / 统计值 / 筛选，数据层实测）**

统计口径：用临时组件对 22 个真实字段（文本 / 数值 / 日期 / 富文本 / 附件 / 评分 / 多选 / 单选 / 关联 / 联系人）
+ 临时新建的 10 个类型字段（复选框 / 时间 / 百分比 / 进度条 / 编号 / 创建时间 / 公式 / 电话 / 邮箱 / 货币）
逐一写入，**写入成功且 `dashboard_chart_data` 返回有效数据**才算「支持」；非法组合返回
`400100 第三方服务错误：Internal error / Unknown enum`。

| 字段类型 | 维度 | 分组 | 统计值 | 筛选字段 |
|---------|------|------|--------|---------|
| 文本 `MultiLineText` · 编号 `AutoNumber` · 电话 `Phone` · 邮箱 `Email` | ✅ | ✅ | `count` `uniqueCount` | ✅ |
| 数值 `Number` · 百分比 `Percentage` · 货币 `Currency` · 进度条 `Complete` · 评分 `Rating` · 数值公式 `Formula` | ✅ | ✅ | `count` `sum` `avg` `max` `min` `uniqueCount` | ✅ |
| 日期 `Date` · 时间 `Time` · 创建时间 `CreatedTime` | ✅ | ✅ | `count` `uniqueCount` `minTime` `maxTime` | ✅ |
| 单选 `SingleSelect` · 多选 `MultipleSelect` · 联系人 `Contact` · 复选框 `Checkbox` | ✅ | ✅ | `count` `uniqueCount` | ✅ |
| 富文本 `Note` · 附件 `Attachment` · 关联 `Object` | ❌ `400100` | ❌ `400100` | ❌ 任意选项均 `400100` | ✅（能写，命中与否看数据） |

实测要点：

- **维度 / 分组的排除规则与词云「关键词字段」完全一致**（`Note` / `Attachment` / `Object`，另含 `Signature`），
  可统一记成一条；被排除时**写入即报错**，不会静默；
- **分组字段不能等于维度字段**：`dimensionFieldId` 与 `groupFieldId` 传同一个字段时，写 `groupFieldId` 返回
  `400100`（`J`/`J`、`D`/`D` 实测）；
- **筛选字段是例外**：所有类型都能写（含富文本 / 附件 / 关联），失败只表现为 0 命中；`Empty` / `NotEmpty`
  必须传 `values: []`（落盘为 `values:null`），实测语义正确（文本 27/1、姓名 0/28、数值 7/21）；
- ⚠️ **`400100` + `Internal error` 在本服务里是「校验拒绝」的常规报错**（字段类型不匹配 / 枚举非法 /
  分组等于维度），看着像服务故障，**别当故障反复重试**，先检查字段类型与枚举组合；
- ⚠️ 排查「图表空白」时先看 `onlyCountDimension`：`false` 且 `statisticSettingAddList` 为空时图表会渲染为
  空（本次实测踩到过，容易被误判成筛选失效）——要么置 `true` 走记录总数，要么补一条统计系列。

#### 返回值说明

```json
{
  "detail": {
    "webExtension": {
      "webExtensionKey": "{F4FD07CD-9414-4EFB-873E-3E9ED5D69EF8}",
      "webExtensionType": "DbDataSource",
      "dataSourceType": "dbTable",
      "statisticsModule": {
        "id": "C",
        "dataSourceSheetId": 1,
        "dimensionFieldId": "E",
        "groupFieldId": "A",
        "sortByRow": true,
        "ascending": true,
        "onlyCountDimension": false
      }
    }
  },
  "result": "ok"
}

```

| 字段 | 类型 | 说明 |
|------|------|------|
| `detail.webExtension.webExtensionKey` | string | 组件 key（{GUID}），后续 update/delete/取数用 |
| `detail.webExtension.webExtensionType` | string | 部件类型 |
| `detail.webExtension.statisticsModule` | string | 生效后的统计配置 |
| `result` | string | ok 表示成功 |


---

## 9. dbsheet.dashboard_component_update

#### 功能说明

更新仪表盘图表组件的统计配置、筛选条件或前端属性。
`web_extension_key` 来自 `dbsheet.dashboard_component_list` 返回的 `webExtensionKey`。
只改布局/名称（chart_info）时不传 `statistics_module_setting` 也安全：不传时统计配置保持不变。

按「要改的东西存在哪」二选一，两个通道互不干扰：
- **前端属性**（标题、配色、组件背景色、纵轴、数据标签、图例、网格线）→ 走 `property_modify_list`；
- **统计配置**（数据源、维度、统计值、分组、排序、筛选）→ 走 `statistics_module_setting`。

改样式前先 `dbsheet.dashboard_component_list` 读 `properties` 取现值：独立属性是**整串覆盖**，
只传子键会丢掉同属性内的其它子键。完整 key 对照表、筛选写法与陷阱见 `param_detail`。

#### 调用约束

- **前置检查**：`web_extension_key` 来自 `dbsheet.dashboard_component_list`，不得捏造；改前端属性前先用同一工具读 `properties` 取现值（独立属性为整串覆盖）；新增筛选条件前先 `clearFilter:true`（追加语义）
- **后置验证**：`dbsheet.dashboard_component_list` 回读 `properties`/`statisticsModule` 确认已更新；筛选/统计的数值变化用 `dbsheet.dashboard_chart_data` 核对

**幂等性**：是

#### 调用示例

将统计字段从计数改为求和：

```json
{
  "file_id": "string",
  "dashboard_id": 6,
  "web_extension_key": "{E65A9EA9-D0D0-451D-988F-F46B1FD20EAD}",
  "statistics_module_setting": {
    "statisticSettingChangeList": [
      {
        "oldFldId": "C",
        "newFldId": "D",
        "statisticOptions": "sum"
      }
    ]
  }
}
```

清空筛选条件：

```json
{
  "file_id": "string",
  "dashboard_id": 6,
  "web_extension_key": "{E65A9EA9-D0D0-451D-988F-F46B1FD20EAD}",
  "statistics_module_setting": {
    "clearFilter": true
  }
}
```

#### 参数说明

- `url` (string, 三选一必填: `url` / `link_id` / `file_id`): 文档 URL
- `link_id` (string, 三选一必填: `url` / `link_id` / `file_id`): 分享链接 ID
- `file_id` (string, 三选一必填: `url` / `link_id` / `file_id`): 文件 ID
- `dashboard_id` (integer, 必填): 仪表盘 ID
- `web_extension_key` (string, 必填): 组件 key（{GUID}）
- `statistics_module_setting` (object, 可选): 统计配置（字段同 create，见该工具附录 A）。更新专用字段：statisticSettingChangeList[{oldFldId,newFldId,statisticOptions}]、statisticSettingRemoveList[{fieldId}]、filterCriteriasRemoveList[{fieldId}]、clearFilter:true 清空筛选、filtersOp:"And"|"Or"。⚠️ filterCriteriasAddList / statisticSettingAddList 是**追加**语义（不替换），改条件前必须先 clearFilter 或 RemoveList，否则条件累积、互相 AND 成空结果。不传时保持原值（合并语义）
- `chart_info` (object, 可选): 图表前端属性（同 create 附录 B）；会自动包装成 `chart-info` 属性，与 property_modify_list 二选一
- `property_modify_list` (array, 可选): 前端属性通道：[{key, value}]。`key` 用**存储键**（chart-info / allLabelStyle / legendStyle / yAxisStyle / ext-base-name / cardBgColor 等），`value` 一律为字符串（对象/数组写 JSON 串，需转义）。一次可传多个；key 对照表见 param_detail
- `plugin_config` (object, 可选): 插件类组件配置（预留）

**通道一：property_modify_list（前端属性）**

`key` 必须是**存储键**（如数据标签要写 `allLabelStyle`，写面板内部用名不生效）；
`value` 一律字符串——标量写字面量，对象/数组写 JSON 串（需转义）。
⚠️ **各图族的默认值速查表见 `dbsheet.dashboard_component_create` 的附录 F**：用 API 建出来的组件 `chart-info` 只存 `layout`/`name`/`type`，其余默认值**不落盘**——**想对齐面板效果必须显式写入**，别以为「面板里看着是 X」就等于组件里存着 X。

| 存储键 | 含义 | 值 |
|--------|------|-----|
| `ext-base-name` | 卡片标题 | 字符串 |
| `chart-info` | 图表本体（JSON 串） | `{layout, type, colors, splitLine, blankType, stackType, showLabel, showLegend, legendPosition, legendFontStyle}`；**漏斗图另有** `funnelType`（`base`/`convert`）与 `funnelDataSum`（`true` 包含 / `false` 不包含）；**雷达图另有** `shape`、`chartStyle`、`valueAxisConfig`、`categoryAxisConfig`、`showSplitLine`/`showAxisLine`/`showSplitArea`；**矩阵气泡图另有** `xAxisBaseField`/`yAxisBaseField`（横轴/纵轴字段 ID）、`minBubbleSize`/`maxBubbleSize`（气泡大小）、数据标签数字格式 `labelNumberUnit`/`labelNumberFormat`/`labelDecimalPlaces`/`labelShowThousandSeparator`/`labelCurrencySymbol`；**桑基图另有** 平铺的数据标签键 `showLabel`/`labelContent`/`labelFontStyle`（同时还会写 `allLabelStyle`）；**共现网络图另有** `layoutType`（`"circular"`/`"cola"`）、`firstNodeCategoryField`/`secondNodeCategoryField`、`minNodeSize`/`maxNodeSize`、平铺的数据标签键与 `legendPosition`（图例位置**同时**写在 `chart-info` 和 `legendStyle` 里）；**统计数字另有** `valueLabel`（统计值说明）、`formatType`（数字格式，7 档）、`colors`（主题色，`refIdx` = 色板下标）；**进度图另有** `progressType`（`line`/`semicircle`/`circle`）、`dataType`（`numberType`/`dateType`）、`targetValueType`+`targetValue`、`currentValueType`+`currentValue`、`dateRange`+`customDateRange`+`currentDate`（日期态）、`statisticFormat`+`statisticDecimalPlaces`+`percentDecimalPlaces`+`showThousandSeparator`（数字格式）、`progressColor`（主题色，色板下标 0 起或色值字符串） |
| `allLabelStyle` | 数据标签样式（JSON 串） | 直角坐标图 `{labelContent:["value"\|"categoryName"\|"seriesName"\|"percent"], barLabelPosition, fontStyle:{fontSize,bold,italic}}`；**饼图用专属键** `{pieLabelContent:[…], pieLabelPosition:"outside"\|"inside", fontStyle:{…}}`（写 `barLabelPosition` 对饼图无效）；**漏斗图是「按区块的对象」** `{funnelLayer:{show,labelContent,fontStyle}, …}`（详见取值要点与陷阱 14）；面板选项：标签内容——直角坐标图（柱状/条形/折线/柱线）为 **系列名称/分类名称/值**，饼图为 **分类名称/值/百分比**；标签位置——柱状图 居中/顶部/外部/底部、折线图 顶部/居中/底部、饼图 外部/内部。⚠️ 标签位置的存储键**按图族不同**：柱状图 `barLabelPosition`、条形图 `horizBarLabelPosition`、折线图 `lineLabelPosition`、饼图 `pieLabelPosition`；上面写的 `barLabelPosition` **只适用于柱状图**（取值见取值要点） |
| `legendStyle` | 图例（JSON 串） | `{legendPosition:"top"\|"bottom"\|"left"\|"right", fontStyle:{fontSize,bold,italic}}`（面板 **顶部/底部/左侧/右侧 四档**，不是只有上下） |
| `yAxisStyle` | 纵轴（JSON 串） | `{show, leftYAxis:{unit, format:{type,config}, min, max, interval, showLabel, showAxisLine, fontStyle}}`；**柱线图**在此属性内加 `rightYAxis:{…}` 表示次轴 |
| `cardBgColor` / `cardBgOpacity` / `cardBgBlur` | 组件背景色（三键一组） | `"#EFB310"` / `"100"` / `"0"` |
| `valueFormatStyle` | **统计数字（Value）** 的「小数位数 + 千位分隔符」（JSON 串，**独立属性，不在 `chart-info` 里**） | `{"decimalPlaces":"Integer"\|"Decimal1"\|"Decimal2","showThousandSeparator":true\|false}`（面板改「小数位数」或「千位分隔符」时整体重写；进度图**不用**这个键，它写在 `chart-info` 里） |
| `blankType`（在 `chart-info` 里） | **折线图 / 面积图**的空值显示方式（面板「空单元格显示为」，在**基础配置**） | 数字：`2` = 面板「**零值**」（默认）/ `1` = 「**空距**」/ `3` = 「**用直线连接数据点**」（折线图三档实测回读；面积图切换后变的也只有这一个键） |
| `valueAxisConfig`（在 `chart-info` 里） | **雷达图**的「数值轴」 | `{show, unit, format:{type,config}, showLabel, showAxisLine, fontStyle, min?, max?, interval?}`；⚠️ 雷达图**不写** `yAxisStyle`（实测：面板把「数值轴」落到这里） |
| `categoryAxisConfig`（在 `chart-info` 里） | **雷达图**的「类别轴」 | `{show, fontStyle}`（面板**只给文本样式**，没有单位/最小最大值等） |
| `shape` / `chartStyle`（在 `chart-info` 里） | **雷达图**的「形状」与「类型」 | `shape`：`"circle"` 圆形（默认）/ `"polygon"` 多边形；`chartStyle`：`"stroke"` = 类型「雷达图」（默认）/ `"fill"` = 「填充雷达图」（两键均实测回读） |
| `showSplitLine` / `showAxisLine` / `showSplitArea`（在 `chart-info` 里） | **雷达图**的「网格线 / 刻度线 / 网格区域」（面板在「自定义配置 > 其他」） | 平铺 boolean（实测关掉「网格区域」→ `chart-info.showSplitArea` 变 false；写 `gridElements` 不生效） |
| `xAxisBaseField` / `yAxisBaseField`（在 `chart-info` 里） | **矩阵气泡图**的「横轴 / 纵轴」（横轴=类别字段、纵轴=数值字段） | 字段 ID 字符串（实测 `"B"` = 填报内容 = 面板「横轴」、`"C"` = 数量 = 面板「纵轴」）；statistics 侧对应 `statisticsModule.dimensionFieldId`（横轴）/ `groupFieldId`（纵轴） |
| `minBubbleSize` / `maxBubbleSize`（在 `chart-info` 里） | **矩阵气泡图**的「气泡大小」（面板「自定义配置 > 图表元素 > 气泡大小」的最小/最大气泡半径） | 0~999 整数（实测 `8` → `9` 回读 `minBubbleSize:9`） |
| `labelNumberUnit` / `labelNumberFormat` / `labelDecimalPlaces` / `labelShowThousandSeparator` / `labelCurrencySymbol`（在 `chart-info` 里） | **矩阵气泡图**数据标签的「显示单位 / 数字格式 / 小数位数 / 千位分隔符 / 货币符号」 | `labelNumberUnit` 与 `leftYAxis.unit` **同一套量级**（`1`=无 … `1000000000000`=兆）；`labelNumberFormat` 与 `leftYAxis.format.type` **同一套枚举**（`0` 来自源数据 / `1` 数值 / `2` 百分比 / `3` 货币；实测选「数值」→ 0→1）；其余为小数位数（integer）、千位分隔符（boolean）、货币符号（`"¥"`） |
| `shape` / `actualTextNum`（在 `chart-info` 里） | **词云图**的「词云形状」与「展示个数」 | `shape`：`"cloud"` 云朵（默认）/ `"heart"` 心形 / `"light"` 灯泡 / `"toast"` 气泡 / `"good"` 点赞 / `"notGood"` 点踩 / `"up"` 向上 / `"down"` 向下（实测 `cloud`→`toast`→`cloud`）；`actualTextNum`：正整数，默认 `50`（实测改成 `1` 后词云只剩 1 个词）。⚠️ 词云图的 `shape` 与雷达图的 `shape`（`circle`/`polygon`）**同键不同义**，按 `type` 区分 |
| `emptyCellDisplayMode` | ⚠️ **别用**：只出现在**折线图**属性里（值形如 `"Zero"`；**面积图组件里没有这个键**，实测是历史实验写入折线图的残留），**面板不写它**、改「空单元格显示为」也不会动它（实测）| 控制空值显示请改 `chart-info.blankType` |
| `weboffice-chart-<GUID>` | 图表引擎开关 | `"true"` |

**取值要点**（均实测）

- `colors.value` 就是**调色板**，可传自定义色值数组（`chart-info.colors`）；
- `chart-info.lineEffect`（**折线图 / 面积图专属**，面板叫「效果元素 > 效果」）：`"straight"` 直线 / `"smooth"` 曲线 / `"step"` 阶梯，**值是字符串**（`"step"` 实测渲染成 L 形阶梯线）；旧记的 `"null"` 与直线同效，面板取值为 `"straight"`；三个档位由面板按钮 value 实测；
- `leftYAxis.unit` 是**数值量级**不是序号（10 档全部实测，轴顶会显示对应中文单位）：
  `1`=无、`100`=百、`1000`=千、`10000`=万、`100000`=十万、`1000000`=百万、`10000000`=千万、
  `100000000`=亿、`1000000000`=十亿、`1000000000000`=兆；
  传非法值不报错，会被当**除数**静默生效（传 `2` → 刻度变原值÷2）；
- `leftYAxis.format.type`：`0` 来自源数据 / `1` 数值 / `2` 百分比 / `3` 货币
  （`config:{}` 即用默认符号与 2 位小数；`0`/`2`/`3` 已实测渲染：百分比 → `0%`、货币 → `¥30.00`）；
- `leftYAxis.min` / `max` / `interval` / `fontStyle`：**JSON 里删掉该键 = 恢复「自动」**；
- 字号可选值：`10 / 12 / 14 / 16 / 18 / 20 / 24 / 30 / 36`；
- 图例是**双写**：`legendPosition`/`legendFontStyle` 在 `legendStyle` 与 `chart-info` 里各存一份，改一处会不一致；
- **饼图专属**：无坐标轴（不要写 `yAxisStyle`）；数据标签 `allLabelStyle.labelContent` 面板选项为 分类名称/值/**百分比**（`percent`，实测渲染成 `3.57%`，与 `value`/`categoryName` 可组合）；配色面板叫**「扇区颜色」**（不是「数据点颜色」；同系列颜色的写后不可回读通道）；维度/统计在面板叫**「扇区分组」/「扇区数值」**；其标签位置/内容是**饼图专属键**（浏览器实测）：位置写 `allLabelStyle.pieLabelPosition`（`outside`/`inside`）、内容写 `allLabelStyle.pieLabelContent`（如 `["categoryName","value"]`）——面板把「标签位置」切到「内部」后回读新增的就是这两个键，写老键 `barLabelPosition`/`position` 对饼图**无效**（渲染实测 `inside` 后标签进入扇区内部）；
- **「标签位置」的存储键按图族各自独立（实测 5 个键），且取值与文案错位**（实测，切换后回读 `allLabelStyle`，已恢复原值）：**柱状图** = `barLabelPosition`（居中 `inside` / 顶部 `insideTop` / **外部 `top`** / 底部 `insideBottom`）；**条形图** = **`horizBarLabelPosition`**（居中 `inside` / 顶部 `insideRight` / 外部 `outside` / 底部 `insideLeft`）；**折线图** = **`lineLabelPosition`**（顶部 `top` / 居中 `inside` / 底部 `bottom`）；**面积图** = **`areaLabelPosition`**（顶部 `top` / 居中 `inside` / 底部 `bottom`，三档实测回读）；饼图 = `pieLabelPosition`。⚠️ 给条形图 / 折线图写 `barLabelPosition` 是**无效键**；⚠️ 折线图遇到枚举外的值会**回落到第一档「顶部」**（实测把条形图遗留的 `insideLeft` 留在折线图上，面板就显示「顶部」）；⚠️ 不要按文案猜值——柱状图「外部」存 `top`、「顶部」存 `insideTop`，条形图「顶部」存 `insideRight`、「底部」存 `insideLeft`。⚠️ 面积图**既不是** `lineLabelPosition` 也**不是** `barLabelPosition`（实测另有独立键 `areaLabelPosition`），别按同族类推；⚠️ 键名不能按同族类推，改前先回读 `properties`、改后再回读校验。
- **折线图 / 面积图专属**：`lineEffect`（面板「效果元素 > 效果」，三档图标按钮 `straight` / `smooth` / `step`）与空值显示方式（面板「空单元格显示为」，基础配置）——**写 `chart-info.blankType`**（`2` 零值 / `1` 空距 / `3` 用直线连接数据点），不是 `emptyCellDisplayMode`；数据标签位置**只有 顶部 / 居中 / 底部 三档**（**没有「外部」**，与柱状图的四档不同）；两者都**没有**「显示范围」（配置代码里根本没有 TopN，实测把「排序依据」改成纵轴也不出现）；面积图另有 `stackType` **三档**：`noStack` 基础面积图 / `stack` 堆积面积图 / **`percentStack` 百分比堆积面积图**（面板经「类型」选，三档全部实测回读）；
- **漏斗图专属（实测）**：① 维度面板叫「漏斗层」，另有「漏斗层顺序」拖拽列表——**顺序在 API 侧读不到**（`properties`/`statisticsModule` 里没有对应字段，只看到 `sortType:"Custom"`），只能拖面板改；② 类型 = `chart-info.funnelType`（`base` 普通漏斗 / `convert` 转化漏斗，实测切换只改这一个键）；③ 「累计是否包含下层数据」= `chart-info.funnelDataSum`（包含=`true` 默认 / 不包含=`false`）；④ **标签设置全在 `allLabelStyle` 的「分区块对象」里**：`funnelLayer`（漏斗层标签：名称/值）、`funnelTransLayer`（转化层标签：名称/百分比，**仅转化漏斗**）、`funnelTotalTransRate`（总转化率：名称/百分比），每块 `{show, labelContent, fontStyle}`——和直角坐标图的扁平结构完全不同，别照搬；⑤ 配色面板叫「**漏斗颜色**」（按层逐项）；⑥ 没有坐标轴/网格线/排序依据/分组（图例）/显示范围/数据标签位置；⑦ ⚠️ 漏斗图**吃 `chart-info.showLegend`**（旧文档「不吃该键、走 `legendStyle.visible`」**已推翻**）：面板「图例」开关写 `chart-info.showLegend` 并同时新建顶层 `legendStyle:{"legendPosition":"top"}`；A/B 实测置 `false` 重载后图例消失、置 `true` 重载后图例出现，`legendStyle` 里**没有** `visible` 键；⑧ **④ 的分区块结构已有落盘证据（实测）**：逐个关标签开关会在 `allLabelStyle` 里**按块追加**——关「漏斗层标签」→ `funnelLayer:{show:false,labelContent:["categoryName","value"],…}`；关「总转化率」→ 追加 `funnelTotalTransRate:{show:false,labelContent:["categoryName","percent"],…}`；「类型」切转化漏斗后关「转化层标签」→ 追加 `funnelTransLayer:{show:false,labelContent:["percent"],…}`（三块默认 `labelContent` **不一致**，转化层只有 `["percent"]`）；
- **雷达图专属（实测）**：① 「数值轴」**不写 `yAxisStyle`**，而是 `chart-info.valueAxisConfig`（面板：最小值 / 最大值 / 显示轴线 / 显示标签 / 数字格式 / 文本样式，**没有「显示单位」**）；② 「类别轴」= `chart-info.categoryAxisConfig`（面板只给文本样式）；③ 数据标签**不走 `allLabelStyle`**，是平铺的 `showLabel` / `labelContent` / `labelPosition` / `labelFontStyle`（「标签位置」三档 = `top` 顶部 / `bottom` 底部 / **`inside` 内部**）；④ 网格线 / 刻度线 / 网格区域 = 平铺的 `showSplitLine` / `showAxisLine` / `showSplitArea`（写 `gridElements` 不生效）；⑤ 形状 = `chart-info.shape`（`circle`/`polygon`）；类型 = `chart-info.chartStyle`（`stroke` 雷达图 / `fill` 填充雷达图）；⑥ 基础配置的「**类别轴**」是**多字段 + 每字段聚合**、「**数值轴（系列）**」是单字段——`statisticsModule.dimensionFieldId` 对应的是**「数值轴（系列）」**（实测 `B` = 填报内容），不是「类别轴」；类别轴那串字段**读不回来**；⑦ 「图例」开关写 `chart-info.showLegend`，同时会在属性表里**新建顶层 `legendStyle`**（`{"legendPosition":"top","fontStyle":{…}}`，键写进去删不掉，见陷阱 2）；⑧ 雷达图组件的 `properties` 里**没有** `yAxisStyle` / `allLabelStyle`（实测只有 `chart-info` + `ext-base-name`）；⑨ **`mapType` / `sortBy` / 嵌套 `chart-info.legendStyle` 三者已实测**：面板**没有**对应控件，但 API **可写且渲染会变**——A/B 实测写入 `{"mapType":"recordCategory","sortBy":"recordOrder"}` 并重载 → 雷达由「同心环 + 数值轴刻度」变成「按记录分布的多轴形态」，删掉这两键重载 → 还原（`mapType`：`fieldCategory` 默认 / `recordCategory`；`sortBy`：`categoryValue` 默认 / `recordOrder`）；**嵌套的 `chart-info.legendStyle` 由「图例位置」写入**（实测 顶部→底部：`chart-info.legendPosition` + `chart-info.legendStyle` + 顶层属性 `legendStyle` 三处同步），只拨「图例」开关只会写 `chart-info.showLegend` + 新建顶层 `legendStyle`；
- **矩阵气泡图专属（实测）**：① 「横轴」= 类别字段、「纵轴」= 数值字段，两者下方各有一个**值列表**（可拖动排序，最多展示 20 个选项）；② 存储键 = `chart-info.xAxisBaseField` / `yAxisBaseField`，statistics 侧对应 `statisticsModule.dimensionFieldId`（横轴）/ `groupFieldId`（纵轴）；③ 「气泡值」= 记录总数 / 指定字段（实测 `onlyCountDimension:true`）；④ 自定义配置 = 色彩元素（主题色 / 组件背景色 / 系列颜色[按纵轴数值逐项]）→ 图表元素（数据标签 / 图例 / **气泡大小**）→ **网格线**；⑤ 「气泡大小」的「最小/最大气泡半径」= `chart-info.minBubbleSize` / `maxBubbleSize`（实测 8→9 回读 `minBubbleSize:9`）；⑥ 「网格线」是**单个开关** → 写 `chart-info.splitLine`（实测 true→false），**没有**「刻度线 / 网格区域」子项；⑦ 数据标签**没有**「标签内容 / 标签位置」，只有「标签样式 / 显示单位 / 数字格式」（选 数值/百分比/货币 时多出「小数位数」「千位分隔符」，货币再多一个符号 `¥`/`€`）→ 写 `labelNumberUnit` / `labelNumberFormat` / `labelDecimalPlaces` / `labelShowThousandSeparator` / `labelCurrencySymbol`；⑧ 「图例」开关写 `chart-info.showLegend`，而**图例位置写在独立属性 `legendStyle`**（`chart-info` 里**没有** `legendPosition`）；⑨ 面板**没有**坐标轴样式区块（属性里的 `yAxisStyle` 写了不生效，同陷阱 6）；⑩ 基础配置**没有** 类型 / 分组（图例）/ 排序依据 / 排序方式 / 显示范围 / 空单元格显示为；
- **桑基图专属（实测）**：① 基础配置 = 标题 / 数据源 / 数据范围 / **节点类别**（**两个字段**：源节点 = `statisticsModule.dimensionFieldId`、目标节点 = `groupFieldId`；实测第二个字段 数量→日期 时 `groupFieldId:"C"→"D"`）/ **节点高度(值)**（记录总数 | 指定字段 → 实测 `onlyCountDimension:true→false`，指定字段时多出「字段 + 聚合」= `statisticSettingAddList`）/ **排序方式**（默认 | 自定义拖拽 → 实测 `sortType:"Custom"` + `sortByRow:false`；切回默认 → `"SortByHorizontalAxis"` + `true`）/ 明细数据显示的字段；② 自定义配置 = 色彩元素（主题色 / 组件背景色）→ **节点颜色**（按节点逐项）→ 图表元素（数据标签：标签内容 = **值 / 节点类别**，加标签样式）；**没有** 网格线 / 图例 / 效果元素 / 坐标轴区块；③ 数据标签是**平铺键**（`chart-info.showLabel` / `labelContent` / `labelFontStyle`，同雷达图写法），实测关掉「数据标签」→ `showLabel:true→false`；属性表里同时有 `allLabelStyle:{labelContent,fontStyle}`；④ 组件 `properties` 没有 `yAxisStyle` / `legendStyle`；统计配置与数据标签的明细**不落盘到可读字段**。
- **共现网络图专属（实测）**：① 基础配置 = 标题 / 数据源 / 数据范围 / **类型**（环形布局 / 自由布局）/ **节点类别**（两个字段）/ **连接线(边)**（记录总数 | 指定字段）/ 明细数据显示的字段；② 自定义配置 = 色彩元素（主题色 / 组件背景色）→ **节点颜色**（按节点逐项）→ 图表元素（数据标签[开关 + 标签内容 值/节点类别 + 标签样式] / 图例[开关 + 图例位置 顶部·底部·左侧·右侧 + 文本样式] / 节点大小[最小节点半径 / 最大节点半径]）；③ 实测映射：「类型」→ `chart-info.layoutType`（`circular` / `cola`）；「节点类别」→ `chart-info.firstNodeCategoryField` / `secondNodeCategoryField`（= statistics 的 `dimensionFieldId` / `groupFieldId`）；「连接线(边)」→ `statisticsModule.onlyCountDimension`；「节点大小」→ `chart-info.minNodeSize` / `maxNodeSize`；「数据标签」开关 → `chart-info.showLabel`（**并会新建顶层 `allLabelStyle`**）；④ ⚠️「图例位置」**同时写 `chart-info.legendPosition` 和 `legendStyle.legendPosition`**（实测切「底部」两处都变）——与矩阵气泡图相反；⑤ 组件 `properties` 没有 `yAxisStyle`。
- **词云图专属（实测）**：① 基础配置 = 标题 / 数据源 / 数据范围 / 类型（词云族只有「词云图」1 个变体）/ 关键词字段 / 统计方式 / 分割符号（条件出现）/ 展示个数 / 词频表 / 明细数据显示的字段，**没有** 分组（图例）、排序依据、排序方式、显示范围、空单元格显示为；② 自定义配置 = 色彩元素（主题色 / 组件背景色）→ 词云形状（8 种下拉），**没有** 图表元素 / 数据标签 / 图例 / 网格线 / 坐标轴区块；③ 写入映射：「统计方式」只改 `statisticsModule.onlyCountDimension`（关键词次数 = `true`；指定其他字段 / 按分割符 = `false`）；「词云形状」= `chart-info.shape`；「展示个数」= `chart-info.actualTextNum`（默认 50）；④ 「指定其他字段」的「字段 + 聚合」（聚合枚举：求和 / 计数 / 最大值 / 最小值 / 平均值 / 去重计数）与「按分割符」的「分割符号」（斜杠 / 分号 / 逗号 / 空格 / 其他 + 单字符输入）**不落盘到可读字段**，`dashboard_component_list` 的 `properties` / `statisticsModule` 里读不到（实测切换前后回读，只有 `onlyCountDimension` 变了），改完刷新页面 UI 状态仍在；「多项内容拆开统计」复选框（仅当关键词字段支持拆分时出现）同样**读不到**；⑤ 词云图**不渲染数据标签**，给词云图写 `allLabelStyle` / `showLabel` 没有意义。
- **统计数字（Value）专属（实测）**：① 基础配置 = 标题 / 数据源 / 数据范围 / **统计值说明**（`chart-info.valueLabel`）/ **统计值（纵轴）**（记录总数 | 指定字段 → `statisticsModule.onlyCountDimension`）/ 明细数据显示的字段；**没有**「类型」下拉、排序、分组、显示范围、空值显示方式；② 自定义配置 = 色彩元素（主题色 / 组件背景色）→ **统计元素**（数字格式 / 小数位数 / 千位分隔符）；**没有** 数据标签、图例、网格线、坐标轴（不要写 `yAxisStyle` / `allLabelStyle`）；③ ⚠️ **数字格式的键分两处**：「数字格式」→ `chart-info.formatType`（**7 档** `Num`/`Percent`/`RMB`/`Dollar`/`Euro`/`TenThousand`/`TenThousandYuan`），而「小数位数」+「千位分隔符」→ **独立属性 `valueFormatStyle`**（实测：改成 1位小数 + 显示 → `valueFormatStyle:{"decimalPlaces":"Decimal1","showThousandSeparator":true}`，`chart-info` 里没有这两个键，卡片渲染同步变 `28.0`）；④ 「主题色」→ `chart-info.colors`（`{name,value:[6 个 hsl],refIdx,version}`，`refIdx` 是色板下标，实测选第 3 个色块 → `refIdx:2`）；「组件背景色」仍是 `cardBgColor`/`cardBgOpacity`/`cardBgBlur` 三键；⑤ 用 API 新建的 Value 组件 `chart-info` 只有 `layout`/`name`/`type`——`valueLabel`/`formatType`/`valueFormatStyle` 都要自己写，别指望有默认值。
- **进度图（Progress）专属（实测）**：① 基础配置 = 标题 / **类型**（数值 | 日期）/ 数据源 / 数据范围 / **图表类型**（条形 / 半环 / 环形）/ **目标值** / **当前值**（自定义值 | 选择字段）/ 明细数据显示的字段；切到「日期」后只剩 标题 / 类型 / 图表类型 / **日期范围** / **当前日期**，其余项不渲染；② 自定义配置 = 色彩元素（主题色 / 组件背景色）→ **当前/目标值**（数字格式 / 小数点位数 / 千位分隔符）→ **百分比设置**（小数点位数）；**没有** 数据标签、图例、网格线、坐标轴；③ 实测键名（全在 `chart-info` 里平铺）：`progressType`（`line` 条形 / `semicircle` 半环 / `circle` 环形）、`dataType`（`numberType`/`dateType`）、`targetValue`（默认 100）/ `currentValue`（默认 50）、`dateRange`（`thisMonth`/`thisQuarter`/`thisYear`/`custom`）+ `customDateRange`（如 `["2026-09-15","2026-09-29"]`）+ `currentDate`（`today`/`yesterday`/`custom`）、`statisticFormat`（**9 档**，比统计数字多 `JPY` 日元 / `GBP` 英镑）+ `statisticDecimalPlaces` + `percentDecimalPlaces` + `showThousandSeparator`；④ ⚠️ **进度图的数字格式与统计数字键名完全不同**：进度图写 `chart-info.statisticFormat` / `statisticDecimalPlaces` / `percentDecimalPlaces` / `showThousandSeparator`，**不是**统计数字那套 `formatType` + 独立 `valueFormatStyle`；⑤ 「主题色」→ `chart-info.progressColor`（**两态**：色板第一行 6 色写 **0 起下标**，实测第 3 个色块 → `2` 且渲染为紫 `#9670FF` = `colors.value[2]`；第二行 8 个标准色写**色值字符串**，实测红块 → `"#EB5451"`），不是 `colors` / `theme_style`；⑥ 缺省值（未设置时的表现）：`statisticFormat` = `Num`、`statisticDecimalPlaces` = `Integer`、`percentDecimalPlaces` = `Decimal2`、`showThousandSeparator` = false、`progressType` = 条形；日期态的百分比由「当前日期在所选区间内的占比」自动算出（区间=本月、今天=9/22 → 73%）。
- **轴样式一律写 `yAxisStyle`**：条形图的数值轴是横轴，但**存储键仍是 `yAxisStyle`**（实测：写 `yAxisStyle.unit=100` → 条形图横轴显「百」；写 `xAxisStyle` 是无效键）。**面板侧双向实测**：在条形图「自定义配置 > 图表元素 > 横轴 > 显示单位」改成「万」后回读，变的是 `yAxisStyle.leftYAxis.unit=10000`，`xAxisStyle` 原样不动 → 面板只写 `yAxisStyle`；
- **不要按其他组件的当前值推断"默认值"**：未设 `legendStyle` 时图例默认渲染在**顶部**（折线图/条形图实测）；某组件图例在底部，通常是被显式设过 `bottom`。改样式前先读 `properties` 取现值；
- `chart-info`、`allLabelStyle`、`yAxisStyle` 内部均为**整串覆盖**——改一个子键须带全同属性其它子键。

**通道二：statistics_module_setting（统计配置）**

| 字段 | 说明 |
|------|------|
| `dimensionFieldId` | 维度（横轴）字段 ID |
| `onlyCountDimension` | 统计值：`true` 记录总数 / `false` 指定字段 |
| `statisticSettingAddList` / `RemoveList` | 统计字段 `[{fieldId, statisticOptions, plotType?, yAxisIndex?}]`（**追加**）。**柱线图（组合图）**用 `plotType`（`Bar`/`Line`）指定该系列画柱/画线、`yAxisIndex`（`0` 左轴 / `1` 右轴）指定归属轴（实测新增系列 `{fieldId:"I", statisticOptions:"count", plotType:"Line", yAxisIndex:1}` → 新系列即为折线并落在右轴） |
| `groupFieldId` | 分组（图例）字段 ID。⚠️ **只能设置，无法清除**，清除需在页面操作；⚠️ **不能与 `dimensionFieldId` 相同**（`dim=J`+`group=J`、`dim=D`+`group=D` 实测均 `400100`），且富文本 / 附件 / 关联字段不可用作分组（同维度，见 create 附录 G） |
| `sortByRow` | 排序依据：`true` 横轴 / `false` 纵轴；**自动联动 `sortType`**（`SortByHorizontalAxis`/`SortByVerticalAxis`） |
| `ascending` | 排序方式：`true` 正序 / `false` 倒序 |
| `filterCriteriasAddList` | 筛选条件（**追加**） |
| `filterCriteriasRemoveList` | 删除筛选 `[{fieldId}]` |
| `clearFilter` | `true` 清空全部筛选条件；⚠️ **不清 `filtersOp`** |
| `filtersOp` | 条件关系：`"And"` / `"Or"` |
| `dataDetailVisibleFields` | ⚠️ **无效键**：明细数据显示的字段是**面板专属设置**，「公开 API 写不了、也读不到」（实测传对象 / 字符串 / 数字都返回成功，组件与面板毫无变化；回读也没有该键）→ 要改就请用户在面板上勾选 |
| `chartStyle` | 系列颜色 `{chartElementType:"Series", identifier:"{}", style:"{\"color\":\"#2E8B57\"}"}`（**只写不读**） |
| `onlyTopNDimensions` / `retainEqualNthDimension` | ⚠️ **实测写不进**：显示范围 TopN 只有柱状图 / 条形图有且需满足条件（见陷阱 8），但**公开 API 写不进去**——柱状图实测写 `5`（含与 `retainEqualNthDimension:true` 同写、先改排序依据再写）、以及 `0` / `1` / `20` / `21` 全部返回成功但**回读不到、组件无变化**，`""` / `null` 报 `400100` → 只能在面板选（见陷阱 26）|

**筛选条件写法（实测）**

⚠️ **筛选与组件类型无关**：`filter` 写在 `statisticsModule` 下、与图表类型无关，
14 个数据组件（柱状 / 条形 / 折线 / 面积 / 饼状 / 柱线 / 雷达 / 漏斗 / 矩阵气泡 / 桑基 / 关系 / 词云 /
**统计数字 / 进度图**）实测**全部生效**——同一条件 `Less` + `{type:"Text", value:"2026/09/20"}`
在各组件上都让数值同步下降，`clearFilter` 后全部还原；**统计数字 / 进度图** 虽 `dimensionFieldId` 为空
串、`onlyCountDimension:true`，筛选照常生效（`result.data` 28 → 4）。
❌ **例外**：明细表（表格组件，`webExtensionType` 为表格）不走本路径——组件载荷里**只有 `viewSetting`**、
没有 `statisticsModule`，其筛选应为视图/组件自带设置（未验证）。

`filterCriteriasAddList: [{fieldId, criteria:{op, values}}]`，算子全集（**不按字段类型校验**，写错不报错只会静默空结果）：

`Intersected` · `Equals` · `NotEqu` · `Contains` · `NotContains` · `BeginWith` · `EndWith` ·
`Greater` · `GreaterEqu` · `Less` · `LessEqu` · `GreaterEquAndLessEqu` · `LessOrGreater` · `Empty` · `NotEmpty`

`values[].type` 只有三种合法值，且**日期两种形态必须配对算子**：

| 场景 | 写法 |
|------|------|
| 文本/选项 | `{type:"Text", value:"好评"}` |
| 固定日期 · **等于某天** | `{type:"Date", year:2026, mon:9, mday:22}` + 算子 **`Intersected`**（`mon` 为 1-based） |
| 固定日期 · **比较/区间** | `{type:"Text", value:"2026/09/20"}`（**`yyyy/mm/dd`**，与面板保存的形态一致）+ 比较算子 |
| 动态日期 | `{type:"DynamicSimple", dynamicType:"last7Days"}` + 比较算子 |
| 空值判断 | `Empty` / `NotEmpty`，且 **`values` 必须传 `[]`**（传 `null` 条件会被静默丢弃） |

⚠️ **日期筛选两种形态不可互换（实测）**：`{type:"Date",year,mon,mday}` 只认 `Intersected`（当天）；
比较算子（`Less` / `LessEqu` / `Greater` / `GreaterEqu` / `GreaterEquAndLessEqu` / `LessOrGreater`）
必须写 `{type:"Text", value:"yyyy/mm/dd"}`。写反了（`Date` 配比较、或 `Text` 配 `Intersected`）
**返回成功但静默 0 命中**，是最常见的「筛选没生效」根因。

实测样例（柱线图组件，字段 `D`=日期，无筛选时总量 156.01）：`{"op":"Less","values":[{"type":"Text","value":"2026/09/20"}]}`
→ 64.01；补集 `{"op":"GreaterEqu","values":[{"type":"Text","value":"2026/09/21"}]}` → 92，两者相加恰好
等于无筛选总量 → 说明比较是按日期语义（而非字符串）比较的，可放心用于「某日之前 / 之后」。

`dynamicType` 合法值：`today` · `yesterday` · `tomorrow` · `last7Days` · `last30Days` ·
`last7DaysContainToday` · `last30DaysContainToday` · `thisWeek` · `thisMonth` · `thisYear` · `lastMonth`。
⚠️ 动态日期按**当前日期**换算（实测边界可能与本地时钟不一致），且 `Intersected` + `DynamicSimple`
实测**不成立**（`today` / `tomorrow` 都取不到当天）；要精确到某一天，用上面的固定日期写法而不是 `DynamicSimple`。

**陷阱清单**

1. `filterCriteriasAddList` / `statisticSettingAddList` 是**追加**语义 → 改条件前先 `clearFilter:true`；
2. `statistics_module_setting` 的字段**清不掉**（传 `""` 被忽略、传 `null` 报错）——`groupFieldId` 需在页面清除（实测传 `""` 静默成功但旧值仍在；`onlyTopNDimensions` 则是压根写不进去，见陷阱 26）；**「整个属性键」也删不掉**（实测给 `property_modify_list` 传 `value:""` 返回 `400100 InvalidArgument`），只能写回默认值（如 `"{}"`）——所以**别在正式组件上做属性试验**，写进去的键会永久留在 `properties` 里（写 `"{}"` 后渲染与「没这个键」一致，但键还在）；`property_modify_list` 的**属性内部子键**可以清（JSON 里删掉该子键即可），但**整个属性键写了就删不掉**（写空串被拒 `400100`），只能写回默认值；
3. **返回成功 ≠ 生效**（未知 / 无效参数会被静默忽略），必须回读或看渲染结果；判别「键是否有效」的实测方法：
   **真正的统计键传错类型会 `400100`**（实测 `onlyCountDimension:"abc"`、`dataSourceSheetId:"abc"`、`statisticSettingAddList:"abc"` 均报错），而**无效键一律返回成功**（实测 `dataDetailVisibleFields` 传对象 / 字符串 / 数字都成功、`fooBarBaz` 无论顶层还是嵌套也都成功）→ 只回成功且不做类型校验的键，基本可判定是被**静默丢弃的无效键**；
   ⚠️ 明细数据显示的字段就属于这一类：**面板专属**设置，公开 API 既写不了也读不到（实测写入后组件载荷与面板均无变化），只能让用户在面板上勾选；
4. 系列颜色、`statisticSettingAddList`、`chartStyle` **写后不可回读**，无法读-改-写（**例外**：柱线图的系列配置可从 `dashboard_chart_data` 的 `result.series[]` 读到 `name`/`statisticOption`/`plotType`/`yAxisIndex`/`numberFormat`，可据此读-改-写）；
5. 一次只动一项统计配置——统计值+分组+筛选同时改可能让图表渲染为空；
6. 部分图表类型不读 `yAxisStyle`（如矩阵气泡图、柱线图），写入不报错也无效果；
7. ⚠️ **属性表是自由 map**：`property_modify_list` 写**任意** key 都会被原样存进 `properties` 并回读成功（实测写 `zzBogusKeyXyz` 也能存能读）→ **"写成功 + 能回读" ≠ "属性有效"**，未知键会被渲染器忽略。判断属性真伪只能靠**渲染效果**（或在面板上操作后回读）；
8. **类型专属字段**（实测：柱状/条形/折线/面积/饼状/雷达/矩阵气泡图）——**折线图 / 面积图**有 `lineEffect`（面板「效果元素 > 效果」）/空值显示方式（面板「空单元格显示为」= `chart-info.blankType`）与 `stackType`（折线经「类型 → **堆积折线图**」选；面积图经「类型 → 基础面积图 / 堆积面积图 / **百分比堆积面积图**」选，取值为 `noStack` / `stack` / `percentStack` 三档），但**没有**「显示范围」；**柱状图/条形图**有 `stackType`（堆叠，经「类型 → 堆积柱状图 / 堆积条形图」选），都没有折线样式/空值显示方式——⚠️ **「显示范围」(TopN) 只有柱状图 / 条形图有，且是「条件出现」**：仅当「分组（图例）为空 + 排序依据=纵轴（按统计值排序）+ 统计值要么记录总数、要么指定字段 ≤1 个」时才渲染（浏览器实测：柱状图排序依据=横轴时无，改成纵轴后出现，再设分组又消失）；折线图 / 面积图 / 饼图 / 柱线图的配置代码里**根本没有** TopN，别写；**饼图**有 `chart-info.pieType`（`pie`/`ring`，经「类型 → 饼图 / 圆环图」选），**没有坐标轴**（不写 `yAxisStyle`）、没有网格线/堆叠/显示范围/分组（图例），配色面板叫「扇区颜色」。**雷达图**另有 `shape`/`chartStyle`/`valueAxisConfig`/`categoryAxisConfig`（**不写** `yAxisStyle`/`allLabelStyle`，详见陷阱 15）。改样式前先确认该类型是否支持，不要照搬其他类型；
9. **排序入口是通用的**：`sortByRow`（true 按类目轴 / false 按值轴）+ `ascending`，**饼图同样生效**（实测扇区顺序变化）——不要因为面板呈现不同就以为饼图要另走一套；
10. **柱线图（组合图）双轴写在同一个 `yAxisStyle` 里**：主轴=`leftYAxis`、次轴=`rightYAxis`，且**外层必须同时保留扁平字段**（`show`/`unit`/`format`/`showLabel`/`showAxisLine`）——只写 `leftYAxis` 或只写扁平层**都不生效**（实测）。完整可用形态：`{"show":true,"unit":1,"format":{"type":0,"config":{}},"showLabel":true,"showAxisLine":true,"leftYAxis":{"unit":100,…},"rightYAxis":{"unit":10000,…}}`（单位写在各自的 `leftYAxis.unit`/`rightYAxis.unit`；实测主轴显「百」、次轴显「万」）。⚠️ `mainYAxisStyle`/`mainYAxis_style`/`secondaryYAxisStyle`/`secondaryYAxis_style`/`subYAxisStyle` 均为**无效键**（能存下但渲染忽略，别用）；
11. **组合图的系列形态可读可写**：`statisticSettingAddList[].plotType`（`Bar`/`Line`）+ `.yAxisIndex`（`0` 左轴 / `1` 右轴）实测生效（新增 `{fieldId:"I",statisticOptions:"count",plotType:"Line",yAxisIndex:1}` → 新系列即折线且落右轴）；删除系列用 `statisticSettingRemoveList:[{fieldId}]`（实测干净移除）；改前先从 `dashboard_chart_data` 的 `result.series[]` 读现值。
12. **面板选项枚举（浏览器实测）**：数据标签内容——直角坐标图（柱状/条形/折线/面积/柱线）都是 **系列名称 / 分类名称 / 值**，饼图是 **分类名称 / 值 / 百分比**；数据标签位置——柱状图 **居中 / 顶部 / 外部 / 底部**、折线图 / 面积图 **顶部 / 居中 / 底部**（无「外部」）、饼图 **外部 / 内部**、雷达图 **顶部 / 底部 / 内部**；空单元格显示为（折线图 / 面积图）——**空距 / 零值 / 用直线连接数据点**（存的是 `chart-info.blankType` = 1 / 2 / 3）；图例位置——**顶部 / 底部 / 左侧 / 右侧**（四档）；显示单位——无 / 百 / 千 / 万 / 十万 / 百万 / 千万 / 亿 / 十亿 / 兆（与 `leftYAxis.unit` 十档一致）；显示范围——**全部数据 / 自定义范围**（仅柱状图 / 条形图，且需满足陷阱 8 的条件；自定义范围 1~20）；饼图排序依据面板文案——**扇区名称 / 扇区数值**（值仍是 `SortByHorizontalAxis` / `SortByVerticalAxis`）。
13. **饼图的数据标签位置/内容是「专属键」**：位置 = `allLabelStyle.pieLabelPosition`（`"outside"` / `"inside"`）、内容 = `allLabelStyle.pieLabelContent`（如 `["categoryName","value"]`）。浏览器实测：面板把「标签位置」从 外部 切到 内部 后回读，`allLabelStyle` 新增的正是这两个键，老的 `barLabelPosition` / `position` **原样不动且对饼图无效**；渲染确认 `pieLabelPosition:"inside"` 时标签绘制在扇区内部（外部时带引导线在外侧）→ 改饼图标签位置别再写 `barLabelPosition`。
14. **漏斗图（Funnel）的面板与键名都和直角坐标图不同**：维度叫「**漏斗层**」，下面跟一个「**漏斗层顺序**」拖拽列表——**层顺序在 API 侧读不到**（`properties`/`statisticsModule` 里没有对应字段，只有 `sortType:"Custom"`），要改顺序只能拖面板；类型 = `chart-info.funnelType`（`"base"` 普通漏斗 / `"convert"` 转化漏斗）；「累计是否包含下层数据」= `chart-info.funnelDataSum`（包含 `true` / 不包含 `false`）；**标签设置写在 `allLabelStyle` 的「分区块对象」里**：`funnelLayer`（漏斗层标签：名称/值）、`funnelTransLayer`（转化层标签：名称/百分比，**仅转化漏斗**）、`funnelTotalTransRate`（总转化率：名称/百分比），每块 `{show, labelContent, fontStyle}`；配色面板叫「**漏斗颜色**」（按层逐项）；面板**没有**坐标轴、网格线、排序依据/排序方式、分组（图例）、显示范围、数据标签位置；⚠️ 漏斗图**吃 `chart-info.showLegend`**（旧文档「无效、走 `legendStyle.visible`」**已推翻**）：面板「图例」开关写 `chart-info.showLegend` 并同时新建顶层 `legendStyle:{"legendPosition":"top"}`；A/B 实测置 `false` 重载后**图例消失**、置 `true` 重载后**图例出现**，`legendStyle` 里**没有** `visible` 键。另附**实测落盘证据**：逐个关标签开关会在 `allLabelStyle` 里**按块追加**——关「漏斗层标签」→ `funnelLayer:{show:false,labelContent:["categoryName","value"],fontStyle:{…}}`；关「总转化率」→ 追加 `funnelTotalTransRate:{show:false,labelContent:["categoryName","percent"],…}`；「类型」切转化漏斗后再关「转化层标签」→ 追加 `funnelTransLayer:{show:false,labelContent:["percent"],…}`（**转化层默认只有 `["percent"]`**，三块默认 `labelContent` 并不一致）。
15. **雷达图（Radar）的指向和其它图都不同**：「数值轴」= `chart-info.valueAxisConfig`（**不是** `yAxisStyle`）、「类别轴」= `chart-info.categoryAxisConfig`（面板只给文本样式）、数据标签 = 平铺的 `chart-info.showLabel`/`labelContent`/`labelPosition`/`labelFontStyle`（**不是** `allLabelStyle`；标签位置 = `top` 顶部 / `bottom` 底部 / **`inside` 内部**）、「网格线/刻度线/网格区域」= 平铺的 `chart-info.showSplitLine`/`showAxisLine`/`showSplitArea`（写 `gridElements` 不生效）、形状 = `chart-info.shape`（`circle`/`polygon`）、类型 = `chart-info.chartStyle`（`stroke` 雷达图 / `fill` 填充雷达图）；⚠️ `statisticsModule.dimensionFieldId` 在雷达图里是面板的**「数值轴（系列）」**（实测 `B` = 填报内容），而「类别轴」的多字段+聚合**读不回来**；基础配置没有 分组（图例）/排序依据/显示范围/空单元格显示为；「图例」开关除了 `chart-info.showLegend` 还会在属性表里**新建顶层 `legendStyle`**（键删不掉）。
16. **矩阵气泡图（MatrixBubble）的轴与标签另有一套**：「横轴」= 类别字段、「纵轴」= 数值字段（存储键 `chart-info.xAxisBaseField` / `yAxisBaseField`，statistics 侧对应 `dimensionFieldId`[横轴] / `groupFieldId`[纵轴]）；「气泡值」= 记录总数 / 指定字段（实测 `onlyCountDimension:true`）；「气泡大小」= `chart-info.minBubbleSize` / `maxBubbleSize`（实测 8→9 回读 `minBubbleSize:9`）；「网格线」是**单个开关** → 写 `chart-info.splitLine`（实测 true→false），**没有**「刻度线 / 网格区域」子项；数据标签**没有**「标签内容 / 标签位置」，「显示单位 / 数字格式 / 小数位数 / 千位分隔符 / 货币符号」全部写在 `chart-info.labelNumberUnit` / `labelNumberFormat` / `labelDecimalPlaces` / `labelShowThousandSeparator` / `labelCurrencySymbol`（`labelNumberUnit` 与 `leftYAxis.unit`、`labelNumberFormat` 与 `leftYAxis.format.type` 是同一套取值）；⚠️ 关掉「数据标签」除了 `chart-info.showLabel=false`，还会**新建 `allLabelStyle`**（`{"fontStyle":{…}}`，键删不掉，见陷阱 2）；「图例」开关写 `chart-info.showLegend`，而**图例位置在独立属性 `legendStyle`**（`chart-info` 里**没有** `legendPosition`）；属性里虽有 `yAxisStyle`，但面板**没有**坐标轴样式区块（写了不生效，见陷阱 6）；基础配置没有 类型 / 分组（图例）/ 排序依据 / 排序方式 / 显示范围 / 空单元格显示为。
17. **桑基图（Sankey）用「节点类别 + 节点高度 + 排序方式」，和其它图都不一样**：「节点类别」是**两个字段**（源=`statisticsModule.dimensionFieldId`、目标=`groupFieldId`，实测第二个字段 数量→日期 → `groupFieldId` C→D）；「节点高度(值)」= 记录总数 / 指定字段（实测 `onlyCountDimension` true→false，指定字段时多出「字段 + 聚合」两个下拉 = `statisticSettingAddList`）；「排序方式」只有 **默认 / 自定义拖拽** 两档（不是「排序依据 + 排序方式」），实测 `sortType` = `SortByHorizontalAxis`/`Custom` 且 `sortByRow` true/false 同步变；数据标签**平铺在 `chart-info` 里**（`showLabel`/`labelContent`/`labelFontStyle`，同雷达图），并且**同时**写 `allLabelStyle:{labelContent,fontStyle}`；组件 `properties` 里**没有** `yAxisStyle`/`legendStyle`；基础配置没有 类型 / 分组（图例）/ 显示范围 / 空单元格显示为，自定义配置没有 网格线 / 图例 / 坐标轴区块（配色面板叫「**节点颜色**」，按节点逐项）。
18. **共现网络图（CoGraph）不是「图的图」而是独立一套**：「类型」= 环形布局 / 自由布局（写入 `chart-info.layoutType` 的 `"circular"` / `"cola"`；下拉里两项只有图标、没文字，选中后框里才显示中文名）；「节点类别」= **两个字段**（`chart-info.firstNodeCategoryField` / `secondNodeCategoryField`，实测 B/C，对应 statistics 的 `dimensionFieldId`/`groupFieldId`）；「连接线(边)」= `onlyCountDimension`；「节点大小」= `chart-info.minNodeSize`/`maxNodeSize`（默认 8/20）；数据标签是平铺键（`showLabel`/`labelContent`/`labelFontStyle`），打开开关会**新建顶层 `allLabelStyle`**（陷阱 2 同款，键删不掉）；⚠️「图例位置」**写两处**（`chart-info.legendPosition` + `legendStyle.legendPosition`），改之前两处都要对齐，否则读到旧值；组件里**没有** `yAxisStyle`，也没有「分组（图例）」字段、排序、显示范围、空单元格显示为。
19. **词云图（WordCloud）只有三处能回读的写入**：`chart-info.type="WordCloud"`、`chart-info.shape`（词云形状）、`chart-info.actualTextNum`（展示个数，默认 50），加上 statistics 侧的 `onlyCountDimension`（「统计方式」，只有「关键词次数」为 `true`）。自定义配置**只有** 色彩元素（主题色 / 组件背景色）+ 词云形状两块——**没有** 图表元素 / 数据标签 / 图例 / 网格线 / 坐标轴，所以不要照搬其它类型的 `allLabelStyle` / `legendStyle` / `yAxisStyle`。「统计方式」选「指定其他字段」后出现的「字段 + 聚合」、选「按分割符」后出现的「分割符号」，这两项的明细**不落盘到可读字段**，`properties` / `statisticsModule` 里都看不到，API 侧读写不了。词云图对「关键词字段」的候选**只滤掉**关联类（`Link`/`OneWayLink`，含 `Object` 关联字段）、富文本（`Note`）、图片和附件（`Attachment`）——**复选框、单选、多选、日期、数字、文本、联系人、评分都保留**（实测新建一个复选框字段后确实出现在候选列表里，旧文档「复选框/单选出不来」的说法**已推翻**）。另外「**多项内容拆开统计**」复选框**已实测**：关键词字段选到支持拆分的多选字段（`supportSplitOption`）且统计方式为「关键词次数 / 指定其他字段」时，才出现在下拉框正下方；勾选后整页重载仍保持，但**API 读不到**（该开关不落盘到可读字段），别指望用 API 复现或校验。
20. **三种直角坐标图的「标签位置」是三个不同的键**：柱状图 `allLabelStyle.barLabelPosition`、条形图 `allLabelStyle.horizBarLabelPosition`、折线图 `allLabelStyle.lineLabelPosition`（饼图是 `pieLabelPosition`）——照搬 `barLabelPosition` 给条形图 / 折线图**不生效**。取值也不能望文生义：柱状图 居中 `inside` / 顶部 `insideTop` / 外部 `top` / 底部 `insideBottom`，条形图 居中 `inside` / 顶部 `insideRight` / 外部 `outside` / 底部 `insideLeft`，折线图只有 3 档（顶部 `top` / 居中 `inside` / 底部 `bottom`，**没有「外部」**）。
21. **统计数字（Value）的数字格式藏在两个地方**：「数字格式」写 `chart-info.formatType`，而「小数位数 / 千位分隔符」写**独立属性 `valueFormatStyle`**（`{"decimalPlaces":"Decimal1","showThousandSeparator":true}`，**不在 `chart-info` 里**）——只改 `formatType` 不会动小数位，反过来也一样（实测：两处分别写入、互不覆盖；卡片渲染同步变 `28.0`）。⚠️ 两个坑：① `formatType:"Percent"` 是**值 ×100 再加 %**（实测 28 → `2800%`）；② 把 `valueFormatStyle` 写进 `chart-info` **也能生效**（实测 `28.00`），但面板规范位置是独立属性，优先写属性。
22. **进度图（Progress）的键名与统计数字几乎全不一样，别混用**：进度图用 `chart-info.statisticFormat` / `statisticDecimalPlaces` / `percentDecimalPlaces` / `showThousandSeparator` 与 `progressColor`（**两态**：色板**第一行 6 色**写 **0 起下标**且顺序与 `chart-info.colors.value` **一致**——实测点第 3 个色块 → `2` → 渲染紫 `#9670FF`，默认 `0` = 蓝 `#2977FF`；**第二行 8 个标准色写色值字符串**——实测点红块 → `"#EB5451"`。两行语义不同，选色后建议回读确认），统计数字用 `chart-info.formatType` / 独立属性 `valueFormatStyle` / `chart-info.colors`；两者**都不写** `yAxisStyle`、`allLabelStyle`。另注意进度图「日期」态会隐藏数据源/数据范围/目标值/当前值等控件，且老组件上可能残留旧写法 `chart-info.valueFormatStyle:{decimalPlaces}`（面板已改平铺键，别照抄）。

23. **`400100 第三方服务错误：Internal error / Unknown enum` = 校验拒绝，不是服务故障**：字段类型不被该位置接受
    （富文本 / 附件 / 关联做维度·分组·统计值）、枚举非法（`filledCount` / `checkedCount` / `dayRange` 等 10 个
    未实现选项）、`groupFieldId` 与维度同字段，都会返回这个报错。**别当故障反复重试**，先对照 create
    「附录 G：字段类型支持矩阵」检查组合；同批写入合法组合成功即可佐证。
24. **字段类型决定「能选什么」**：维度 / 分组 / 统计值的候选与词云「关键词字段」同一套规则——排除
    富文本（`Note`）/ 附件（`Attachment`）/ 关联（`Object`）；统计选项按字段类型分档（数值类才有
    `sum`/`avg`/`max`/`min`，日期类才有 `minTime`/`maxTime`，其余只有 `count`/`uniqueCount`）。
    完整矩阵见 `dashboard_component_create` 附录 G；**筛选字段是例外**，所有类型都能写（失败只表现为 0 命中）。
25. **图表空白的第一嫌疑是 `onlyCountDimension:false` + 无统计系列**：实测把统计系列移除后只剩
    `onlyCountDimension:false`，图表直接渲染为空（`categories:null`），极易被误判成「筛选没生效」——
    排查顺序：① `onlyCountDimension` 与 `statisticSettingAddList` 是否匹配；② 分组字段是否为空字段
    （如按全空字段分组）；③ 最后才怀疑筛选条件。

26. **面板专属属性（公开 API 读不到也写不了，别硬试）**：下面这些在面板里能改，API 侧**没有任何通道**——
    实测传任意值都「返回成功」，但**组件载荷毫无变化**（明细字段另经面板核对：面板显示也不变；TopN 只验到载荷）。遇到这类需求要**如实告诉用户「这一步请在面板操作」**
    并给出路径，不要用「已改好」糊过去：

    | 属性 | 面板位置 |
    |------|----------|
    | 明细数据显示的字段（已确认有该控件的：折线图、词云图、统计数字、进度图；其他图族以面板是否出现该控件为准）| 基础配置 → 「明细数据显示的字段」下拉（全选 = 顶部「全部字段 (N)」）|
    | 显示范围 TopN（`onlyTopNDimensions` / `retainEqualNthDimension`）| 基础配置 → 「显示范围」（仅柱状图 / 条形图，且需满足陷阱 8 的条件才出现）|
    | 漏斗图「漏斗层顺序」| 基础配置 → 「漏斗层顺序」拖拽列表 |
    | 雷达图「类别轴」的多字段列表（含每字段聚合）| 基础配置 → 「类别轴」|
    | 词云图「统计方式 → 指定其他字段」的字段+聚合 /「按分割符」的分割符号 /「多项内容拆开统计」| 基础配置 → 「统计方式」区块 |
    | 明细表 / 表格组件（`webExtensionType` 不是 `DbDataSource`）| 组件自带的表格设置（本工具不覆盖，未验证）|

27. **清不掉、删不掉的怎么处理：删组件 + 重建**（实测可行的纯 API 绕过）：`groupFieldId` 传 `""` 会**静默成功但保留旧值**
    （`null` 报错），`property_modify_list` 写进去的属性键也删不掉（陷阱 2）。要回到「没设过」的状态：
    ① `dashboard_component_create` 按目标配置**新建**组件——实测新建出来的组件 `groupFieldId:""`、无 TopN、`properties` 干净；
    ② `dashboard_component_delete` 删掉旧组件（参数是**单数字符串** `web_extension_key`；写成复数 / 传数组会返回 `400001`）。
    ⚠️ 重建前先把**读不回来的属性**（系列颜色 / 统计字段列表，陷阱 4）按目标值写全，否则重建即丢失；
    ⚠️ 重建会换掉 `webExtensionKey`，有外部引用需同步更新。

#### 返回值说明

```json
{
  "detail": {
    "webExtension": {
      "webExtensionKey": "{F4FD07CD-9414-4EFB-873E-3E9ED5D69EF8}",
      "webExtensionType": "DbDataSource",
      "statisticsModule": { "id": "C", "dataSourceSheetId": 1 }
    }
  },
  "result": "ok"
}

```

| 字段 | 类型 | 说明 |
|------|------|------|
| `detail.webExtension.webExtensionKey` | string | 组件 key |
| `detail.webExtension.statisticsModule` | string | 更新后的统计配置 |
| `result` | string | ok 表示成功 |


---

## 10. dbsheet.dashboard_component_delete

#### 功能说明

删除仪表盘中的单个图表组件。`web_extension_key` 来自 `dbsheet.dashboard_component_list`。
只删一个组件；清空全部用 `dbsheet.dashboard_clear`。

#### 调用约束

- **前置检查**：`web_extension_key` 来自 `dbsheet.dashboard_component_list`，核对组件名称后再删。

**幂等性**：是

#### 调用示例

删除单个组件：

```json
{
  "file_id": "string",
  "dashboard_id": 6,
  "web_extension_key": "{E65A9EA9-D0D0-451D-988F-F46B1FD20EAD}"
}
```

#### 参数说明

- `url` (string, 三选一必填: `url` / `link_id` / `file_id`): 文档 URL
- `link_id` (string, 三选一必填: `url` / `link_id` / `file_id`): 分享链接 ID
- `file_id` (string, 三选一必填: `url` / `link_id` / `file_id`): 文件 ID
- `dashboard_id` (integer, 必填): 仪表盘 ID
- `web_extension_key` (string, 必填): 要删除的组件 key（{GUID}）

#### 返回值说明

```json
{
  "detail": {
    "webExtension": { "webExtensionKey": "{E65A9EA9-D0D0-451D-988F-F46B1FD20EAD}" }
  },
  "result": "ok"
}

```

| 字段 | 类型 | 说明 |
|------|------|------|
| `detail.webExtension.webExtensionKey` | string | 被删除的组件 key |
| `result` | string | ok 表示成功 |


---

## 11. dbsheet.dashboard_chart_list

#### 功能说明

查询仪表盘的所有图表清单（id、名称、类型、是否有效）。
返回的 `id` 是 `dbsheet.dashboard_chart_data` 取数的图表标识；组件增删改用 `dbsheet.dashboard_component_list` 的 webExtensionKey。

**幂等性**：是

#### 调用示例

列出仪表盘图表：

```json
{
  "file_id": "string",
  "dashboard_id": 6
}
```

#### 参数说明

- `url` (string, 三选一必填: `url` / `link_id` / `file_id`): 文档 URL
- `link_id` (string, 三选一必填: `url` / `link_id` / `file_id`): 分享链接 ID
- `file_id` (string, 三选一必填: `url` / `link_id` / `file_id`): 文件 ID
- `dashboard_id` (integer, 必填): 仪表盘 ID

#### 返回值说明

```json
{
  "detail": {
    "charts": [
      {
        "id": "{E65A9EA9-D0D0-451D-988F-F46B1FD20EAD}",
        "name": "月度销售额",
        "type": "BasicBar",
        "dataSourceType": "dbsheet",
        "valid": true
      }
    ]
  },
  "result": "ok"
}

```

| 字段 | 类型 | 说明 |
|------|------|------|
| `detail.charts[].id` | string | 图表 id（{GUID}），dashboard_chart_data 的入参 |
| `detail.charts[].name` | string | 图表名称 |
| `detail.charts[].type` | string | 图表类型（BasicBar/Value 等） |
| `detail.charts[].dataSourceType` | string | dbsheet=数据源为数据表 |
| `detail.charts[].valid` | boolean | 图表配置是否有效 |
| `result` | string | ok 表示成功 |


---

## 12. dbsheet.dashboard_chart_data

#### 功能说明

查询仪表盘图表的运行时统计数据，用于**回读验证**（核对组件统计数值是否正确）。
图表 id 来自 `dbsheet.dashboard_chart_list`；`charts` 与 `chart_ids` 二选一。

**幂等性**：是

#### 调用示例

按图表 id 列表取数：

```json
{
  "file_id": "string",
  "dashboard_id": 6,
  "chart_ids": [
    "{E65A9EA9-D0D0-451D-988F-F46B1FD20EAD}",
    "{F4FD07CD-9414-4EFB-873E-3E9ED5D69EF8}"
  ]
}
```

带额外筛选取数：

```json
{
  "file_id": "string",
  "dashboard_id": 6,
  "charts": [
    {
      "id": "{E65A9EA9-D0D0-451D-988F-F46B1FD20EAD}",
      "filter": {}
    }
  ]
}
```

#### 参数说明

- `url` (string, 三选一必填: `url` / `link_id` / `file_id`): 文档 URL
- `link_id` (string, 三选一必填: `url` / `link_id` / `file_id`): 分享链接 ID
- `file_id` (string, 三选一必填: `url` / `link_id` / `file_id`): 文件 ID
- `dashboard_id` (integer, 必填): 仪表盘 ID
- `chart_ids` (array, 可选): 图表 id 字符串数组（便捷入参），如 ["{GUID}"]；与 charts 二选一
- `charts` (array, 可选): 图表请求对象数组 [{id, filter?}]；filter 为**临时**额外筛选（不落盘），格式与 dashboard_component_update 的 filterCriteriasAddList 一致：[{fieldId, criteria:{op, values}}]，日期同样遵守「等于某天」={type:"Date",year,mon,mday}+`Intersected`、「比较/区间」={type:"Text",value:"yyyy/mm/dd"}+比较算子的配对规则（写反会静默 0 命中），动态日期用 {type:"DynamicSimple",dynamicType}；与 chart_ids 二选一，都传时以 charts 为准
- `lang` (object, 可选): 多语言配置 {yes, no, empty, count, unnamedRecord, series}；不传时取数数值不受影响

#### 返回值说明

```json
{
  "detail": {
    "charts": [
      {
        "id": "{E65A9EA9-D0D0-451D-988F-F46B1FD20EAD}",
        "valid": true,
        "dataSourceType": "dbsheet",
        "name": "总销售额",
        "type": "Value",
        "result": { "type": "Normal", "data": "1000.0" }
      },
      {
        "id": "{F4FD07CD-9414-4EFB-873E-3E9ED5D69EF8}",
        "valid": true,
        "dataSourceType": "dbsheet",
        "name": "月度销售额",
        "type": "BasicBar",
        "result": {
          "type": "Normal",
          "series": [ { "name": "销售额", "numberFormat": "0.00_ " } ],
          "categories": [
            { "name": "1月", "data": ["90"] },
            { "name": "2月", "data": ["83"] }
          ]
        }
      }
    ]
  },
  "result": "ok"
}

```

| 字段 | 类型 | 说明 |
|------|------|------|
| `detail.charts[].id` | string | 图表 id |
| `detail.charts[].valid` | boolean | 图表是否有效；false 时其余字段可能缺失 |
| `detail.charts[].type` | string | 图表类型 |
| `detail.charts[].result.type` | string | Normal 正常 / Empty 数据为空 / Invalid 无效 / DataIsProtected 数据受保护；非 Normal 时无 data |
| `detail.charts[].result.data` | string | 统计数字（指标卡）的统计值 |
| `detail.charts[].result.series` | array | 系列 [{name, numberFormat, plotType?, yAxisIndex?}] |
| `detail.charts[].result.categories` | array | 类别 [{name, data:[…], total, identifier, groupKey}]，空字符串表示无数据 |
| `detail.charts[].result.categories[].total` | string | 该类别合计数（字符串），**验证筛选/统计效果时读它最方便** |
| `detail.charts[].result.categories[].identifier` | string | 类别/系列标识（JSON 串），如 {"type":"option","value":{…}} 或 {"type":"none"}；对应 update 的 chartStyle.identifier |
| `detail.charts[].result.exceededLimitTypes` | array | 类别/系列因超限被截断的提示 |
| `result` | string | ok 表示成功 |

**结果解读**

- 指标卡（Value）：`result.data` 为统计值字符串（如 `"1000.0"`）。
- 普通图表：`result.categories[].data` 与 `result.series[]` 按下标对应；空字符串表示该点无数据。
- `result.type` 非 `Normal`（Empty / Invalid / DataIsProtected）时 result 无 data 字段，
  须如实告知用户原因（如 Empty=数据源无记录），不得编造数值。
- **验证筛选/统计是否生效**：加总 `categories[].total` 与改动前对比即可（无需截图）；
  数值变化说明条件已生效，此时再看页面渲染是否正常。
