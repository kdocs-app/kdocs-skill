# 幻灯片 · 批注

> facet: slide.comments

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `wpp.slide.comments.count` | 查询幻灯片评论评论总数 |  |
| `wpp.slide.comments.data` | 幻灯片评论评论实体：支持 删除/插入 | `author`, `author_initials`, `comment_index`, `left`, `slide_id`, `text`, `top`, `verb` |
| `wpp.slide.comments.list` | 查询幻灯片评论评论列表 |  |
| `wpp.slide.comments.reply` | 设置幻灯片评论回复评论 | `author`, `author_initials`, `comment_index`, `slide_id`, `text` |
| `wpp.slide.comments.thread` | 查询幻灯片评论回复线程 |  |

---

## 1. wpp.slide.comments.count

#### 功能说明

查询幻灯片评论评论总数。

**幂等性**：是 — safe

#### 调用示例

查询幻灯片评论总数：

```json
{
  "file_id": "<FILE_ID>",
  "slide_id": 150995203
}
```

#### 参数说明

- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `slide_id` (number, 可选): 幻灯片 ID
- `url` (string, 可选): 文档 URL；与 link_id、file_id 三选一

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "data": {
      "count": 1
    }
  }
}
```


---

## 2. wpp.slide.comments.data

#### 功能说明

幻灯片评论评论实体：支持 删除/插入。

**幂等性**：是 — safe

#### 调用示例

在幻灯片上插入批注：

```json
{
  "file_id": "<FILE_ID>",
  "slide_id": 150995203,
  "author": "测试评论",
  "author_initials": "CS",
  "text": "批注E1",
  "left": 50,
  "top": 50,
  "verb": "insert"
}
```

删除幻灯片批注：

```json
{
  "file_id": "<FILE_ID>",
  "slide_id": 150995203,
  "comment_index": 1,
  "verb": "delete"
}
```

#### 参数说明

- `author` (string, 必填): 评论作者
- `author_initials` (string, 必填): 评论作者缩写
- `comment_index` (number, 必填): 评论索引（1 起）
- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `left` (number, 必填): 左边距
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `slide_id` (number, 必填): 幻灯片 ID
- `text` (string, 必填): 文本内容
- `top` (number, 必填): 上边距
- `url` (string, 可选): 文档 URL；与 link_id、file_id 三选一
- `verb` (string, 必填): HTTP 动作：delete（删除） / insert（插入）。可选值：`delete` / `insert`

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "data": {
      "slide_id": 150995203
    }
  }
}
```


---

## 3. wpp.slide.comments.list

#### 功能说明

查询幻灯片评论评论列表。

**幂等性**：是 — safe

#### 调用示例

查询幻灯片评论列表：

```json
{
  "file_id": "<FILE_ID>",
  "slide_id": 150995203
}
```

#### 参数说明

- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `slide_id` (number, 可选): 幻灯片 ID
- `url` (string, 可选): 文档 URL；与 link_id、file_id 三选一

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "data": {
      "count": 1,
      "comments": [
        {
          "author": "测试用户",
          "author_initials": "CS",
          "comment_index": 1,
          "datetime": "2026-09-21 10:00:00",
          "left": 2.5,
          "slide_id": 150995201,
          "text": "批注内容",
          "top": 2.5
        }
      ]
    }
  }
}
```


---

## 4. wpp.slide.comments.reply

#### 功能说明

设置幻灯片评论回复评论。

**幂等性**：是 — safe

#### 调用示例

回复幻灯片批注：

```json
{
  "file_id": "<FILE_ID>",
  "slide_id": 150995203,
  "comment_index": 1,
  "author": "测试回复",
  "author_initials": "HF",
  "text": "回复E1"
}
```

#### 参数说明

- `author` (string, 必填): 评论作者
- `author_initials` (string, 必填): 评论作者缩写
- `comment_index` (number, 必填): 评论索引（1 起）
- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `slide_id` (number, 必填): 幻灯片 ID
- `text` (string, 必填): 文本内容
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

## 5. wpp.slide.comments.thread

#### 功能说明

查询幻灯片评论回复线程。

**幂等性**：是 — safe

#### 调用示例

查询幻灯片评论回复线程：

```json
{
  "file_id": "<FILE_ID>",
  "slide_id": 150995203
}
```

#### 参数说明

- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `slide_id` (number, 可选): 幻灯片 ID
- `url` (string, 可选): 文档 URL；与 link_id、file_id 三选一

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "data": {
      "count": 1,
      "comments": [
        {
          "author": "测试用户",
          "author_initials": "CS",
          "comment_index": 1,
          "datetime": "2026-09-21 10:00:00",
          "left": 2.5,
          "replies": [
            {
              "author": "测试用户",
              "reply_index": 1,
              "text": "回复内容"
            }
          ],
          "slide_id": 150995201,
          "text": "批注内容",
          "top": 2.5
        }
      ]
    }
  }
}
```
