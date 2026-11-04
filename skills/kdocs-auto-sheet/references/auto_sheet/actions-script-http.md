# 图像识别（`db_ocr_word`）

对记录里**图片附件**做 OCR，把证件/票据的结构化字段识别出来。

```jsonc
{ "key": "s2", "action": "db_ocr_word", "label": "图像识别",
  "params": {
    "file_link":     { "$var": "trigger_dataframe.附件列" },  // 变量引用，不是链接字符串
    "ocr_type":      "bill",                                  // 见 get_context 的 ocr_types
    "ocr_word_list": [ "date", "pretax_amount", "seller" ]    // 必须属于该 ocr_type
  } }
```

三条硬约束（都已实测）：

1. **只吃图片**：`jpeg / png / gif / jpg / webp / bmp`。附件是 **PDF 会被拒**
   （`OCR_WORD_NOT_SUPPORT_IMAGE 不支持的图片类型`）—— 数电普票 PDF 走不了这个动作，
   这种情况要用表格自己的「发票识别」字段，别硬套这个动作。
2. **`ocr_type` / `ocr_word_list` 是枚举**：合法值在 `auto_sheet.get_context` 的 `ocr_types`
   小节（12 类证件/票据 + 每类可提取字段）。`list_actions` 给不出这些值（模板下拉是动态的），
   所以**必须**去 `ocr_types` 查。写错会在运行时直接报参数错误，服务端也会在保存前拦下。
3. **`file_link` 是"逐记录一组附件"的列表的列表**：多记录检索时每条记录的附件都会参与，
   **只要有一条不是图片，整个节点就失败** —— 所以筛选条件要收敛到图片那一条
   （例如按某个能唯一定位那条记录的字段过滤）。

> **已验证**：真实发票图片（jpeg）识别成功。**未验证**：png/gif/webp/bmp 等其它格式，
> 以及非发票的证件类型（身份证/营业执照/行驶证…）—— 参数目录里有，但没真机跑过。
# HTTP 请求（`send_http_request_v3`）

```jsonc
{ "key": "s3", "action": "send_http_request_v3", "params": {
    "method": "post",
    "url": "https://example.com/api/notify",        // 可嵌 ${...} 变量
    "query":  { "id": "${s1.记录ID}" },              // 键值对象，工具会转成底层 [{key,value}]
    "header": { "Content-Type": "application/json" },
    "request_body": { "msg": "订单 ${s1.订单号} 已通过" },  // 对象会自动转成 JSON 文本
    "response_type": "json" } }                      // json（默认）/ text / none
```

- `method`：`get` / `post` / `put` / `delete` / `patch` / `head` / `options`（大小写不限，工具会归一）；
- `query` / `header` 可以写成**键值对象**（工具转成底层的 `[{key,value}]`），也可以直接写数组；
- `request_body` 写对象/数组会被自动序列化成 JSON 文本；
- ⚠️ `response_type: json`（**默认**）打一个返回 HTML/纯文本的地址会报
  「返回数据格式转换错误」→ 这种目标要显式写 `response_type: "text"`；
- ⚠️ `request_type` 底层**只有 `none` / `json`** —— **没有 multipart，传不了文件**；
- ⚠️ **公网目标走平台沙盒代理**（内网/私有化环境是直连）：代理不健康时请求会失败，
  实测遇到过一次 503。内网地址需要业务登录态，自动化里未必带得上（实测 403）。

> ⚠️ **这是能往外发数据的动作**。URL 或请求体里带记录内容 = 把业务数据发到外部地址。
> 用户没有明确要求时，**不要主动**把表里的数据往外部地址带。

> **已验证**：GET（含 query 键值）、POST（含 JSON 请求体）、`text` 与 `json` 两种响应类型，
> 真机均 `http请求发送成功`。
# 执行脚本（`run_python_script_v2` / `airscript_v2`）

执行**文件里已有的** Python / AirScript 脚本。两个动作参数面完全相同，只有脚本语言不同。

