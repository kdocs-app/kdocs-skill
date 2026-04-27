# otl.block_insert

## 1. otl.block_insert

#### 功能说明

向智能文档插入一个或多个块，适合在指定父块下按位置追加段落、列表、表格等结构化内容。



#### 操作约束

- **前置检查**：先 otl.block_query 了解文档块结构，确认插入位置
- **提示**：返回结果因内容和文档状态不同而异，以 code == 0 判断成功

#### 调用示例

在文档开头插入段落：

```json
{
  "file_id": "string",
  "params": {
    "blockId": "doc",
    "index": 1,
    "content": [
      {
        "type": "paragraph",
        "content": [
          {
            "type": "text",
            "content": "hello"
          }
        ]
      }
    ]
  }
}
```


#### 参数说明

- `file_id` (string, 必填): 智能文档文件 ID
- `params` (object, 必填): 插入操作
  - `blockId` (string, 常用): 目标父块 ID，例如 `doc`
  - `index` (integer, 常用): 插入位置索引。**当 `blockId='doc'` 时，`index=0` 指向 title 节点（传 0 会报 `insert position is invalid`）；正文插入起点为 `index>=1`**；精确值需先用 `otl.block_query` 获取目标父块真实结构
  - `content` (array, 常用): 待插入的块内容数组

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


# 在智能文档指定位置插入内容

调用`otl.block_insert`工具，传入智能文档的 `file_id` 和相关参数，可在指定位置插入内容，适合局部新增。

---

## 参数说明

- `file_id` (string, 必填): 智能文档文件 ID
- `params` (object, 必填): 插入操作
  - `blockId` (string, 必填): 目标父块 ID，为"doc"表示在文档中插入，其他块ID可通过查询块获得
  - `index` (integer, 必填): 插入位置索引
  - `content` (array, 必填): 待插入的节点数组，节点类型和属性可参考`references/otl/node.md`

---

## 调用示例

在文档开头插入段落节点

```json
{
  "params": {
    "blockId": "doc",
    "index": 1,
    "content": [
      {
        "type": "paragraph",
        "content": [
          {
            "type": "text",
            "content": "一些文字"
          }
        ]
      }
    ]
  }
}
```

将段落的首个子节点后插入文本节点

```json
{
  "params": {
    "blockId": "PARA_ID",
    "index": 1,
    "content": [
      {
        "type": "text",
        "content": "希望插入的文字"
      }
    ]
  }
}
```

## 典型用法

1. **文档内插入新内容**： blockId设置为"doc"，确定好插入index和插入的子节点。 插入子节点的类型和属性可参考 `references/otl/node.md`
2. **段落内插入新文本**: blockId设置为通过查询得到的段落id，确定好插入index和插入的子节点。插入子节点的类型和属性可参考 `references/otl/node.md`

--- 

## 注意事项

1. 当blockId为"doc"时，index必须大于等于1，因为doc的首个子节点必须是title（全局唯一），如果需要在正文开头插入内容，index应为1。 
2. 若查询到的块的content里包含rangeMarkBegin或rangeMarkEnd，计算index时应忽略它们，它们是虚拟节点。
