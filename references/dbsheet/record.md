# 记录操作

## 1. dbsheet.create_records

#### 功能说明

在指定数据表中批量创建记录。每条记录通过 `fields_value` 字段传入一个**序列化的 JSON 字符串**，
字符串内部为字段名（或字段 ID）到值的映射。



#### 操作约束

- **后置验证**：调用 list_records 或 get_record 确认记录已创建
> `fields_value` 必须是序列化后的 JSON 字符串，直接传对象会导致参数解析失败
> prefer_id=true 时，fields_value 内部的 key 应为字段 ID（由 get_schema 返回），而非字段名
> 响应体中记录的字段值通过 `records[].fields` 返回（非 `fields_value`），包含创建后所有字段值（含系统自动字段）
> Url 字段传入为对象 `{address, displayText}`，响应中以数组形式返回
> Rating 字段的上限由 `max_value`（非 `max`）定义，创建字段时通过 `dbsheet.create_fields` 设置

#### 调用示例

按字段名批量创建记录：

```json
{
  "file_id": "VsdfG0001234567",
  "sheet_id": 3,
  "prefer_id": false,
  "records": [
    {
      "fields_value": "{\"文本\":\"第一行文本\",\"日期\":\"2024/12/20\"}"
    },
    {
      "fields_value": "{\"文本\":\"第二行文本\",\"日期\":\"2024/12/21\"}"
    }
  ]
}
```

覆盖大部分可写字段类型创建记录：

```json
{
  "file_id": "VsdfG0001234567",
  "sheet_id": 3,
  "prefer_id": false,
  "records": [
    {
      "fields_value": "{\"日期\":\"2025/11/15\",\"时间\":\"11:12:15\",\"数字\":125,\"货币\":215,\"多行文本\":\"yesit'sright\",\"百分比\":98,\"身份证\":\"110101**************9\",\"电话\":\"18800000000\",\"超链接\":{\"address\":\"https://www.baidu.com\",\"displayText\":\"百度\"},\"单选项\":\"选项1\",\"多选项\":[\"多选选项1\",\"多选选项2\"],\"等级\":2,\"联系人\":[{\"id\":\"281653414\",\"nickname\":\"nickName1\"}],\"地址\":{\"districts\":[\"广东省\",\"珠海市\",\"香洲区\"],\"detail\":\"前岛环路金山软件园\"},\"富文本\":{\"fileId\":\"GEYDAMJTHE3DKMJYHE2DURKDINBUQS2ZGRAUGUKIKE\",\"summary\":\"富文本简略内容\",\"modifyDate\":\"2025/12/31\"}}"
    }
  ]
}
```


#### 参数说明

- `file_id` (string, 必填): 多维表格文件 ID（路径参数）
- `sheet_id` (integer, 必填): 数据表 ID（路径参数）
- `records` (array[object], 必填): 要创建的记录列表，每个元素包含 `fields_value` 字段
- `prefer_id` (boolean, 可选): 为 `true` 时 `fields_value` 内部使用字段 ID 作为 key；默认 `false`（使用字段名）

**records 元素结构：**

| 字段 | 类型 | 是否必填 | 说明 |
|------|------|----------|------|
| `fields_value` | string | 是 | 序列化的 JSON 字符串，内容为字段名（或字段 ID）→ 值的映射，需符合各字段类型规范 |

> `fields_value` 是一个 **JSON 字符串**，不是对象。调用时需先将字段映射序列化为字符串再传入。

**各字段类型的 fields_value 填写规范：**

