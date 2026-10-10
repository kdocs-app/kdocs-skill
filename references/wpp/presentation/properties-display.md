# 演示文稿 · 显示与主题

> facet: presentation.properties

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `wpp.presentation.properties.color_scheme` | 查询演示文稿的配色方案 |  |
| `wpp.presentation.properties.designs` | 查询演示文稿的设计集合 |  |
| `wpp.presentation.properties.display_comments` | 演示文稿的显示批注：支持 查询/设置（是否在全稿范围显示批注） | `verb` |
| `wpp.presentation.properties.extra_colors` | 查询演示文稿的额外颜色 |  |
| `wpp.presentation.properties.fonts` | 查询演示文稿的字体集合 |  |
| `wpp.presentation.properties.line_break` | 查询演示文稿的换行字符 |  |
| `wpp.presentation.properties.section_properties` | 查询演示文稿的节属性 |  |

---

## 1. wpp.presentation.properties.color_scheme

#### 功能说明

查询演示文稿的配色方案。

**幂等性**：是 — safe

#### 调用示例

查询演示文稿的配色方案：

```json
{
  "file_id": "<FILE_ID>"
}
```

#### 参数说明

- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `url` (string, 可选): 文档 URL；与 link_id、file_id 三选一

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "data": {
      "color_scheme": {}
    }
  }
}
```


---

## 2. wpp.presentation.properties.designs

#### 功能说明

查询演示文稿的设计集合。

**幂等性**：是 — safe

#### 调用示例

查询演示文稿的设计集合：

```json
{
  "file_id": "<FILE_ID>"
}
```

#### 参数说明

- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `url` (string, 可选): 文档 URL；与 link_id、file_id 三选一

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "data": {
      "designs": [
        {
          "index": 1,
          "name": "Office Theme"
        }
      ]
    }
  }
}
```


---

## 3. wpp.presentation.properties.display_comments

#### 功能说明

演示文稿的显示批注：支持 查询/设置（是否在全稿范围显示批注）。

**幂等性**：是 — safe

#### 调用示例

查询演示文稿的显示批注：

```json
{
  "file_id": "<FILE_ID>",
  "verb": "query"
}
```

设置演示文稿的显示批注：

```json
{
  "file_id": "<FILE_ID>",
  "verb": "update",
  "display_comments": false
}
```

#### 参数说明

- `display_comments` (boolean, 可选): 是否显示批注
- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `url` (string, 可选): 文档 URL；与 link_id、file_id 三选一
- `verb` (string, 必填): HTTP 动作：query（查询） / update（设置）。可选值：`query` / `update`

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "data": {
      "display_comments": true
    }
  }
}
```


---

## 4. wpp.presentation.properties.extra_colors

#### 功能说明

查询演示文稿的额外颜色。

**幂等性**：是 — safe

#### 调用示例

查询演示文稿的额外颜色：

```json
{
  "file_id": "<FILE_ID>"
}
```

#### 参数说明

- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
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

## 5. wpp.presentation.properties.fonts

#### 功能说明

查询演示文稿的字体集合。

**幂等性**：是 — safe

#### 调用示例

查询演示文稿的字体集合：

```json
{
  "file_id": "<FILE_ID>"
}
```

#### 参数说明

- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `url` (string, 可选): 文档 URL；与 link_id、file_id 三选一

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "data": {
      "fonts": [
        {
          "name": "微软雅黑"
        }
      ]
    }
  }
}
```


---

## 6. wpp.presentation.properties.line_break

#### 功能说明

查询演示文稿的换行字符。

**幂等性**：是 — safe

#### 调用示例

查询演示文稿的换行字符：

```json
{
  "file_id": "<FILE_ID>"
}
```

#### 参数说明

- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `url` (string, 可选): 文档 URL；与 link_id、file_id 三选一

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "data": {
      "line_break": {}
    }
  }
}
```


---

## 7. wpp.presentation.properties.section_properties

#### 功能说明

查询演示文稿的节属性。

**幂等性**：是 — safe

#### 调用示例

查询演示文稿的节属性：

```json
{
  "file_id": "<FILE_ID>"
}
```

#### 参数说明

- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `url` (string, 可选): 文档 URL；与 link_id、file_id 三选一

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "data": {
      "section_properties": {
        "count": 0
      }
    }
  }
}
```
