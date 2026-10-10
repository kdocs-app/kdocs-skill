# 导出

> facet: export

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| `wpp.export.export.image` | 导出在线演示文稿为图片（v3 同步导出，返回云存储 url/key） |  |
| `wpp.export.export.pdf` | 导出在线演示文稿为 PDF（v3 异步导出；创建任务或按 task_id 轮询） |  |

---

## 1. wpp.export.export.image

#### 功能说明

导出在线演示文稿为图片（v3 同步导出，返回云存储 url/key）。

**幂等性**：是 — safe

#### 调用示例

导出演示文稿为图片：

```json
{
  "link_id": "<LINK_ID>",
  "image_type": "PNG",
  "from_slide": 1,
  "to_slide": 1
}
```

#### 参数说明

- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一
- `from_slide` (number, 可选): 起始页，默认 1
- `image_type` (string, 可选): 图片类型：PNG / JPG，默认 PNG
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一（推荐）
- `to_slide` (number, 可选): 结束页，默认 9999
- `url` (string, 可选): 文档 URL；与 link_id、file_id 三选一
- `dpi` (number, 可选): 如 96、150、300，默认 96
- `combine_long_pic` (boolean, 可选): 是否合并为长图
- `water_mark` (boolean, 可选): 是否含水印，默认 true
- `client_id` (string, 可选): 客户端标识
- `password` (string, 可选): 文档打开密码
- `store_type` (string, 可选): 如 ks3、cloud
- `use_xva` (boolean, 可选): 是否使用 XVA 渲染

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "key": "export/xxx.png",
    "url": "https://example.com/out.png"
  }
}
```


---

## 2. wpp.export.export.pdf

#### 功能说明

导出在线演示文稿为 PDF（v3 异步导出；创建任务或按 task_id 轮询）。

**幂等性**：是 — safe

#### 调用示例

导出演示文稿为 PDF：

```json
{
  "file_id": "<FILE_ID>",
  "from_slide": 1,
  "to_slide": 1
}
```

#### 参数说明

- `file_id` (string, 可选): 文件 id；与 url、link_id 三选一（创建任务）
- `from_slide` (number, 可选): 起始页，默认 1
- `link_id` (string, 可选): 分享 id；与 url、file_id 三选一（创建任务）
- `to_slide` (number, 可选): 结束页，默认 9999
- `url` (string, 可选): 文档 URL；与 link_id、file_id 三选一（创建任务）
- `task_id` (string, 可选): 轮询时必填
- `task_type` (string, 可选): 轮询时填 normal_export
- `client_id` (string, 可选): 客户端标识
- `export_modify_password` (string, 可选): 导出 PDF 修改密码
- `export_open_password` (string, 可选): 导出 PDF 打开密码
- `multipage` (number, 可选): 默认 1
- `opt_frame` (boolean, 可选): 是否优化帧，默认 true
- `password` (string, 可选): 源文件密码
- `store_type` (string, 可选): 如 ks3

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "task_id": "abc123"
  }
}
```
