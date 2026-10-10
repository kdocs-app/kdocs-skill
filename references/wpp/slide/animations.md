# 幻灯片 · 动画

> facet: slide.animations

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `wpp.slide.animations.data` | 幻灯片动画动画实体：支持 删除/插入 | `effect_id`, `effect_index`, `shape_id`, `slide_id`, `verb` |
| `wpp.slide.animations.effects` | 查询幻灯片动画动画效果列表 | `slide_id` |
| `wpp.slide.animations.order` | 设置幻灯片动画动画顺序 | `effect_index`, `slide_id`, `to_pos` |
| `wpp.slide.animations.timeline` | 查询幻灯片动画动画时间线 | `slide_id` |
| `wpp.slide.animations.timing` | 设置幻灯片动画动画时序 | `effect_index`, `slide_id`, `timing` |
| `wpp.slide.animations.trigger` | 设置幻灯片动画触发方式 | `effect_index`, `slide_id`, `trigger` |
| `wpp.slide.animations.types` | 查询幻灯片动画动画类型列表 | `slide_id` |

---

## 1. wpp.slide.animations.data

#### 功能说明

幻灯片动画动画实体：支持 删除/插入。

**幂等性**：是 — safe

#### 调用示例

为幻灯片形状插入淡入动画：

```json
{
  "file_id": "<FILE_ID>",
  "slide_id": 150995203,
  "shape_id": 2,
  "effect_id": 10,
  "effect_type": "ENTRANCE",
  "trigger": 1,
  "verb": "insert"
}
```

删除幻灯片动画节点：

```json
{
  "file_id": "<FILE_ID>",
  "slide_id": 150995203,
  "effect_index": 1,
  "verb": "delete"
}
```

#### 参数说明

- `effect_id` (number, 必填): 动画效果 ID（MsoAnimEffect 整型，如 Fade=10）
- `effect_index` (number, 必填): 动画节点索引（1 起）
- `effect_type` (string, 可选): 动画效果类型：ENTRANCE / EMPHASIS / EXIT / PATH
- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `index` (number, 可选): 插入位置索引
- `level` (number, 可选): 动画分级（MsoAnimateByLevel，默认 0）
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `shape_id` (number, 必填): 形状 ID
- `slide_id` (number, 必填): 幻灯片 ID
- `trigger` (number, 可选): 触发方式：1 单击 / 2 与上一动画同时 / 3 上一动画之后 / 4 单击形状
- `url` (string, 可选): 文档 URL；与 link_id、file_id 三选一
- `verb` (string, 必填): HTTP 动作：delete（删除） / insert（插入）。可选值：`delete` / `insert`

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

## 2. wpp.slide.animations.effects

#### 功能说明

查询幻灯片动画动画效果列表。

**幂等性**：是 — safe

#### 调用示例

查询幻灯片动画效果列表：

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
      "count": 1,
      "effects": [
        {
          "delay": 5,
          "display_name": "标题 1",
          "duration": 1,
          "effect_index": 1,
          "effect_type": 10,
          "shape_id": 2,
          "trigger": 2
        }
      ]
    }
  }
}
```


---

## 3. wpp.slide.animations.order

#### 功能说明

设置幻灯片动画动画顺序。

**幂等性**：是 — safe

#### 调用示例

调整幻灯片动画播放顺序：

```json
{
  "file_id": "<FILE_ID>",
  "slide_id": 150995203,
  "effect_index": 1,
  "to_pos": 1
}
```

#### 参数说明

- `effect_index` (number, 必填): 动画节点索引（1 起）
- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `slide_id` (number, 必填): 幻灯片 ID
- `to_pos` (number, 必填): 目标位置（1 起）
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

## 4. wpp.slide.animations.timeline

#### 功能说明

查询幻灯片动画动画时间线。

**幂等性**：是 — safe

#### 调用示例

查询幻灯片动画时间线：

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
      "count": 1,
      "effects": [
        {
          "delay": 5,
          "display_name": "标题 1",
          "duration": 1,
          "effect_index": 1,
          "effect_type": 10,
          "shape_id": 2,
          "trigger": 2
        }
      ]
    }
  }
}
```


---

## 5. wpp.slide.animations.timing

#### 功能说明

设置幻灯片动画动画时序。

**幂等性**：是 — safe

#### 调用示例

设置幻灯片动画时序（时长/延迟/重复）：

```json
{
  "file_id": "<FILE_ID>",
  "slide_id": 150995203,
  "effect_index": 1,
  "timing": {
    "duration": 0.8,
    "delay": 0.2,
    "repeat_count": 1,
    "speed": 1
  }
}
```

#### 参数说明

- `effect_index` (number, 必填): 动画节点索引（1 起）
- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `slide_id` (number, 必填): 幻灯片 ID
- `timing` (object, 必填): 动画 TimeNode 对象：{duration, delay, repeat_count, speed}
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

## 6. wpp.slide.animations.trigger

#### 功能说明

设置幻灯片动画触发方式。

**幂等性**：是 — safe

#### 调用示例

设置幻灯片动画触发方式为与上一动画同时：

```json
{
  "file_id": "<FILE_ID>",
  "slide_id": 150995203,
  "effect_index": 1,
  "trigger": 2
}
```

#### 参数说明

- `effect_index` (number, 必填): 动画节点索引（1 起）
- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一
- `slide_id` (number, 必填): 幻灯片 ID
- `trigger` (number, 必填): 触发方式：1 单击 / 2 与上一动画同时 / 3 上一动画之后 / 4 单击形状
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

## 7. wpp.slide.animations.types

#### 功能说明

查询幻灯片动画动画类型列表。

**幂等性**：是 — safe

#### 调用示例

查询幻灯片动画类型列表：

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
      "count": 1,
      "types": [
        {
          "display_name": "标题 1",
          "effect_index": 1,
          "effect_type": 10,
          "shape_id": 2
        }
      ]
    }
  }
}
```