| 字段名（类型） | 值类型 | 示例值 | 备注 |
|--------------|--------|-------|------|
| 文本类（SingleLineText / MultiLineText） | string | `"任务描述"` | — |
| 日期（Date） | string | `"2025/11/15"` | 须符合字段 `number_format` 格式 |
| 时间（Time） | string | `"11:12:15"` | 须符合字段时间格式 |
| 数字 / 货币 / 百分比（Number / Currency / Percent） | int \| float | `125` / `215` / `98` | — |
| 复选框（Checkbox） | boolean | `true` | — |
| 进度（Complete） | int | `100` | 进度值 0–100 |
| 等级（Rating） | int | `3` | 不超过字段 `max_value` 上限 |
| 单选项（SingleSelect） | string | `"选项1"` | 必须是已有选项的 `value` |
| 多选项（MultipleSelect） | string[] | `["选项1","选项2"]` | 每项均须是已有选项 `value` |
| 电话 / 邮箱 / 身份证（Phone / Email / ID） | string | `"18800000000"` | — |
| 超链接（Url） | object | `{"address":"https://…","displayText":"百度"}` | 响应中以数组形式返回 |
| 关联（Link） | string[] | `["recXXXXXX"]` | 关联记录的 id 数组 |
| 联系人（Contact） | object[] | `[{"id":"uid","nickname":"张三","avatar_url":"https://…"}]` | `id` 为用户 uid |
| 地址（Address） | object | `{"districts":["广东省","珠海市","香洲区"],"detail":"详细地址"}` | `districts` 层级与字段 `address_level` 一致 |
| 富文本（Note） | object | `{"fileId":"…","summary":"摘要","modifyDate":"2024/12/09 12:00:00"}` | — |
| 附件（Attachment） | object[] | `[{"uploadId":"…","fileName":"a.png","size":1024,"source":"Cloud","type":"image/png"}]` | 需先上传获得 `uploadId`；`linkUrl`、`imgSize` 选填 |
| 最后修改时间（LastModifiedTime） | string | `"2024/12/10 10:10:10"` | 通常由系统自动维护 |
| 公式 / 编号 / 创建时间 / 创建人 / 最后修改者 / 引用（Formula / AutoNumber / CreatedTime / CreatedBy / LastModifiedBy / Lookup） | — | **不可填写** | 自动字段，传入会被忽略或报错 |

字段类型定义及 `data` 配置可参考 `dbsheet.create_fields`（`param_detail` 各类型节）。

**请求体示例（覆盖大部分可写字段类型）**

```json
{
  "prefer_id": false,
  "records": [
    {
      "fields_value": "{\"日期\":\"2025/11/15\",\"时间\":\"11:12:15\",\"数字\":125,\"货币\":215,\"多行文本\":\"yesit'sright\",\"百分比\":98,\"身份证\":\"110101**************9\",\"电话\":\"18800000000\",\"超链接\":{\"address\":\"https://www.baidu.com\",\"displayText\":\"百度\"},\"单选项\":\"选项1\",\"多选项\":[\"多选选项1\",\"多选选项2\"],\"等级\":2,\"联系人\":[{\"id\":\"281653414\",\"nickname\":\"nickName1\"}],\"地址\":{\"districts\":[\"广东省\",\"珠海市\",\"香洲区\"],\"detail\":\"前岛环路金山软件园\"},\"富文本\":{\"fileId\":\"GEYDAMJTHE3DKMJYHE2DURKDINBUQS2ZGRAUGUKIKE\",\"summary\":\"富文本简略内容\",\"modifyDate\":\"2025/12/31\"}}"
    }
  ]
}
```

`fields_value` 反序列化后内容（各字段类型对应值）：

| 字段名 | 字段类型 | 值示例 |
|--------|----------|--------|
| `日期` | Date | `"2025/11/15"` |
| `时间` | Time | `"11:12:15"` |
| `数字` | Number | `125` |
| `货币` | Currency | `215` |
| `多行文本` | MultiLineText | `"yesit'sright"` |
| `百分比` | Percent | `98` |
| `身份证` | ID | `"110101**************9"` |
| `电话` | Phone | `"18800000000"` |
| `超链接` | Url | `{"address":"https://www.baidu.com","displayText":"百度"}` |
| `单选项` | SingleSelect | 已有选项的 `value` 字符串 |
| `多选项` | MultipleSelect | 已有选项 `value` 的字符串数组 |
| `等级` | Rating | 不超过字段 `max_value` 的整数 |
| `联系人` | Contact | `[{"id":"uid","nickname":"昵称","avatar_url":"…"}]` |
| `地址` | Address | `{"districts":["广东省","珠海市","香洲区"],"detail":"…"}` |
| `富文本` | Note | `{"fileId":"…","summary":"摘要","modifyDate":"2025/12/31"}` |
| `复选框` | Checkbox | `true` / `false` |
| `进度` | Complete | `0`–`100` 整数 |
| `邮箱` | Email | `"user@example.com"` |
| `附件` | Attachment | `[{"uploadId":"…","fileName":"…","size":0,"source":"Cloud","type":"image/png"}]` |

