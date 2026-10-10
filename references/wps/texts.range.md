# wps.texts.range

## 1. wps.texts.range

#### 功能说明

查询在线文字文档段落文本范围。

> 返回 range_begin/range_end（0-based 字符区间）。部分文档查询返回 range_begin=0,range_end=0（区间接口对该文档无数据），并非调用错误；不要据此推断段落为空。
> 段落字符区间兜底：用 wps.texts.search 找文本拿 0-based 区间，或导出 docx 解析 XML。
> **状态任务行高亮**：`search`（`is_all=true`）仅 1 处命中时，**禁止**先 `wps.texts.count` 或 P1…PN 二分/线性试探。对命中所在段落调用本工具**一次**（`paragraph_index` 以 search 结果或 workflow 为准），使 `range_begin ≤ hit.begin < range_end` 后 `highlight`。流程见 `references/workflows/wps-multi-target-search-style.md`（场景 2 路径 C）。
> **选区段落边框 / 长文档段落映射**：`search` 得 `begin` 后，先 `wps.texts.count` 得 N；**N>10** 时对 `[1,N]` **二分**查本工具（mid 段比较 `range_begin`/`range_end` 缩区间，约 ⌈log₂N⌉ 次）；**N≤10** 时从 P1 顺序查命中即停。**禁止**不论 N 从 P1 线性扫到文末（见 `references/workflows/wps-paragraph-align-indent.md`（场景 7））。

#### 调用示例

段落查询：

```json
{
  "file_id": "<FILE_ID>",
  "paragraph_index": 1
}
```

#### 参数说明

- `file_id` (string, 三选一必填: `url` / `link_id` / `file_id`): 文件 id
- `link_id` (string, 三选一必填: `url` / `link_id` / `file_id`): 分享 id
- `paragraph_index` (number, 必填): 段落索引
- `url` (string, 三选一必填: `url` / `link_id` / `file_id`): 文档 URL

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "range_begin": 0,
    "range_end": 75
  }
}
```

| 字段 | 类型 | 说明 |
|------|------|------|
| `code` | number | 0 表示成功 |
| `data` | object | 业务数据 |
| `message` | string | 结果说明 |