| 参数 | 说明 |
|------|------|
| `script_id` | **必填**，已有脚本文件的 id。⚠️ **本工具不创建脚本** —— 必须由用户提供（编辑器里选中脚本后复制 id）。留空只能存草稿，开启时平台会判为必填缺失 |
| `params` | 脚本入参（可选）。两种写法等价：`{"limit": 10, "名称": {"$var": "trigger_dataframe.B"}}` 或 `[{"key":"limit","value":10}]`。**字面量原样传给脚本**；`$var` / `$template` 写法编译成表达式对象、运行时求值 |
| `resp_schema` | 脚本返回值结构（可选，JSON **字符串**，默认 `{}`），例如 `{"name":"zhangsan","id":1}`。非法 JSON 会在保存前被拦下 |

脚本里怎么读入参：

- Python：`Context['argv']['<key>']`
- AirScript：`Context.argv.<key>`

形状依据（不猜）：线上模板 3 个参数 + chatflow 源码（运行期结构是 `{key,value}` 列表，
重复 key 直接报错）+ 参考实现 action-meta（三个参数都是 `original`，入参数组是**裸 JSON**）。
入参顺序按 key 排序，保证同一份 DSL 编出同样的字节。

写进流程后：

```json
{"key":"s1","action":"run_python_script_v2","params":{
  "script_id":"<已有脚本 id>",
  "params":{"日期": {"$var":"trigger_dataframe.D"}},
  "resp_schema":"{}"}}
```

**验证级别以生成块为准**（见 `auto_sheet/actions-catalog.md` 动作总表的"验证级别"列 —— 那是从验证级别表渲染出来的，有离线测试防过期）。**不要在本手册里另抄一份级别表**：
上一版这里手抄过一份，`run_python_script_v2` 早已升为真机验证、抄的那份还停在
`shape_only`，两处打架。

canonical **以界面产物为准**逐字段对齐（用户界面建的流程里的 `airscript_v2` 步骤），
对齐了三处，缺一处脚本拿到的就不是标量：

1. 变量必须套用模板声明的 `method_options.default_method`（`{"each":"false","format":"json"}`）；
2. 变量项的 `biz_type` 取模板 `val.default_type`（`string`）——**不是** `allow_types`
   （后者只描述"可选的取值来源"，界面实测写的就是 `string`）；
3. 外层 `original` 也要带 `__exp_config__.biz_type`。

入参链路（运行期源码确认）：`params` 的 `[{key,value}]` 先转成 JSON，再解成 `Context.argv`。

入参链路已**端到端闭环**：脚本里加 `console.log(Context.argv.content)` 后重跑，
节点日志回读到的正是流程传入的值。

**未验证**：**带返回值的脚本**与**下游引用返回值**（见下节）。别把"AirScript 验过"泛化成"脚本都能跑"。

### 脚本返回值：脚本里怎么写、流程里怎么用

模板在 `resp_schema` 上自带的说明（原文）：

> 在脚本中直接返回JSON结构体，后续即可使用该返回值，比如：`return {"name":"zhangsan","id":1}`

所以脚本侧：

- **AirScript**：`return { name: "zhangsan", id: 1 };`
- **Python**：`return {"name": "zhangsan", "id": 1}`（同一份模板说明；Python 链路本身未验）

流程侧要把**同一份结构**声明到 `resp_schema` 参数（JSON **字符串**），下游才选得到这些字段。
编辑器用的是**示例值**风格 —— 值的类型即字段类型（`""` = 字符串、`0` = 数字）：

```json
{"key":"s1","action":"airscript_v2","params":{
  "script_id":"<脚本 id>",
  "params":{"content":{"$var":"trigger_dataframe.填报内容"}},
  "resp_schema":"{ \"name\": \"\", \"id\": 0 }"}}
```

（写在 `resp_schema` 里的内容会被**原样保留**，空格也算；本工具写出的 canonical 与界面产物逐字节一致。）