> `Formula`、`AutoNumber`、`CreatedTime`、`CreatedBy`、`LastModifiedBy`、`Lookup` 为系统自动字段，**无需传入**。


#### 返回值说明

```json
{
  "code": 0,
  "msg": "",
  "data": {
    "records": [
      {
        "fields": "{\"日期\":\"2025/11/15\",\"数字\":125,\"公式\":340,\"创建人\":{\"id\":\"280026893\",\"nickName\":\"霧雨澪音\"},\"创建时间\":\"2024/12/09 17:47:18\",\"最后修改者\":{\"id\":\"280026893\",\"nickName\":\"霧雨澪音\"},\"最后修改时间\":\"2024/12/09 17:47:18\",\"编号\":2}",
        "id": "V"
      }
    ]
  }
}

```

| 字段 | 类型 | 说明 |
|------|------|------|
| `code` | integer | 0 表示成功 |
| `data.records[].id` | string | 新建记录 ID |
| `data.records[].fields` | string | 序列化 JSON 字符串，包含创建后所有字段的实际值（含系统自动字段） |


---

## 2. dbsheet.update_records

#### 功能说明

批量更新数据表中已有记录的字段值。每条记录必须提供 `id`（记录 ID）和 `fields_value`
（序列化的 JSON 字符串，内容为字段名或字段 ID 到新值的映射）。



#### 操作约束

- **前置检查**：调用 list_records 或 get_record 确认目标记录 ID 存在及当前字段值
- **后置验证**：调用 get_record 确认字段已更新为预期值
> `fields_value` 必须是序列化后的 JSON 字符串，直接传对象会导致参数解析失败
> 仅传入需要修改的字段即可，未传字段保持原值不变
> prefer_id=true 时，fields_value 内部的 key 应为字段 ID（由 get_schema 返回），而非字段名
> 响应体中记录的字段值通过 `records[].fields` 返回（非 `fields_value`），包含更新后所有字段值（含系统自动字段）
> Url 字段传入为对象 `{address, displayText}`，响应中以数组形式返回
> Rating 字段的上限由 `max_value`（非 `max`）定义，创建字段时通过 `dbsheet.create_fields` 设置

#### 调用示例

按字段名批量更新记录：

```json
{
  "file_id": "VsdfG0001234567",
  "sheet_id": 3,
  "prefer_id": false,
  "records": [
    {
      "id": "G",
      "fields_value": "{\"文本\":\"新的文本\",\"日期\":\"2024/12/21\"}"
    },
    {
      "id": "H",
      "fields_value": "{\"文本\":\"另一行文本\",\"状态\":\"已完成\"}"
    }
  ]
}
```

覆盖大部分可写字段类型更新记录：

```json
{
  "file_id": "VsdfG0001234567",
  "sheet_id": 3,
  "prefer_id": false,
  "records": [
    {
      "id": "recXXXXXXXX",
      "fields_value": "{\"日期\":\"2025/11/15\",\"时间\":\"11:12:15\",\"数字\":125,\"货币\":215,\"多行文本\":\"yesit'sright\",\"百分比\":98,\"身份证\":\"110101**************9\",\"电话\":\"18800000000\",\"超链接\":{\"address\":\"https://www.baidu.com\",\"displayText\":\"百度\"},\"单选项\":\"选项1\",\"多选项\":[\"多选选项1\",\"多选选项2\"],\"等级\":2,\"联系人\":[{\"id\":\"281653414\",\"nickname\":\"nickName1\"}],\"地址\":{\"districts\":[\"广东省\",\"珠海市\",\"香洲区\"],\"detail\":\"前岛环路金山软件园\"},\"富文本\":{\"fileId\":\"GEYDAMJTHE3DKMJYHE2DURKDINBUQS2ZGRAUGUKIKE\",\"summary\":\"富文本简略内容\",\"modifyDate\":\"2025/12/31\"}}"
    }
  ]
}
```


#### 参数说明

