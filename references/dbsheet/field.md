# 字段管理

## 1. dbsheet.create_fields

#### 功能说明

在指定数据表中批量创建字段。请求体为 JSON：`fields[]` 每项含 `name`、`type` 及类型相关的 `data`；详见 param_detail 中各字段类型定义。创建成功后由服务端分配字段 `id`，**创建请求中禁止手填 `id`**。



#### 操作约束

- **禁止**：创建请求中禁止手填 `id`，`id` 仅由服务端分配
- **后置验证**：get_schema 确认字段已创建

**幂等性**：否 — 重复调用会创建重复字段，先确认是否已成功

> 选项类字段的选项定义在 `data.items` 内；与旧文档中把 `items` 写在字段根级的写法不同，以本接口 JSON 为准。
> 身份证字段类型名为 `ID`；部分历史示例写作 `Id`，以平台校验为准。

#### 调用示例

创建多种类型字段：

```json
{
  "file_id": "string",
  "sheet_id": 3,
  "prefer_id": false,
  "fields": [
    {
      "name": "状态",
      "type": "SingleSelect",
      "data": {
        "allow_add_item_while_inputting": true,
        "items": [
          {
            "value": "待处理",
            "id": "1"
          },
          {
            "value": "进行中",
            "id": "2"
          },
          {
            "value": "已完成",
            "id": "3"
          }
        ]
      }
    },
    {
      "name": "截止日期",
      "type": "Date",
      "data": {
        "number_format": "yyyy/mm/dd",
        "default_value_type": "RecordCreateTime"
      }
    },
    {
      "name": "备注",
      "type": "MultiLineText",
      "data": {
        "unique_value": false
      }
    }
  ]
}
```


#### 参数说明

- `file_id` (string, 必填): 多维表格文件 ID（路径参数）
- `sheet_id` (integer, 必填): 数据表 ID
- `fields` (array, 必填): 待创建字段列表；每项为对象，至少含 `name`、`type`，多数类型需额外 `data`（结构因类型而异，见 param_detail）
  - `name` (string, 必填): 字段显示名称
  - `type` (string, 必填): 字段类型枚举（见 param_detail 完整列表；如 `Date`、`ID`、`Percent`/`Percentage` 等以平台校验为准）
  - `data` (object, 视类型必填/可选): 类型专属配置；无专属属性时可省略或 `{}`
  - **禁止**在创建请求中传入 `id`：`id` 仅创建成功后由服务端返回
