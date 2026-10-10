# 母版 · 内容与动效

> facet: master.properties

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `wpp.master.properties.headers_footers` | 查询母版的页眉页脚 |  |
| `wpp.master.properties.hyperlinks` | 查询母版的超链接 |  |
| `wpp.master.properties.shapes` | 查询母版的形状集合（母版上的占位形状） |  |
| `wpp.master.properties.text_styles` | 查询母版的文本样式 |  |
| `wpp.master.properties.timeline` | 查询母版的动画时间线 |  |
| `wpp.master.properties.transition` | 查询母版的切换动画 |  |

---

## 1. wpp.master.properties.headers_footers

#### 功能说明

查询母版的页眉页脚。

**幂等性**：是 — safe

#### 调用示例

查询母版页眉页脚：

```json
{
  "file_id": "<FILE_ID>",
  "master_type": "SLIDE_MASTER"
}
```

#### 参数说明

- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `master_type` (string, 可选): 母版类型：SLIDE_MASTER / NOTES_MASTER / HANDOUT_MASTER
- `url` (string, 可选): 文档 URL；与 link_id、file_id 三选一

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "data": {
      "headers_footers": {}
    }
  }
}
```


---

## 2. wpp.master.properties.hyperlinks

#### 功能说明

查询母版的超链接。

**幂等性**：是 — safe

#### 调用示例

查询母版超链接：

```json
{
  "file_id": "<FILE_ID>",
  "master_type": "SLIDE_MASTER"
}
```

#### 参数说明

- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `master_type` (string, 可选): 母版类型：SLIDE_MASTER / NOTES_MASTER / HANDOUT_MASTER
- `url` (string, 可选): 文档 URL；与 link_id、file_id 三选一

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "data": {
      "hyperlinks": {
        "count": 0
      }
    }
  }
}
```


---

## 3. wpp.master.properties.shapes

#### 功能说明

查询母版的形状集合（母版上的占位形状）。

**幂等性**：是 — safe

#### 调用示例

查询母版形状集合：

```json
{
  "file_id": "<FILE_ID>",
  "master_type": "SLIDE_MASTER"
}
```

#### 参数说明

- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `master_type` (string, 可选): 母版类型：SLIDE_MASTER / NOTES_MASTER / HANDOUT_MASTER
- `url` (string, 可选): 文档 URL；与 link_id、file_id 三选一

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "data": {
      "shapes": [
        {
          "id": 2,
          "name": "Title Placeholder 1",
          "type": 14
        },
        {
          "id": 3,
          "name": "Text Placeholder 2",
          "type": 14
        }
      ]
    }
  }
}
```


---

## 4. wpp.master.properties.text_styles

#### 功能说明

查询母版的文本样式。

**幂等性**：是 — safe

#### 调用示例

查询母版文本样式：

```json
{
  "file_id": "<FILE_ID>",
  "master_type": "SLIDE_MASTER"
}
```

#### 参数说明

- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `master_type` (string, 可选): 母版类型：SLIDE_MASTER / NOTES_MASTER / HANDOUT_MASTER
- `url` (string, 可选): 文档 URL；与 link_id、file_id 三选一

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "data": {
      "text_styles": {
        "body": {},
        "title": {}
      }
    }
  }
}
```


---

## 5. wpp.master.properties.timeline

#### 功能说明

查询母版的动画时间线。

**幂等性**：是 — safe

#### 调用示例

查询母版动画时间线：

```json
{
  "file_id": "<FILE_ID>",
  "master_type": "SLIDE_MASTER"
}
```

#### 参数说明

- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `master_type` (string, 可选): 母版类型：SLIDE_MASTER / NOTES_MASTER / HANDOUT_MASTER
- `url` (string, 可选): 文档 URL；与 link_id、file_id 三选一

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "data": {
      "timeline": {
        "count": 0
      }
    }
  }
}
```


---

## 6. wpp.master.properties.transition

#### 功能说明

查询母版的切换动画。

**幂等性**：是 — safe

#### 调用示例

查询母版切换动画：

```json
{
  "file_id": "<FILE_ID>",
  "master_type": "SLIDE_MASTER"
}
```

#### 参数说明

- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `master_type` (string, 可选): 母版类型：SLIDE_MASTER / NOTES_MASTER / HANDOUT_MASTER
- `url` (string, 可选): 文档 URL；与 link_id、file_id 三选一

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "data": {
      "transition": {}
    }
  }
}
```