- `file_id` (string, 必填): 多维表格文件 ID（路径参数）
- `sheet_id` (integer, 必填): 数据表 ID（路径参数）
- `records` (array[object], 必填): 要更新的记录列表，每个元素包含 `id`（记录 ID）和 `fields_value`（序列化 JSON 字符串）
- `prefer_id` (boolean, 可选): 为 `true` 时 `fields_value` 内部使用字段 ID 作为 key；默认 `false`（使用字段名）

**records 元素结构：**

| 字段 | 类型 | 是否必填 | 说明 |
|------|------|----------|------|
| `id` | string | 是 | 目标记录的记录 ID（由 `list_records` / `get_record` 返回） |
| `fields_value` | string | 是 | 序列化的 JSON 字符串，内容为字段名（或字段 ID）→ 新值的映射，仅需传入要修改的字段 |

> `fields_value` 是 **JSON 字符串**，不是对象。只传需要修改的字段，未传字段保持原值不变。

**各字段类型的 fields_value 填写规范：**

| 字段名（类型） | 值类型 | 示例值 | 备注 |
|--------------|--------|-------|------|
| 文本类（SingleLineText / MultiLineText） | string | `"新文本"` | — |
| 日期（Date） | string | `"2025/11/15"` | 须符合字段 `number_format` 格式 |
| 时间（Time） | string | `"11:12:15"` | 须符合字段时间格式 |
| 数字 / 货币 / 百分比（Number / Currency / Percent） | int \| float | `125` / `215` / `98` | — |
| 复选框（Checkbox） | boolean | `false` | — |
| 进度（Complete） | int | `50` | 进度值 0–100 |
| 等级（Rating） | int | `3` | 不超过字段 `max_value` 上限 |
| 单选项（SingleSelect） | string | `"选项1"` | 必须是已有选项的 `value` |
| 多选项（MultipleSelect） | string[] | `["选项1","选项2"]` | 每项均须是已有选项 `value` |
| 电话 / 邮箱 / 身份证（Phone / Email / ID） | string | `"18800000000"` | — |
| 超链接（Url） | object | `{"address":"https://…","displayText":"百度"}` | 响应中以数组形式返回 |
| 关联（Link） | string[] | `["recXXXXXX"]` | 关联记录的 id 数组 |
| 联系人（Contact） | object[] | `[{"id":"uid","nickname":"张三","avatar_url":"https://…"}]` | `id` 为用户 uid |
| 地址（Address） | object | `{"districts":["广东省","珠海市","香洲区"],"detail":"详细地址"}` | `districts` 层级与字段 `address_level` 一致 |
| 富文本（Note） | object | `{"fileId":"…","summary":"摘要","modifyDate":"2024/12/09 12:00:00"}` | — |
| 附件（Attachment） | object[] | `[{"uploadId":"…","fileName":"a.png","size":1024,"source":"Cloud","type":"image/png"}]` | 需先上传获得 `uploadId`；`linkUrl`、`imgSize` 选填 |
| 最后修改时间（LastModifiedTime） | string | `"2024/12/10 10:10:10"` | 通常由系统自动维护 |
| 公式 / 编号 / 创建时间 / 创建人 / 最后修改者 / 引用（Formula / AutoNumber / CreatedTime / CreatedBy / LastModifiedBy / Lookup） | — | **不可填写** | 自动字段，传入会被忽略或报错 |

字段类型定义及值格式完整说明可参考 `dbsheet.create_fields`（`param_detail` 各类型节）。

**请求体示例（覆盖大部分可写字段类型）**

```json
{
  "prefer_id": false,
  "records": [
    {
      "id": "recXXXXXXXX",
      "fields_value": "{\"日期\":\"2025/11/15\",\"时间\":\"11:12:15\",\"数字\":125,\"货币\":215,\"多行文本\":\"yesit'sright\",\"百分比\":98,\"身份证\":\"110101**************9\",\"电话\":\"18800000000\",\"超链接\":{\"address\":\"https://www.baidu.com\",\"displayText\":\"百度\"},\"单选项\":\"选项1\",\"多选项\":[\"多选选项1\",\"多选选项2\"],\"等级\":2,\"联系人\":[{\"id\":\"281653414\",\"nickname\":\"nickName1\"}],\"地址\":{\"districts\":[\"广东省\",\"珠海市\",\"香洲区\"],\"detail\":\"前岛环路金山软件园\"},\"富文本\":{\"fileId\":\"GEYDAMJTHE3DKMJYHE2DURKDINBUQS2ZGRAUGUKIKE\",\"summary\":\"富文本简略内容\",\"modifyDate\":\"2025/12/31\"}}"
    }
  ]
}
```


