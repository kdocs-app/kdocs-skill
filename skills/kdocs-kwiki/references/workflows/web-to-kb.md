# 网页内容存入知识库

> 将网页内容剪藏后存入指定知识库

**适用场景**：把这篇公众号文章存入知识库

**触发词**：存入知识库、保存到知识库、剪藏到知识库

- 用户要求将网页保存到知识库

**工具链**：`scrape_url` → `scrape_progress` → `move_file` → `get_file_link`

## 涉及工具

| 工具 | 服务 | 用途 |
|------|------|------|
| `scrape_url` | drive | 提交网页剪藏任务 |
| `scrape_progress` | drive | 轮询剪藏任务进度 |
| `move_file` | drive | 将剪藏文件移入知识库 |
| `get_file_link` | drive | 获取文档在线链接 |

## 执行流程

> **触发示例**：「把公众号文章存入XX知识库」「把这个链接存到知识库里」

**流程**
1. 主流程：scrape_url → scrape_progress(status=1) → move_file → get_file_link
2. 降级流程（scrape_url 失败/status=-1，如公众号等 JS 渲染页面）：
   browser 抓取正文
   → create_file_with_content(name="标题", file_extension="docx", content=<抓取 Markdown 明文>)
   → move_file → get_file_link

| 注意 | 说明 |
|------|------|
| 降级新建 | 用 create_file_with_content；content 为 Markdown 明文，不需要进行 Base64 编码 |
| ID 体系 | kwiki 内部 kuid 需通过 `kwiki.list_items` 获取 |

调用接口前先执行以下命令获取接口参数及约束：
`kdocs-cli help drive.scrape_url drive.scrape_progress drive.move_file drive.get_file_link`
