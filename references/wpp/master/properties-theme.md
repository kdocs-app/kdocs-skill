# 母版 · 主题与版式

> facet: master.properties

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `wpp.master.properties.background` | 查询母版的背景（含填充类型与前景色 |  |
| `wpp.master.properties.color_scheme` | 查询母版的配色方案 |  |
| `wpp.master.properties.custom_layouts` | 查询母版的自定义版式（返回母版下全部自定义版式） |  |
| `wpp.master.properties.delete` | 删除母版（删除指定类型母版） |  |
| `wpp.master.properties.design` | 查询母版的设计 |  |
| `wpp.master.properties.name` | 母版的名称：支持 查询/设置（母版名称） | `verb` |
| `wpp.master.properties.size` | 查询母版的页面尺寸（幻灯片页面宽高） |  |
| `wpp.master.properties.theme` | 查询母版的主题 |  |

---

## 1. wpp.master.properties.background

#### 功能说明

查询母版的背景（含填充类型与前景色；需 master_type 指定幻灯片/备注/讲义母版）。

**幂等性**：是 — safe

#### 调用示例

查询母版背景：

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
      "background": {
        "fill_type": 1,
        "fore_color": 7949855
      }
    }
  }
}
```


---

## 2. wpp.master.properties.color_scheme

#### 功能说明

查询母版的配色方案。

**幂等性**：是 — safe

#### 调用示例

查询母版配色方案：

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
    "data": {}
  }
}
```


---

## 3. wpp.master.properties.custom_layouts

#### 功能说明

查询母版的自定义版式（返回母版下全部自定义版式）。

**幂等性**：是 — safe

#### 调用示例

查询母版自定义版式：

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
      "custom_layouts": [
        {
          "index": 1,
          "name": "Title Slide"
        },
        {
          "index": 2,
          "name": "Title and Content"
        }
      ]
    }
  }
}
```


---

## 4. wpp.master.properties.delete

#### 功能说明

删除母版（删除指定类型母版）。

**幂等性**：是 — safe

#### 调用示例

删除母版：

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
  "data": {}
}
```


---

## 5. wpp.master.properties.design

#### 功能说明

查询母版的设计。

**幂等性**：是 — safe

#### 调用示例

查询母版设计：

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
      "design": {}
    }
  }
}
```


---

## 6. wpp.master.properties.name

#### 功能说明

母版的名称：支持 查询/设置（母版名称）。

**幂等性**：是 — safe

#### 调用示例

查询母版名称：

```json
{
  "file_id": "<FILE_ID>",
  "master_type": "SLIDE_MASTER",
  "verb": "query"
}
```

设置母版名称：

```json
{
  "file_id": "<FILE_ID>",
  "master_type": "SLIDE_MASTER",
  "verb": "update",
  "name": "批次D母版测试名"
}
```

#### 参数说明

- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `master_type` (string, 可选): 母版类型：SLIDE_MASTER / NOTES_MASTER / HANDOUT_MASTER
- `url` (string, 可选): 文档 URL；与 link_id、file_id 三选一
- `verb` (string, 必填): HTTP 动作：query（查询） / update（设置）。可选值：`query` / `update`

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "data": {
      "name": "Office Theme"
    }
  }
}
```


---

## 7. wpp.master.properties.size

#### 功能说明

查询母版的页面尺寸（幻灯片页面宽高）。

**幂等性**：是 — safe

#### 调用示例

查询母版页面尺寸：

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
      "size": {
        "height": 540,
        "width": 960
      }
    }
  }
}
```


---

## 8. wpp.master.properties.theme

#### 功能说明

查询母版的主题。

**幂等性**：是 — safe

#### 调用示例

查询母版主题：

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
      "theme": {
        "name": "Theme"
      }
    }
  }
}
```