#### 返回值说明

```json
{
  "code": 0,
  "msg": "",
  "data": {
    "records": [
      {
        "fields": "{\"日期\":\"2025/11/15\",\"数字\":125,\"公式\":340,\"最后修改者\":{\"id\":\"280026893\",\"nickName\":\"霧雨澪音\"},\"最后修改时间\":\"2024/12/09 17:47:18\"}",
        "id": "recXXXXXXXX"
      }
    ]
  }
}

```

| 字段 | 类型 | 说明 |
|------|------|------|
| `code` | integer | 0 表示成功 |
| `data.records[].id` | string | 已更新记录 ID |
| `data.records[].fields` | string | 序列化 JSON 字符串，包含更新后所有字段的实际值（含系统自动字段） |


---

## 3. dbsheet.list_records

#### 功能说明

分页遍历数据表中的记录，支持按视图过滤、指定返回字段，以及通过 `filter` 参数实现复杂查询条件（多字段 AND/OR 组合筛选）。



#### 调用示例

基础分页查询：

```json
{
  "file_id": "string",
  "sheet_id": 1,
  "page_size": 100,
  "offset": "",
  "fields": [
    "名称",
    "状态",
    "截止日期"
  ]
}
```

带筛选条件查询：

```json
{
  "file_id": "string",
  "sheet_id": 1,
  "page_size": 100,
  "offset": "",
  "filter": {
    "mode": "AND",
    "criteria": [
      {
        "field": "状态",
        "op": "Intersected",
        "values": [
          "进行中"
        ]
      },
      {
        "field": "数量",
        "op": "Greater",
        "values": [
          "10"
        ]
      },
      {
        "field": "名称",
        "op": "Contains",
        "values": [
          "关键词"
        ]
      }
    ]
  }
}
```


#### 参数说明

- `file_id` (string, 必填): 多维表格文件 ID
- `sheet_id` (integer, 必填): 目标数据表 ID
- `page_size` (integer, 可选): 每页记录数
- `offset` (string, 可选): 翻页游标，首次请求传空字符串，后续传响应中的 `offset` 值
- `view_id` (string, 可选): 按指定视图返回记录
- `max_records` (integer, 可选): 最多返回的记录总数
- `fields` (array, 可选): 只返回指定字段列表，不填则返回所有字段
- `filter` (object, 可选): 筛选条件
  - `mode` (string, 必填): 条件连接方式：`"AND"` 或 `"OR"`
  - `criteria` (array, 必填): 筛选条件列表
    - `field` (string, 必填): 字段名称或 ID
    - `op` (string, 必填): 筛选操作符（见附录：筛选规则）
    - `values` (array, 可选): 筛选值，`Empty`/`NotEmpty` 时可省略
- `prefer_id` (boolean, 可选): 是否使用字段 ID 作为 key
- `text_value` (string, 可选): 文本值格式：`"original"`（原始值）或 `"display"`（显示值）
- `link_value` (string, 可选): 关联字段值格式：`"id"` 或 `"value"`
- `show_record_extra_info` (boolean, 可选): 是否返回记录额外信息
- `show_fields_info` (boolean, 可选): 是否在响应中返回字段定义信息

> **分页说明**：响应中的 `offset` 指向下一页第一条记录，下次请求将该值传入 `offset` 即可翻页。最后一页不再返回 `offset`。


#### 返回值说明

```json
{
  "detail": {
    "offset": "D",
    "records": [
      { "id": "E", "fields": { "名称": "任务A", "状态": "进行中", "数量": 15 } },
      { "id": "F", "fields": { "名称": "任务B", "状态": "进行中", "数量": 20 } }
    ]
  },
  "result": "ok"
}

```

| 字段 | 类型 | 说明 |
|------|------|------|
| `detail.offset` | string | 下一页游标，无更多数据时不返回此字段 |
| `detail.records[].id` | string | 记录 ID |
| `detail.records[].fields` | object | 各字段的值 |
| `result` | string | ok 表示成功 |