- `prefer_id` (boolean, 可选): 默认 `false`（以字段**名称**解析关联）。为 `true` 时，**Lookup** 的 `link_field`/`lookup_field`、**LastModifiedBy**/**LastModifiedTime** 的 `watched_field` 等须传**字段 id**

**请求详情**

| 项目 | 值 |
|------|-----|
| Method | `POST` |
| Content-Type | `application/json` |

**请求体根级**

| 名称 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `fields` | array[object] | 是 | 每项：`name`、`type`、可选 `data`（见下文各类型） |
| `prefer_id` | boolean | 否 | 默认 `false`。`true` 时 Lookup / 监控类字段中的引用须用**字段 id** |

**`fields[]` 通用属性（创建）**

| 属性 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `name` | string | 是 | 字段显示名称 |
| `type` | string | 是 | 字段类型（下列各节枚举） |
| `data` | object | 视类型 | 该类型的专属配置；无配置时可省略或 `{}` |
| `id` | string | **禁止** | 仅创建后由服务端返回，**不可**在 CreateField 中手填 |

以下为各 **`type`** 在 **`data`** 中的要求及录入值类型说明（**CreateRecord** 仅帮助理解字段语义；写记录请用 `dbsheet.create_records`）。

---

**1. `Date` 日期**

| `data` 键 | 类型 | 说明 |
|-------------|------|------|
| `number_format` | string | 日期显示格式（Excel 风格数字格式串） |
| `default_value_type` | string | `RecordCreateTime` 记录创建时间；`Normal` 指定默认日期；不传则不自动填 |
| `default_value` | string | 当 `default_value_type` 为 `Normal` 时必填，如 `"2024/11/23"` |

录入值：符合 `number_format` 的日期字符串。

---

**2. `Time` 时间**

| `data` 键 | 类型 | 说明 |
|-------------|------|------|
| `number_format` | string | 时间格式，如 `hh:mm:ss;@`、`[$-409]h:mm:ss AM/PM;@` |

录入值：符合格式的时间字符串。

---

**3. `Number` 数字**

| `data` 键 | 类型 | 说明 |
|-------------|------|------|
| `number_format` | string | 数字显示格式，如 `"0_ "`、`"#,##0.000_ "` |

录入值：`int` / `float`。

---

**4. `Currency` 货币**

| `data` 键 | 类型 | 说明 |
|-------------|------|------|
| `number_format` | string | 货币格式，如 `"$#,##0.000_ "` |

录入值：`int` / `float`。

---

**5. `MultiLineText` 多行文本**

| `data` 键 | 类型 | 说明 |
|-------------|------|------|
| `unique_value` | boolean | 是否禁止重复录入 |

录入值：string。

---

**6. `Percent` / `Percentage` 百分比**

类型名以平台为准（文档中常见 `Percent` 与 `Percentage` 两种写法）。

| `data` 键 | 类型 | 说明 |
|-------------|------|------|
| `number_format` | string | 如 `"0.00%"` |

录入值：`int` / `float`（与格式一致的百分比数值）。

---

**7. `ID` 身份证**

| `data` 键 | 类型 | 说明 |
|-------------|------|------|
| `unique_value` | boolean | 是否禁止重复 |

录入值：string。

---

**8. `Phone` 电话**

| `data` 键 | 类型 | 说明 |
|-------------|------|------|
| `unique_value` | boolean | 是否禁止重复 |

录入值：string。

---

**9. `LastModifiedBy` 最后修改者**

| `data` 键 | 类型 | 说明 |
|-------------|------|------|
| `watch_all` | boolean | 是否监控所有字段，默认 `true` |
| `watched_field` | string[] | `watch_all=false` 时**必填**：被监控字段的 **id** 列表（与 `prefer_id` 一致时用 id） |

系统字段：**CreateRecord 不支持手填**，由系统自动维护。

---

**10. `LastModifiedTime` 最后修改时间**

| `data` 键 | 类型 | 说明 |
|-------------|------|------|
| `watch_all` | boolean | 默认 `true` |
| `watched_field` | string[] | `watch_all=false` 时必填，字段 id 数组 |
| `number_format` | string | 显示格式 |

录入值：时间字符串。系统字段：**CreateRecord 一般不手填**（以产品行为为准）。

---

**11. `Formula` 公式**

| `data` 键 | 类型 | 说明 |
|-------------|------|------|
| `formula` | string | 公式串，如 `"=[单件盈利]*[件数]"`（列名用 `[]`） |
| `number_format` | string | 结果展示格式 |
| `value_type` | string | **仅响应**：如 `Fvt_number`、`Fvt_Text`；公式引用文本列时常用 `Fvt_Text` |

**CreateRecord**：自动计算，**不支持手填**。

---

**12. `AutoNumber` 编号**

| `data` 键 | 类型 | 说明 |
|-------------|------|------|
| `number_format` | string | 如 `"000000"` 控制显示位数 |

**CreateRecord**：自动字段，**不支持手填**。

---

**13. `CreatedTime` 创建时间**

| `data` 键 | 类型 | 说明 |
|-------------|------|------|
| `number_format` | string | 如 `yyyy-mm-dd hh:mm;@` |

**CreateRecord**：自动字段，**不支持手填**。

---

**14. `Link` 关联**

| `data` 键 | 类型 | 说明 |
|-------------|------|------|
| `link_sheet` | integer | 关联数据表 id |
| `link_view` | string | 关联表视图 id |
| `is_auto` | boolean | 是否自动关联 |
| `multiple_links` | boolean | 是否允许多条关联 |
| `filter` | object | 可选；自动关联条件，结构见下 **FieldLinkFilter** |

**FieldLinkFilter**

| 键 | 类型 | 说明 |
|----|------|------|
| `mode` | string | 如 `"And"` |
| `conditions` | array | 每项可含 `current_sheet_field_id`、`link_sheet_field_id` 等 |

录入值：关联记录 id 的字符串数组 `[]string`。

---

**15. `Lookup` 引用**

| `data` 键 | 类型 | 说明 |
|-------------|------|------|
| `link_field` | string | 对应 **Link** 字段的 id |
| `lookup_field` | string | 被引用表中的字段 id |
| `aggregation` | string | `ToString`、`Origin`、`Sum`、`Counta`、`Average`、`Max`、`Min`、`Unique`、`CountaUnique` 等 |
| `base_type` | string | 引用语义分类；如 `"1"` 引用字段 / `"MultiLineText"` 等与开放文档一致 |
| `filter` | object | 可选；**统计 / 查找**类可与 `Link` 相同结构的 FieldLinkFilter |
| `lookup_sheet_id` | integer | 与 `link_field` 常互斥：直接指定被引用表时可不传 `link_field` |

**CreateRecord**：自动字段，**不支持手填**。

---

**16. `Url` 超链接**

| `data` 键 | 类型 | 说明 |
|-------------|------|------|
| `display_text` | string | 按钮文案；不填则非按钮模式展示 |

录入值：对象 `{ "address": string, "displayText": string }`（`displayText` 在非按钮模式下为显示文本）。

---

**17. `SingleSelect` 单选项**

| `data` 键 | 类型 | 说明 |
|-------------|------|------|
| `items` | array | **FieldSelectItem**：`value`（必填）、`color`（可选 int）、`id`（**仅响应**或预置时可选） |
| `allow_add_item_while_inputting` | boolean | 是否允许输入时新增选项 |

录入值：string（选中项的 `value`）。

---

**18. `MultipleSelect` 多选项**

同 SingleSelect 的 `items` / `allow_add_item_while_inputting`。

录入值：`string[]`（多个选项 `value`）。

---

**19. `Rating` 等级**

| `data` 键 | 类型 | 说明 |
|-------------|------|------|
| `max` / `max_value` | integer | 等级上限；不同环境字段名可能为 `max` 或 `max_value`，与 **get_schema 返回**保持一致 |

录入值：int。

---

**20. `Contact` 联系人**

| `data` 键 | 类型 | 说明 |
|-------------|------|------|
| `default_value_type` | string | `RecordCreator` 记录创建者；`Normal` 搭配 `default_value` 填指定用户 uid |
| `default_value` | string | `Normal` 时：用户 uid |
| `multiple_contacts` | boolean | 是否支持多联系人 |
| `notice_new_contact` | boolean | 是否通知联系人 |
| `support_multi` | boolean | **仅响应**；创建时可不传 |

录入值：对象数组，元素含 `id`、`nickname`、`avatar_url` 等。

---

**21. `Attachment` 附件**

| `data` 键 | 类型 | 说明 |
|-------------|------|------|
| `only_upload_by_camera` | boolean | 是否仅允许拍照上传 |

录入值：对象数组（`uploadId`、`fileName`、`size`、`source`、`type`、`linkUrl`、`imgSize` 等）。

---

**22. `Address` 地址**

| `data` 键 | 类型 | 说明 |
|-------------|------|------|
| `address_level` | int | 层级：1 省 … 5 省/市/区/街道/社区 |
| `detailed_address` | boolean | 是否要求额外详细地址 |
| `preset_address` | object | `detail`（详细）、`districts`（与层级对应的地区名数组） |

录入值：`{ "districts": string[], "detail": string }`。

---

**23. `Note` 富文本**

无专属 `data` 配置。

录入值：`{ "fileId", "summary", "modifyDate" }`（以开放文档为准）。

---

**24. `Checkbox` 复选框**

无专属 `data`。

录入值：boolean。

---

**25. `Complete` 进度**

无专属 `data`。

录入值：int。

---

**26. `Email` 邮箱**

无专属 `data`。

录入值：string。

---

**27. `CreatedBy` 创建人**

无专属 `data`。

**CreateRecord**：自动字段，**不需要传入**；返回含 `id`、`nickName` 等。

---

**其它**

- **`SingleLineText`** 等未在上表展开的类型：以开放文档及 `get_schema` 返回为准。
- 响应中可能含 **`data.value_type`**（公式等）、**`default_value_array`**（联系人）等仅响应字段，创建时勿强行照搬。

**请求体节选示例**

```json
{
  "fields": [
    { "name": "单选项", "type": "SingleSelect", "data": { "allow_add_item_while_inputting": true, "items": [{ "value": "选项1" }] } },
    { "name": "关联", "type": "Link", "data": { "link_sheet": 12, "link_view": "Y", "is_auto": true, "multiple_links": true } }
  ],
  "prefer_id": false
}
```


#### 返回值说明

```json
{
  "detail": {
    "fields": [
      {
        "id": "K",
        "name": "状态",
        "type": "SingleSelect",
        "items": [
          { "id": "E", "value": "待处理" },
          { "id": "F", "value": "进行中" },
          { "id": "G", "value": "已完成" }
        ]
      },
      { "id": "L", "name": "截止日期", "type": "Date" }
    ]
  },
  "result": "ok"
}

```

| 字段 | 类型 | 说明 |
|------|------|------|
| `detail.fields[].id` | string | 新建字段 ID |
| `detail.fields[].name` | string | 字段名称 |
| `detail.fields[].type` | string | 字段类型 |
| `detail.fields[].items` | array | 选项列表（选项类字段） |
| `result` | string | ok 表示成功 |


---

## 2. dbsheet.update_fields

#### 功能说明

批量更新数据表中已有字段的名称、选项等属性。每项更新必须包含字段 ID。


#### 操作约束

- **前置检查**：get_schema 确认目标字段存在及当前属性

**幂等性**：是

> 更新字段时，`id` 为必填项（与创建字段相反，创建时禁止传入 `id`）；可通过 get_schema 获取字段 id。
> 选项类字段（SingleSelect / MultipleSelect）更新 `items` 时，含 `id` 的项为更新，不含 `id` 的项为新增，未出现的 `id` 对应选项会被删除。

#### 调用示例

更新日期字段格式：

```json
{
  "file_id": "abc123",
  "sheet_id": 1,
  "fields": [
    {
      "id": "q",
      "name": "日期",
      "type": "Date",
      "data": {
        "number_format": "yyyy\"年\"m\"月\"d\"日\";@",
        "default_value_type": "Normal",
        "default_value": "2024/11/23"
      }
    }
  ],
  "prefer_id": true
}
```


#### 参数说明

- `file_id` (string, 必填): 多维表格文件 ID（路径参数）
- `sheet_id` (integer, 必填): 目标数据表 ID
- `fields` (array, 必填): 待更新字段列表；每项为对象，必须含 `id`，其余可更新属性与创建字段一致（见 param_detail）
  - `id` (string, **必填**): 目标字段 ID（通过 get_schema 获取）
  - `name` (string, 可选): 更新后的字段显示名称
  - `type` (string, 可选): 字段类型，更新时一般与原类型一致
  - `data` (object, 视类型可选): 类型专属配置，结构与创建字段相同（见 param_detail）
- `prefer_id` (boolean, 可选): 默认 `false`（以字段**名称**解析关联）。为 `true` 时，**Lookup** 的 `link_field`/`lookup_field`、**LastModifiedBy**/**LastModifiedTime** 的 `watched_field` 等须传**字段 id**

