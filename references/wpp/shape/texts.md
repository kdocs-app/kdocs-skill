# 形状 · 文本

> facet: shape.texts

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `wpp.shape.texts.append` | append形状append | `slide_id` |
| `wpp.shape.texts.find_replace` | 设置形状find replace | `slide_id` |
| `wpp.shape.texts.font` | 查询形状字体格式（需 slide_id、shape_id | `slide_id` |
| `wpp.shape.texts.font_paragraph` | 设置形状字体格式段落格式 | `slide_id` |
| `wpp.shape.texts.paragraph` | 查询形状段落格式（需 slide_id 与 shape_id） | `slide_id` |
| `wpp.shape.texts.text` | 形状文本内容：支持 查询/设置（需 slide_id 与 shape_id） | `slide_id`, `verb` |

---

## 1. wpp.shape.texts.append

#### 功能说明

append形状append。

**幂等性**：是 — safe

#### 参数说明

- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `shape_id` (number, 可选): 形状 ID
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

## 2. wpp.shape.texts.find_replace

#### 功能说明

设置形状find replace。

**幂等性**：是 — safe

#### 参数说明

- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `find_what` (string, 可选): 查找文本
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `match_case` (boolean, 可选): 是否区分大小写
- `replace_what` (string, 可选): 替换文本
- `shape_id` (number, 可选): 形状 ID
- `slide_id` (number, 必填): 幻灯片 ID
- `url` (string, 可选): 文档 URL；与 link_id、file_id 三选一
- `whole_words` (boolean, 可选): 是否全字匹配

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "data": {
      "shape_id": 2
    }
  }
}
```


---

## 3. wpp.shape.texts.font

#### 功能说明

查询形状字体格式（需 slide_id、shape_id；可设 font_name/font_size）。

**幂等性**：是 — safe

#### 调用示例

查询形状字体格式：

```json
{
  "file_id": "<FILE_ID>",
  "slide_id": 1,
  "shape_id": 2
}
```

#### 参数说明

- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `shape_id` (number, 可选): 形状 ID
- `slide_id` (number, 必填): 幻灯片 ID
- `url` (string, 可选): 文档 URL；与 link_id、file_id 三选一

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "data": {
      "font": {
        "name": "微软雅黑",
        "size": 60
      }
    }
  }
}
```


---

## 4. wpp.shape.texts.font_paragraph

#### 功能说明

设置形状字体格式段落格式。

**幂等性**：是 — safe

#### 参数说明

- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `font` (object, 可选): 字体格式对象
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `paragraph` (object, 可选): 段落格式对象；alignment 用字符串枚举 LEFT/CENTER/RIGHT/JUSTIFY/DISTRIBUTE（勿传 query 返回的整数）
- `shape_id` (number, 可选): 形状 ID
- `slide_id` (number, 必填): 幻灯片 ID
- `url` (string, 可选): 文档 URL；与 link_id、file_id 三选一

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "data": {
      "shape_id": 2
    }
  }
}
```


---

## 5. wpp.shape.texts.paragraph

#### 功能说明

查询形状段落格式（需 slide_id 与 shape_id）。

**幂等性**：是 — safe

#### 调用示例

查询形状段落格式：

```json
{
  "file_id": "<FILE_ID>",
  "slide_id": 1,
  "shape_id": 2
}
```

#### 参数说明

- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `shape_id` (number, 可选): 形状 ID
- `slide_id` (number, 必填): 幻灯片 ID
- `url` (string, 可选): 文档 URL；与 link_id、file_id 三选一

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "data": {
      "paragraph": {
        "alignment": 2
      }
    }
  }
}
```


---

## 6. wpp.shape.texts.text

#### 功能说明

形状文本内容：支持 查询/设置（需 slide_id 与 shape_id）。

**幂等性**：是 — safe

#### 调用示例

查询形状文本内容：

```json
{
  "file_id": "<FILE_ID>",
  "slide_id": 1,
  "shape_id": 11,
  "verb": "query"
}
```

设置形状文本内容：

```json
{
  "file_id": "<FILE_ID>",
  "slide_id": 1,
  "shape_id": 11,
  "verb": "update",
  "text": "批次D改文本"
}
```

复核形状文本内容：

```json
{
  "file_id": "<FILE_ID>",
  "slide_id": 1,
  "shape_id": 11,
  "verb": "query"
}
```

#### 参数说明

- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `shape_id` (number, 可选): 形状 ID
- `slide_id` (number, 必填): 幻灯片 ID
- `text` (string, 可选): 文本内容
- `url` (string, 可选): 文档 URL；与 link_id、file_id 三选一
- `verb` (string, 必填): HTTP 动作：query（查询） / update（设置）。可选值：`query` / `update`

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "text": "批次B文本框"
  }
}
```