下游引用（`s1` 是脚本步骤的 `key`）：

```
{"$template":"脚本返回：${s1.resp_schema.name}"}
```

### Python 脚本（`run_python_script_v2`）—— 与 AirScript 的三处差异（真机实测）

| 差异 | Python | AirScript |
|------|--------|-----------|
| 返回内容 | **`return_result({...})`**（平台注入的函数） | `return {...}` |
| 取数 | `dbt(field=[...], sheet_name='数据表')` —— **必须显式传 `sheet_name`** | — |
| 节点 `details` | 只回**打印输出**（`print` → 「输出结果」） | 回整个响应（含 `result`） |

1. **Python 顶层 `return` 是语法错误**，所以平台另造了 `return_result()`。编辑器（Pyright）
   会把它标成「not defined」—— 那是**静态误报**（类型桩里有这个函数，`globals()` 里也有），
   运行时可用；**不要自己 `def` 一个**（会把平台那个盖掉，掩盖问题）。
2. `dbt(field, sheet_name='', book_url='', condition={})`：**默认空 `sheet_name` 在自动化里解析不出
   「当前激活表」** → `RuntimeError: sheet with name [] not exists`（真机踩到，不是脚本逻辑问题）。
   显式传表名即可：`dbt(field=['填报内容','数量'], sheet_name='数据表')`，返回 DataFrame（列名=字段名）。
3. `Context` 是 **dict**：`Context['argv']`；没有入参时用 `Context.get('argv')` 更安全（否则 `KeyError`）。

注入的 API（脚本 `globals()` 实测，无需 import）：

- 数据表：`dbt` / `insert_dbt` / `update_dbt` / `delete_dbt`、`DBTFieldValue` / `DBTAttachment(s)` / `DBTFieldDtype`
- 表格：`xl` / `write_xl` / `delete_xl`
- 其它：`display` / `echarts` / `ocr` / `wps_ocr_v5` / `AMap` / `wpsypi`
- 上下文变量：`file_id` / `script_id` / `script_name` / `office_type` / `sheet_type` …

#### 注入 API 的行为（真机实测，2026-09-17）

这些函数**无需 import**（平台注入进脚本 globals），且**全都要显式传 `sheet_name`** ——
默认空串解析不出「当前激活表」，报 `RuntimeError: sheet with name [] not exists`。

| 接口 | 签名 | 实测行为 |
|------|------|----------|
| 读 | `dbt(field, sheet_name='', book_url='', condition={}) -> dict[str, DataFrame]` | 返回 DataFrame，**index = 记录 id**（如 `['Du','Dv','Dw']`），列名 = 字段名 |
| 增 | `insert_dbt(data, sheet_name='', new_sheet=False) -> None` | 插入成功（dict 也可）；**不传 `sheet_name` → 报错** |
| 改 | `update_dbt(data, sheet_name='') -> None` | **按 DataFrame 的 index（记录 id）定位行**：读出 → 改单元格 → 写回，只改那一行；**不传 `sheet_name` → 报错** |
| 删 | `delete_dbt(sheet_name, book_url='') -> dict[str, DataFrame]` | ⚠️ **清空整张表**（没有过滤条件参数），而且**不报错**（`exec_status=1`） |

⚠️ **安全含义**：**脚本动作不是只读**。一行 `delete_dbt('数据表')` 就能清空整表 ——
所以影响面预审对含脚本的流程只说「**影响面未知**」，绝不说「只读」；**开启前必须人工确认脚本内容**。

> 平台自己的 docstring 有复制粘贴错误（`delete_dbt` 的参数说明写的是**读接口**的文案：
> 「字段所在的数据表名称…如果为 None 则返回全部数据表数」），以上表实测为准。

返回值的下游引用与 AirScript **完全一致**：`${s1.resp_schema.<字段>}`（嵌套对象/入参对象渲染成 JSON 文本）。

**未验证**：Python 顶层返回数组（AirScript 那条会被平台拒，Python 未单独试）、`dbt` 的动态日期筛选。