**请求详情**

| 项目 | 值 |
|------|-----|
| Method | `POST` |
| Content-Type | `application/json` |

**请求体根级**

| 名称 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `fields` | array[object] | 是 | 每项：`id`（必填）、`name`、`type`、可选 `data`（见下文各类型） |
| `prefer_id` | boolean | 否 | 默认 `false`。`true` 时 Lookup / 监控类字段中的引用须用**字段 id** |

**`fields[]` 通用属性（更新）**

| 属性 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `id` | string | **是** | 目标字段 ID，通过 get_schema 获取 |
| `name` | string | 否 | 字段显示名称 |
| `type` | string | 否 | 字段类型，通常与原类型一致 |
| `data` | object | 视类型 | 类型专属配置，结构与 create_fields 相同（见下文） |

以下为各 **`type`** 在 **`data`** 中的配置说明，与 `dbsheet.create_fields` 的 param_detail 一致。

---

**1. `Date` 日期**

| `data` 键 | 类型 | 说明 |
|-------------|------|------|
| `number_format` | string | 日期显示格式（Excel 风格数字格式串） |
| `default_value_type` | string | `RecordCreateTime` 记录创建时间；`Normal` 指定默认日期；不传则不自动填 |
| `default_value` | string | 当 `default_value_type` 为 `Normal` 时必填，如 `"2024/11/23"` |

