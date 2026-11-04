# aippt-upload-local-file

# 本地文件上云（AIPPT 前置）

单文档 / 多文档路由在拿到**云文档 id** 前，若用户给的是**本地文件路径**，须先上云。本 Skill 使用 **`drive.upload_new_file`**（kdocs-cli `drive upload_new_file`）。

## 参数与限制

参数、支持的后缀与错误码以 `kdocs-cli help drive.upload_new_file` 为准；本文只讲本 Skill 的调用时机与结果用法。

## 何时调用

- 路由② / ③ 中，附件为本地路径（非 `*.kdocs.cn/l/{id}`、非已有云文档 id）
- **不要**为可传入格式本地读正文做摘要；只需读取文件二进制做 Base64 上云

## 命令（kdocs-cli）

```bash
kdocs-cli drive upload_new_file --name "报告.docx" --content_base64 "<文件二进制 Base64>"
```

可选：`--parent_id`（目标文件夹 id，规则见 [kdocs-drive 的目录坐标与文件定位说明](../../kdocs-drive/references/file-locating-guide.md)）。

## 成功响应怎么用

- 取 `data.link_url` **路径末尾**的短链 id 作为 v7 云文档 id
- 单文档：下一轮 `aippt.execute_round` 的 `files[].file_type=v7_file_id`，`file_id` 填该 id
- 多文档：对每个本地文件重复上云，再 `action=import_aidocs`（见 `references/workflows/aippt-multi-document.md`）

## 失败时

上云失败则**停止**，向用户说明原因；**不要**在没有云文档 id 时调用 `aippt.execute_round`。

## 相关文档

- 盘位与 `parent_id`：[kdocs-drive 的目录坐标与文件定位说明](../../kdocs-drive/references/file-locating-guide.md)
- 路由与附件过滤：`SKILL.md`
- 单文档编排：`references/workflows/aippt-single-document.md`
