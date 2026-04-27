# 演示文稿与页面

## 1. wpp.insert_slide

#### 功能说明

在**已有**在线演示文稿中插入一页空白幻灯片。

**适用于**：已存在、可编辑的 WPP 演示（`file_id` 对应在线演示文档）。



#### 调用示例

在第 1 页后插入空白页：

```json
{
  "file_id": "string",
  "slide_idx": 1,
  "layer_type": 65538,
  "layout_Id": 134217788
}
```


#### 参数说明

- `file_id` (string, 必填): 演示文稿 file_id
- `slide_idx` (integer, 必填): 插入位置的幻灯片序号，**从 0 开始**
- `layer_type` (integer, 可选): 版式类型，固定值 `65538`
- `layout_Id` (integer, 可选): 指定模板版式 ID，从对应版式新建幻灯片（如结束页、标题页、内容页等）。版式 ID 与演示文稿模板中定义的版式一一对应，不同模板的可用 ID 不同

#### 返回值说明

```json
{
  "result": "ok",
  "detail": {
    "res": [
      {
        "cmdName": "slideInsert",
        "code": 0,
        "errName": "S_OK",
        "msg": "execute result",
        "token": "string"
      }
    ]
  }
}

```

| 字段 | 类型 | 说明 |
|------|------|------|
| `result` | string | 请求成功为 `ok` |
| `detail.res[].cmdName` | string | 内核命令名，如 `slideInsert` |
| `detail.res[].code` | integer | `0` 表示该条执行成功 |
| `detail.res[].errName` | string | 如 `S_OK` |
| `detail.res[].token` | string | 可选，会话/追踪用 |


---