---

**2. `Time` 时间**

| `data` 键 | 类型 | 说明 |
|-------------|------|------|
| `number_format` | string | 时间格式，如 `hh:mm:ss;@`、`[$-409]h:mm:ss AM/PM;@` |

---

**3. `Number` 数字**

| `data` 键 | 类型 | 说明 |
|-------------|------|------|
| `number_format` | string | 数字显示格式，如 `"0_ "`、`"#,##0.000_ "` |

---

**4. `Currency` 货币**

| `data` 键 | 类型 | 说明 |
|-------------|------|------|
| `number_format` | string | 货币格式，如 `"$#,##0.000_ "` |

---

**5. `MultiLineText` 多行文本**

| `data` 键 | 类型 | 说明 |
|-------------|------|------|
| `unique_value` | boolean | 是否禁止重复录入 |

---

**6. `Percent` / `Percentage` 百分比**

| `data` 键 | 类型 | 说明 |
|-------------|------|------|
| `number_format` | string | 如 `"0.00%"` |

---

**7. `ID` 身份证**

| `data` 键 | 类型 | 说明 |
|-------------|------|------|
| `unique_value` | boolean | 是否禁止重复 |

---

**8. `Phone` 电话**

| `data` 键 | 类型 | 说明 |
|-------------|------|------|
| `unique_value` | boolean | 是否禁止重复 |

