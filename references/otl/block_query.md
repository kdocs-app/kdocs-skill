# otl.block_query

## 1. otl.block_query

#### 功能说明

查询指定块的结构与内容，适合在更新前先读取目标块信息。



#### 调用示例

查询全文：

```json
{
  "file_id": "string",
  "params": {
    "blockIds": [
      "doc"
    ]
  }
}
```

查询指定块：

```json
{
  "file_id": "string",
  "params": {
    "blockIds": [
      "目标blockId"
    ]
  }
}
```


#### 参数说明

- `file_id` (string, 必填): 智能文档文件 ID
- `params` (object, 必填): 查询参数对象
  - `blockIds` (array, 常用): 要查询的块 ID 列表

#### 返回值说明

```json
{
  "code": 0,
  "msg": "ok",
  "data": {
    "...": "..."
  }
}

```


---


# 读取智能文档中的块

调用 `otl.block_query` 工具，传入智能文档的 `file_id` 和查询参数，即可读取指定块的结构与内容。适合在更新或删除前先获取目标块信息。

---

## 参数说明

- `file_id` (string, 必填): 智能文档文件 ID
- `params` (object, 必填): 查询参数对象
  - `blockIds` (array, 必填): 要查询的块 ID 列表， 块ID为"doc"可查询完整文档块

---

## 调用示例

查询文档根块：

```json
{
  "params": {
    "blockIds": ["doc"]
  }
}
```

查询多个指定块：

```json
{
  "params": {
    "blockIds": ["blockId_1", "blockId_2"]
  }
}
```

---

## 典型用法

1. **获取文档完整数据**：`blockIds` 传 `["doc"]`，返回文档块（含各子块的 `blockId`、类型等）
2. **查询子块**：从步骤 1 的返回结果中取出子块 ID，在需要时再次调用 `otl.block_query` 查询该子块

---

## 注意事项

1. 查询得到的块类型和属性具体含义可参考 `references/otl/node.md`
