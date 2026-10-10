# 演示文稿 · 文档/文件属性

> facet: presentation.properties

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `wpp.presentation.properties.builtin_properties` | 查询演示文稿的内置文档属性 |  |
| `wpp.presentation.properties.close` | 关闭演示文稿（关闭当前演示文稿会话） |  |
| `wpp.presentation.properties.custom_properties` | 查询演示文稿的自定义文档属性 |  |
| `wpp.presentation.properties.page_setup` | 查询演示文稿的页面设置（页面宽高与方向） |  |
| `wpp.presentation.properties.read_only` | 查询演示文稿的只读状态 |  |
| `wpp.presentation.properties.remarks` | 查询演示文稿的全部备注 |  |
| `wpp.presentation.properties.saved` | 查询演示文稿的已保存状态 |  |
| `wpp.presentation.properties.template_name` | 查询演示文稿的模板名称 |  |

---

## 1. wpp.presentation.properties.builtin_properties

#### 功能说明

查询演示文稿的内置文档属性。

**幂等性**：是 — safe

#### 调用示例

查询演示文稿的内置文档属性：

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

## 2. wpp.presentation.properties.close

#### 功能说明

关闭演示文稿（关闭当前演示文稿会话）。

**幂等性**：是 — safe

#### 调用示例

关闭演示文稿：

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
  "data": {}
}
```


---

## 3. wpp.presentation.properties.custom_properties

#### 功能说明

查询演示文稿的自定义文档属性。

**幂等性**：是 — safe

#### 调用示例

查询演示文稿的自定义文档属性：

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

## 4. wpp.presentation.properties.page_setup

#### 功能说明

查询演示文稿的页面设置（页面宽高与方向）。

**幂等性**：是 — safe

#### 调用示例

查询演示文稿的页面设置：

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
      "page_setup": {
        "height": 540,
        "orientation": "LANDSCAPE",
        "width": 960
      }
    }
  }
}
```


---

## 5. wpp.presentation.properties.read_only

#### 功能说明

查询演示文稿的只读状态。

**幂等性**：是 — safe

#### 调用示例

查询演示文稿的只读状态：

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
      "read_only": false
    }
  }
}
```


---

## 6. wpp.presentation.properties.remarks

#### 功能说明

查询演示文稿的全部备注。

**幂等性**：是 — safe

#### 调用示例

查询演示文稿的全部备注：

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
      "remarks": {
        "count": 0
      }
    }
  }
}
```


---

## 7. wpp.presentation.properties.saved

#### 功能说明

查询演示文稿的已保存状态。

**幂等性**：是 — safe

#### 调用示例

查询演示文稿的已保存状态：

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
      "saved": false
    }
  }
}
```


---

## 8. wpp.presentation.properties.template_name

#### 功能说明

查询演示文稿的模板名称。

**幂等性**：是 — safe

#### 调用示例

查询演示文稿的模板名称：

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
      "template_name": "Office Theme"
    }
  }
}
```