---

## 4. dbsheet.get_record

#### 功能说明

获取数据表中某条指定记录的完整字段内容。


#### 调用示例

获取单条记录：

```json
{
  "file_id": "string",
  "sheet_id": 3,
  "record_id": "B"
}
```


#### 参数说明

- `file_id` (string, 必填): 多维表格文件 ID
- `sheet_id` (integer, 必填): 目标数据表 ID
- `record_id` (string, 必填): 记录 ID
- `prefer_id` (boolean, 可选): 是否使用字段 ID 作为 key
- `text_value` (string, 可选): 文本值格式：`"original"`（原始值）或 `"display"`（显示值）
- `link_value` (string, 可选): 关联字段值格式：`"id"` 或 `"value"`
- `show_record_extra_info` (boolean, 可选): 是否返回记录额外信息
- `show_fields_info` (boolean, 可选): 是否返回字段定义信息

#### 返回值说明

```json
{
  "detail": {
    "id": "B",
    "fields": {
      "名称": "任务A",
      "数量": 123,
      "日期": "2021/5/1",
      "状态": "未开始"
    }
  },
  "result": "ok"
}

```

| 字段 | 类型 | 说明 |
|------|------|------|
| `detail.id` | string | 记录 ID |
| `detail.fields` | object | 各字段的值 |
| `result` | string | ok 表示成功 |


---

## 5. dbsheet.delete_records

#### 功能说明

批量删除数据表中的指定记录。`records` 为记录 ID 的字符串数组，**不是对象数组**。



#### 操作约束

- **前置检查**：调用 list_records 或 get_record 核对拟删记录的内容，确认记录 ID 正确
- **用户确认**：批量删除记录不可恢复，必须向用户确认记录列表和数量
- **禁止**：未经用户在对话中明确同意，禁止调用
> `records` 是字符串数组（记录 ID），**不是**对象数组；传 `[{"id":"G"}]` 等对象格式会导致请求失败
> 删除操作不可逆，执行前务必完成 pre_check 和用户确认

#### 调用示例

批量删除记录：

```json
{
  "file_id": "VsdfG0001234567",
  "sheet_id": 3,
  "records": [
    "G",
    "H"
  ]
}
```


#### 参数说明

- `file_id` (string, 必填): 多维表格文件 ID（路径参数）
- `sheet_id` (integer, 必填): 数据表 ID（路径参数）
- `records` (array[string], 必填): 要删除的记录 ID 列表（字符串数组，每项为一个记录 ID）

**请求体结构：**

| 字段 | 类型 | 是否必填 | 说明 |
|------|------|----------|------|
| `records` | array[string] | 是 | 记录 ID 字符串数组，每个元素为一条记录的 ID |

**请求体示例：**

```json
{
  "records": ["G", "H", "I"]
}
```

> 记录 ID 可通过 `dbsheet.list_records` 或 `dbsheet.get_record` 获取。


#### 返回值说明

```json
{
  "code": 0,
  "msg": "",
  "data": {
    "records": [
      { "id": "G", "deleted": true },
      { "id": "H", "deleted": true }
    ]
  }
}

```

| 字段 | 类型 | 说明 |
|------|------|------|
| `code` | integer | 响应代码，0 表示成功，非 0 表示失败 |
| `msg` | string | 响应信息 |
| `data.records` | array[object] | 删除结果列表 |
| `more` | object | 更多错误信息（失败时返回） |


---

## 6. dbsheet.records_list

#### 功能说明


**请求体（均在 JSON 内，无 URL query）**