---

**9. `LastModifiedBy` 最后修改者**

| `data` 键 | 类型 | 说明 |
|-------------|------|------|
| `watch_all` | boolean | 是否监控所有字段，默认 `true` |
| `watched_field` | string[] | `watch_all=false` 时**必填**：被监控字段的 **id** 列表 |

---

**10. `LastModifiedTime` 最后修改时间**

| `data` 键 | 类型 | 说明 |
|-------------|------|------|
| `watch_all` | boolean | 默认 `true` |
| `watched_field` | string[] | `watch_all=false` 时必填，字段 id 数组 |
| `number_format` | string | 显示格式 |

---

**11. `Formula` 公式**

| `data` 键 | 类型 | 说明 |
|-------------|------|------|
| `formula` | string | 公式串，如 `"=[单件盈利]*[件数]"`（列名用 `[]`） |
| `number_format` | string | 结果展示格式 |

---

**12. `AutoNumber` 编号**

| `data` 键 | 类型 | 说明 |
|-------------|------|------|
| `number_format` | string | 如 `"000000"` 控制显示位数 |

---

**13. `CreatedTime` 创建时间**

| `data` 键 | 类型 | 说明 |
|-------------|------|------|
| `number_format` | string | 如 `yyyy-mm-dd hh:mm;@` |

---

**14. `Link` 关联**

| `data` 键 | 类型 | 说明 |
|-------------|------|------|
| `link_sheet` | integer | 关联数据表 id |
| `link_view` | string | 关联表视图 id |
| `is_auto` | boolean | 是否自动关联 |
| `multiple_links` | boolean | 是否允许多条关联 |
| `filter` | object | 可选；自动关联条件（FieldLinkFilter） |

---

**15. `Lookup` 引用**

| `data` 键 | 类型 | 说明 |
|-------------|------|------|
| `link_field` | string | 对应 **Link** 字段的 id |
| `lookup_field` | string | 被引用表中的字段 id |
| `aggregation` | string | `ToString`、`Origin`、`Sum`、`Counta`、`Average`、`Max`、`Min`、`Unique`、`CountaUnique` 等 |
| `base_type` | string | 引用语义分类 |
| `filter` | object | 可选；统计 / 查找条件 |
| `lookup_sheet_id` | integer | 直接指定被引用表时可不传 `link_field` |

---

**16. `Url` 超链接**

| `data` 键 | 类型 | 说明 |
|-------------|------|------|
| `display_text` | string | 按钮文案；不填则非按钮模式 |

---

**17. `SingleSelect` 单选项**

