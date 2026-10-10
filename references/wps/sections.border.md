# wps.sections.border

## 1. wps.sections.border

#### 功能说明

删除在线文字文档节的页面边框，恢复无边框态（line_style=0、line_width=9999999、art=0）。

**幂等性**：是 — safe

> **添加页面边框**：`break_type=1`（单实线）或 `7`（双线）写入节页面边框；返回 `data.deleted_section` 为 JSAPI **命名遗留**（表示边框操作已作用于该节），**非**删除节。**禁止**用 `wps.texts.border`（段落边框）。
> 写后 `wps.sections.border_data` query；若 `data={}` 空对象但写侧 code=0，仍视为写成功。详见 `references/workflows/wps-section-break-layout.md`（场景 4）。

#### 调用示例

删除页面边框（恢复无边框）：

```json
{
  "file_id": "<FILE_ID>",
  "section_index": 1
}
```

#### 参数说明

- `break_type` (number, 可选): 兼作页面边框线型（WdLineStyle）：0=无边框（清除页面边框，默认）；1=单线；其他值按 WdLineStyle 枚举写入
- `file_id` (string, 三选一必填: `url` / `link_id` / `file_id`): 文件 id；与 url、link_id 三选一
- `link_id` (string, 三选一必填: `url` / `link_id` / `file_id`): 分享 id；与 url、file_id 三选一
- `section_index` (number, 必填): 节索引，从 1 开始
- `url` (string, 三选一必填: `url` / `link_id` / `file_id`): 文档 URL；与 link_id、file_id 三选一

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "deleted_section": 1
  }
}
```

| 字段 | 类型 | 说明 |
|------|------|------|
| `code` | number | 0 表示成功 |
| `data` | object | 业务数据 |
| `message` | string | 结果说明 |
