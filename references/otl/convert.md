# otl.convert

## 1. otl.convert

#### 功能说明

将 HTML、Markdown 等内容转换为智能文档块结构，适合在正式插入前先生成可复用的块内容。


> 仅做格式转换，不会修改文档内容；需配合 `otl.block_insert` 才能将转换结果写入文档
> `format` 只支持 `"html"` 和 `"markdown"` 两种值
> 返回结果中 `blocks` 字段为转换得到的块数组，可直接用于 `otl.block_insert`；块类型和属性说明见 `references/otl/node.md`

#### 调用示例

将 Markdown 内容转为块数据：

```json
{
  "file_id": "string",
  "params": {
    "format": "markdown",
    "content": "# 标题\n\n段落内容"
  }
}
```

将 HTML 内容转为块数据：

```json
{
  "file_id": "string",
  "params": {
    "format": "html",
    "content": "<h1>标题</h1><p>段落内容</p>"
  }
}
```


#### 参数说明

- `file_id` (string, 必填): 智能文档文件 ID
- `params` (object, 必填): 转换参数对象。根据待转换内容类型填写对应字段

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


# 将 HTML/Markdown 数据转为智能文档的块数据

调用 `otl.convert` 工具，将 HTML 或 Markdown 内容转换为智能文档块结构。适合在正式插入前先生成可复用的块内容。

## 典型用法

1. **外部内容转块后插入**：先调用 `otl.convert` 将 HTML/Markdown 转为块结构，再调用 `otl.block_insert` 将转换结果插入文档
2. **预览转换结果**：在正式写入前，通过 convert 预先查看块结构是否符合预期
