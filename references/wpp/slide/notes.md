# 幻灯片 · 备注

> facet: slide.notes

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `wpp.slide.notes.append` | append幻灯片append | `slide_id` |
| `wpp.slide.notes.create` | 插入幻灯片create | `slide_id` |
| `wpp.slide.notes.notes` | 查询幻灯片备注页内容（需 slide_id） | `slide_id` |
| `wpp.slide.notes.set` | 设置幻灯片set | `slide_id` |

---

## 1. wpp.slide.notes.append

#### 功能说明

append幻灯片append。

**幂等性**：是 — safe

#### 调用示例

追加幻灯片备注内容：

```json
{
  "file_id": "<FILE_ID>",
  "slide_id": 150995203,
  "text": "备注E1追加"
}
```

#### 参数说明

- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `slide_id` (number, 必填): 幻灯片 ID
- `text` (string, 可选): 文本内容
- `url` (string, 可选): 文档 URL；与 link_id、file_id 三选一

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {}
}
```


---

## 2. wpp.slide.notes.create

#### 功能说明

插入幻灯片create。

**幂等性**：是 — safe

#### 调用示例

创建幻灯片备注内容：

```json
{
  "file_id": "<FILE_ID>",
  "slide_id": 150995203,
  "text": "备注E1"
}
```

#### 参数说明

- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `slide_id` (number, 必填): 幻灯片 ID
- `text` (string, 可选): 文本内容
- `url` (string, 可选): 文档 URL；与 link_id、file_id 三选一

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "data": {}
}
  }
```


---

## 3. wpp.slide.notes.notes

#### 功能说明

查询幻灯片备注页内容（需 slide_id）。

**幂等性**：是 — safe

#### 调用示例

查询幻灯片备注页内容：

```json
{
  "file_id": "<FILE_ID>",
  "slide_id": 150995203
}
```

#### 参数说明

- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `slide_id` (number, 必填): 幻灯片 ID
- `url` (string, 可选): 文档 URL；与 link_id、file_id 三选一

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "data": {
      "notes_text": "演讲者备注",
      "slide_id": 150995201
    }
  }
}
```


---

## 4. wpp.slide.notes.set

#### 功能说明

设置幻灯片set。

**幂等性**：是 — safe

#### 调用示例

设置幻灯片备注内容：

```json
{
  "file_id": "<FILE_ID>",
  "slide_id": 150995203,
  "text": "备注E1更新"
}
```

#### 参数说明

- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `slide_id` (number, 必填): 幻灯片 ID
- `text` (string, 可选): 文本内容
- `url` (string, 可选): 文档 URL；与 link_id、file_id 三选一

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {}
}
```