### 形状金标准（为什么不要手写 canonical）

流程的 canonical JSON 里有大量**编辑器内部字段与节点快照**：

- 参数 `label`、`biz_type`（同一个参数在不同节点存的值可能不同）、`__exp_config__.sheets`、
  `value_label`、`_temp_id`、`data_type`、成员列表项里的 DSL 糖（`{"$var":…}` / `{"$raw":…}`）；
- 这些值**推导不出来**（同一个参数在不同节点不一样），只能"原样保留"。

由此#### 各动作"能引用什么"（真机 / 源码证据）

| 动作 | 平台声明 | 怎么写 | 证据 |
|------|----------|--------|------|
| `db_search_rows` | pull 1000（`sheet_id`+`fields`） | `${s1.<字段名>}` → 该步骤查的那张表的字段；写错字段名**编译期报错** | 真机 |
| `db_insert_rows` / `db_update_rows` | pull 1004（`sheet_id`+`key_values`） | 同上：**写回的那张表的字段**（字段名 → 列 id） | 源码 + 单测 |
| `send_http_request_v3` | pull 1001（`response_schema`+`response_type`） | `${s1.body}`（响应体）、`${s1.body.<字段>}`（按 `response_schema` 声明的结构逐层取，数组可下标）；⚠️ `${s1.status_code}` **恒为 0**（平台未回填，别用它判断成败） | **真机** |
| `db_ocr_word` | pull 10000（`ocr_type`+`ocr_word_list`） | `${s1.<识别项>}` → 你在 `ocr_word_list` 里配置的键；未匹配只**提示不拦**（列集是源码推断） | 源码 + 单测 |
| 脚本（AirScript/Python） | pull 1005（`resp_schema`） | `${s1.resp_schema.<字段>}` —— **必须带 `resp_schema` 这一层**，少了会静默取空 | 真机 |
| 消息 / 通知 / 延迟 / 邮件 / 各机器人 | `return: null` | ❌ 引用即**编译期报错**（静默取空最难查） | 真机 |

两条纪律：

1. **不要手写或改写 canonical**：一律用 `auto_sheet.save_flow` 的 DSL（它负责形状），
   否则一个字段写错就可能让编辑器判空或让运行期静默取空。
2. **编辑现有流程时，读出来的值原样带回**：`get_flow` → 改你想改的部分 → `save_flow`。
   工具对"读出→存回"做了形状保真（连表达式对象都不会被二次包装）。

工具侧有一道**金标准回归**守着这件事：从平台真实流程抓 canonical 当基准，反编译→重编后
逐字段比对，未登记的差异直接让测试失败（差异清单逐条写了原因与影响）。

### 这个步骤到底有没有返回值（跨步骤引用契约，真机实测）

平台在每个动作的 UI 模板里用 `template.return[].val.pull_rule` **声明**返回值从哪来。
`auto_sheet.save_flow` 据此在**编译期**校验"引用上游输出"的写法（不再只对脚本特例）：

| 动作 | 平台声明 | 校验方式 |
|------|----------|----------|
| `db_search_rows` | `pull_rule{action:1000, params:[sheet_id, fields]}` | 字段 = **该步骤查的那张表的字段**；按列名/列 ID 解析，写错字段名编译期报错 |
| `airscript_v2` / `run_python_script_v2` | `pull_rule{action:1005, params:[resp_schema]}` | 字段 = 该步骤 `resp_schema` 的**声明树**（逐层校验） |
| `db_ocr_word` | `pull_rule{action:10000, params:[ocr_type, ocr_word_list]}` | 暂不解析（未知拉取规则时**宽容放行**） |
| `send_woa_message_v9` / `send_email` | `return: null` | ⚠️ **没有返回值**：引用 `${sN.<字段>}` 直接编译期报错 |

两条纪律：