| 名称 | 类型 | 必填 | 说明 |
|------|------|------|------|
| fields | array[string] | 是* | 指定返回记录中的字段；*文档写必填，若不填则默认返回全部字段。`prefer_id=true` 时填字段 id，否则填字段名 |
| filter | object | 否 | 筛选条件 |
| filter.criteria | array[object] | 条件内必填 | 条件数组，每项含 `field`（字段名/id）、`op`（操作符，如 Contains / Equal / Empty 等）、`values`（筛选值，Empty/NotEmpty 时可省略） |
| max_records | integer | 否 | 最多取前 max_records 条；不填则不限 |
| page_size | integer | 否 | 每页大小，默认 100，范围 1–1000 |
| page_token | string | 否 | 分页游标；有下一页时用上次的 page_token |
| prefer_id | boolean | 否 | 为 true 时 fields 等按字段 id 解析 |
| show_fields_info | boolean | 否 | 是否额外返回字段元信息（类似 Schema fields） |
| show_record_extra_info | boolean | 否 | 是否返回创建者、创建时间、最后修改者、最后修改时间等 |
| text_value | string | 否 | 不填默认 original；可选 original、text、compound |
| view_id | string | 否 | 指定视图则从该视图取用户可见记录；不填从工作表取 |

SkillHub：未传 `body` 或仅 `{}` 时合并默认 `prefer_id=false`、`show_fields_info=false`、`text_value=original`；可与顶层同名字段或整包 `body` 混用，**同键以 MCP 顶层为准**。


> filter.criteria 的结构需符合多维表格开放接口对筛选条件的约定。

#### 调用示例

最简（依赖 SkillHub 默认体）：

```json
{
  "file_id": "string",
  "sheet_id": 1,
  "body": {}
}
```

返回文本值并携带额外信息：

```json
{
  "file_id": "string",
  "sheet_id": 1,
  "prefer_id": false,
  "show_fields_info": false,
  "text_value": "text",
  "show_record_extra_info": true
}
```

分页与视图：

```json
{
  "file_id": "string",
  "sheet_id": 1,
  "view_id": "B",
  "page_size": 50,
  "page_token": ""
}
```


#### 参数说明

- `file_id` (string, 必填): 多维表格文件 ID
- `sheet_id` (integer, 必填): 数据表 id
- `body` (object, 可选): 可选整包请求体；与顶层字段混用时同键以顶层为准
- `fields` (array, 必填): 指定所返回记录中的字段信息，若不填写则默认返回全部字段。依据 prefer_id 的值，需要输入字段名或字段 id
- `filter` (object, 可选): 筛选条件
  - `mode` (string, 必填): 条件连接方式：`"AND"` 或 `"OR"`
  - `criteria` (array, 必填): 筛选条件列表，每项包含：
    - `field` (string, 必填): 字段名称或字段 id（由 prefer_id 决定）
    - `op` (string, 必填): 筛选操作符，见多维表格参数说明（如 `Contains`、`Intersected`、`Greater`、`Less`、`Equal`、`Empty`、`NotEmpty` 等）
    - `values` (array, 可选): 筛选值；`Empty` / `NotEmpty` 操作符时可省略
- `max_records` (integer, 可选): 指定要获取的"前 max_records 条记录"，若不填写则默认返回全部记录
- `page_size` (integer, 可选): 分页获取记录时的每页大小，缺省值为 100，取值范围 1-1000
- `page_token` (string, 可选): 分页起始位置。当存在分页且未查询到最后一页或 max_records 记录时，返回值会包含 page_token
- `prefer_id` (boolean, 可选): 使用 id 来标识字段和选项。为 true 时，参数内全部的 field、fields 参数均按照 id 做解析
- `show_fields_info` (boolean, 可选): 是否额外返回一个 fields 结构体，展示字段信息（类似 Base Schema 中的 fields）
- `show_record_extra_info` (boolean, 可选): 是否额外显示创建者、创建时间、最后修改者、最后修改时间信息（与是否有对应字段无关）
- `text_value` (string, 可选): 返回值类型，不填默认为 original。可选：original（原始值）、text（文本值）、compound（原始值和文本值）
- `view_id` (string, 可选): 指定视图 id。填写后从该视图获取用户所见记录；不填则从工作表获取记录

所有列举参数均在 **POST JSON 请求体** 中，不拼 URL query。

**SkillHub 默认体**：`prefer_id=false`、`show_fields_info=false`、`text_value=original`。

若同时传 `body` 与顶层字段，同键以 **顶层** 为准。


#### 返回值说明