| `data` 键 | 类型 | 说明 |
|-------------|------|------|
| `items` | array | **FieldSelectItem**：`value`（必填）、`color`（可选 int）、`id`（含 id 表示更新已有选项，不含则新增） |
| `allow_add_item_while_inputting` | boolean | 是否允许输入时新增选项 |

---

**18. `MultipleSelect` 多选项**

同 SingleSelect 的 `items` / `allow_add_item_while_inputting`。

---

**19. `Rating` 等级**

| `data` 键 | 类型 | 说明 |
|-------------|------|------|
| `max` / `max_value` | integer | 等级上限；与 get_schema 返回字段名保持一致 |

---

**20. `Contact` 联系人**

| `data` 键 | 类型 | 说明 |
|-------------|------|------|
| `default_value_type` | string | `RecordCreator` 记录创建者；`Normal` 搭配 `default_value` 填指定用户 uid |
| `default_value` | string | `Normal` 时：用户 uid |
| `multiple_contacts` | boolean | 是否支持多联系人 |
| `notice_new_contact` | boolean | 是否通知联系人 |

---

**21. `Attachment` 附件**

| `data` 键 | 类型 | 说明 |
|-------------|------|------|
| `only_upload_by_camera` | boolean | 是否仅允许拍照上传 |

---

**22. `Address` 地址**

| `data` 键 | 类型 | 说明 |
|-------------|------|------|
| `address_level` | int | 层级：1 省 … 5 省/市/区/街道/社区 |
| `detailed_address` | boolean | 是否要求额外详细地址 |
| `preset_address` | object | `detail`（详细）、`districts`（与层级对应的地区名数组） |

---

**其它类型**（`Note` 富文本、`Checkbox` 复选框、`Complete` 进度、`Email` 邮箱、`CreatedBy` 创建人、`SingleLineText` 等）无专属 `data` 配置，以 get_schema 返回及开放文档为准。

**请求体节选示例**

```json
{
  "fields": [
    { "id": "q", "name": "日期", "type": "Date", "data": { "number_format": "yyyy\"年\"m\"月\"d\"日\";@", "default_value_type": "Normal", "default_value": "2024/11/23" } },
    { "id": "E", "name": "优先级", "type": "SingleSelect", "data": { "items": [{ "id": "B", "value": "低" }, { "id": "H", "value": "中" }, { "value": "紧急" }] } }
  ],
  "prefer_id": true
}
```


#### 返回值说明

```json
{
  "code": 0,
  "msg": "",
  "data": {
    "fields": [
      {
        "name": "日期",
        "type": "Date",
        "id": "q",
        "data": {
          "default_value": "2024/11/23",
          "default_value_type": "Normal",
          "number_format": "yyyy\"年\"m\"月\"d\"日\";@"
        }
      }
    ]
  }
}

```

| 字段 | 类型 | 说明 |
|------|------|------|
| `code` | integer | 响应代码，非 0 表示失败 |
| `msg` | string | 响应信息 |
| `data.fields` | array | 更新后的字段列表，详见多维表格参数说明 |
| `more` | object | 更多的错误信息 |


---

## 3. dbsheet.delete_fields

#### 功能说明

批量删除数据表中的指定字段。


#### 操作约束

- **前置检查**：get_schema 核对拟删字段的名称和类型
- **用户确认**：删除字段不可恢复，字段数据将永久丢失，必须向用户确认字段列表

**幂等性**：是

#### 调用示例

删除多个字段：

```json
{
  "file_id": "string",
  "sheet_id": 3,
  "fields": [
    {
      "id": "C"
    },
    {
      "id": "D"
    }
  ]
}
```


#### 参数说明

- `file_id` (string, 必填): 多维表格文件 ID
- `sheet_id` (integer, 必填): 目标数据表 ID
- `fields` (array, 必填): 要删除的字段列表，每项包含 `id`

#### 返回值说明

```json
{
  "detail": {
    "fields": [
      { "id": "C", "deleted": true },
      { "id": "D", "deleted": true }
    ]
  },
  "result": "ok"
}

```

| 字段 | 类型 | 说明 |
|------|------|------|
| `detail.fields` | array | 删除结果列表，每项包含 `id` 和 `deleted` |
| `result` | string | ok 表示成功 |


---