1. **引用没有返回值的动作会静默取空**（流程能存、能跑，值永远是空）——所以必须在编译期拦，
   而不是等运行期排查。报错文案里会写明"平台模板 return 为 null"这个依据。
2. **拿不到模板的未知动作不判断**：不能因为"我们不知道"就阻断保存（宁可放行 + 提示）。

### 数组返回值（已真机实测）

脚本 `return { "items": [ {"age":1,"name":"a"}, {"age":2,"name":"b"} ], "top": "x" }` 时:

| 引用 | 实际结果 |
|------|----------|
| `${s1.resp_schema.top}` | `x` |
| `${s1.resp_schema.items}` | `{"age":1,"name":"a"} {"age":2,"name":"b"}` —— **每个元素的 JSON 用空格拼接**,且元素内 key 顺序不保证 |
| `${s1.resp_schema.items.0}` | `{"age":1,"name":"a"}` |
| `${s1.resp_schema.items.0.name}` | `a` —— **下标访问可用** |
| `${s1.resp_schema.items\|each}` | ❌ **平台 panic**:`reflect: Call using []map[string]interface {} as type []*flowjson.DataFrame`(节点失败且不给可读原因)—— 已改为**编译期拦下** |
| 脚本顶层 `return [1,2,3]` | ❌ 节点失败:「AS脚本输出错误,输出结果不是json格式,请检查脚本代码」—— `resp_schema` 顶层写数组也会被编译期拦下 |

两条硬约束:

1. **脚本必须 return 一个对象**;数组只能放在对象的字段里。
2. **数组字段不能用 `|each`**(平台会崩);逐个取值用下标 `items.0.name`。
   `items` 整体引用虽然可用,但渲染结果不可靠(**空格拼接 + key 顺序随机**),不要拿去解析。

**路径口径已真机验证**（脚本 `return {"id":1,"name":"zhangsan"}`，下游写进新记录再读回）：

| 引用写法 | 取到的值 |
|----------|----------|
| `${s1.resp_schema.name}` | `zhangsan` |
| `${s1.resp_schema.id}` | `1` |
| `${s1.resp_schema}` | `{"id":1,"name":"zhangsan"}`（整个返回对象） |
| `${s1.resp_schema.father.name}` | `zhangyi`（**嵌套逐层写**：脚本返回 `{"father":{"name":"zhangyi","age":30}}`） |
| `${s1.resp_schema.father}` | `{"age":30,"name":"zhangyi"}`（整个嵌套对象） |
| `${s1}` | `{"resp_schema":{"id":1,"name":"zhangsan"}}`（整个节点输出） |

也就是说**返回值在 `resp_schema` 这一层下面**：一般动作的返回值是 `$flow_<节点id>.<字段>`，
脚本动作**多了一层** `resp_schema`（运行期代码就是 `return map[string]any{"resp_schema": resp}`）。
少写这一层会**静默取空** —— 编译期已拦住并提示正确写法。

三条注意：

1. 脚本 return 的值必须是**合法 JSON**，否则运行期报错（`ErrScriptResultUnmarshal`）；
2. Python 侧若"有返回值但取不到"，节点会失败并提示「获取脚本返回值失败」；
3. 引用的字段必须先在 `resp_schema` 里**声明**，否则编译期报「未声明的字段」并列出已声明字段；
   `resp_schema` 为 `{}` 时引用任何字段都会被拦下（先去声明，别让它静默取空）。
4. `resp_schema` 的类型声明**不做强制**：声明 `"name": 0`（数字）而脚本实际返回字符串，
   照样取到值 —— 既不报错也不取空（它是给编辑器的结构声明，不是运行期类型约束）。
   嵌套对象也支持逐层引用（`resp_schema.father.name`）。
   **未验证**：**数组**返回怎么引用、把返回值写进**数字列**时的类型转换。

安全提醒：脚本体内可以做任何事（读写数据、发外部请求、执行任意逻辑），
`impact` 只能提示"会执行已有脚本，影响面无法预审"。开启前务必让用户确认脚本内容与作用。