```json
{
  "code": 0,
  "msg": "",
  "data": {
    "fields_schema": [
      {
        "name": "文本",
        "type": "MultiLineText",
        "id": "B",
        "data": { "unique_value": false }
      },
      {
        "name": "数字",
        "type": "Number",
        "id": "C",
        "data": { "number_format": "0.00_ " }
      }
    ],
    "records": [
      {
        "fields": "{\"单选项\":\"选项1\",\"数字\":\"123.00 \",\"文本\":\"第一行文本\",\"日期\":\"2024/12/20\",\"等级\":\"1\"}",
        "id": "B",
        "created_time": "2024/12/20 11:30:32",
        "creator": "280026893",
        "last_modified_by": "280026893",
        "last_modified_time": "2024/12/20 15:47:01"
      }
    ],
    "page_token": ""
  }
}

```

| 字段 | 类型 | 说明 |
|------|------|------|
| `code` | integer | 响应代码，非 0 表示失败 |
| `msg` | string | 响应信息 |
| `data` | object | 响应数据 |
| `more` | object | 更多的错误信息 |


---

## 7. dbsheet.records_search

#### 功能说明


**请求体（均在 JSON 内，无 URL query）**

| 名称 | 类型 | 必填 | 说明 |
|------|------|------|------|
| records | array[string] | 是 | 记录 id 列表 |
| prefer_id | boolean | 否 | 是否使用字段 / 选项 id 而不是字段 / 选项名来标识 |
| show_fields_info | boolean | 否 | 为 true 时额外返回 fields 结构体展示字段信息；返回范围取决于是否指定 fields 或 view_id |
| show_record_extra_info | boolean | 否 | 为 true 时额外显示创建者、创建时间、最后修改者、最后修改时间（与是否有对应字段无关） |
| text_value | string | 否 | 返回值类型，不填默认 original；可选 original、text、compound |


> records 为必填参数，需传入有效的记录 id 列表。
> records_search 在 MCP 层若要求 filter 为必填参数，传空条件 {"mode":"AND","criteria":[]} 即可满足

#### 调用示例

按记录 id 批量检索：

```json
{
  "file_id": "string",
  "sheet_id": 1,
  "records": [
    "B",
    "C"
  ]
}
```

返回文本值并携带额外信息：

```json
{
  "file_id": "string",
  "sheet_id": 1,
  "records": [
    "B",
    "C"
  ],
  "prefer_id": false,
  "show_fields_info": false,
  "text_value": "text",
  "show_record_extra_info": true
}
```


#### 参数说明

- `file_id` (string, 必填): 多维表格文件 id
- `sheet_id` (integer, 必填): 数据表 id
- `body` (object, 可选): 可选整包请求体；与顶层字段混用时同键以顶层为准
- `records` (array, 必填): 记录 id 列表，指定要检索的记录
- `prefer_id` (boolean, 可选): 是否使用字段 / 选项 id 而不是字段 / 选项名来标识
- `show_fields_info` (boolean, 可选): 是否额外返回 fields 结构体展示字段信息。为 true 时，若指定了 fields 则返回指定字段；未指定 fields 时，根据是否指定 view_id 决定返回视图可见字段或全部字段
- `show_record_extra_info` (boolean, 可选): 是否额外显示创建者、创建时间、最后修改者、最后修改时间信息（与是否有对应字段无关）
- `text_value` (string, 可选): 返回值类型，不填默认为 original。可选：original（原始值）、text（文本值）、compound（原始值和文本值）

#### 返回值说明

```json
{
  "code": 0,
  "msg": "",
  "data": {
    "records": [
      {
        "fields": "{\"单选项\":\"选项1\",\"图片和附件\":\"12KB.docx,aigc\",\"数字\":\"123.00 \",\"文本\":\"第一行文本\",\"日期\":\"2024/12/20\",\"等级\":\"1\"}",
        "id": "B"
      },
      {
        "fields": "{\"单选项\":\"选项2\",\"图片和附件\":\"14.4KB.png\",\"数字\":\"321.00 \",\"文本\":\"第二行文本\",\"日期\":\"2024/12/21\",\"等级\":\"2\"}",
        "id": "C"
      }
    ]
  }
}

```

| 字段 | 类型 | 说明 |
|------|------|------|
| `code` | integer | 响应代码，非 0 表示失败 |
| `msg` | string | 响应信息 |
| `data` | object | 响应数据 |
| `more` | object | 更多的错误信息 |


---

