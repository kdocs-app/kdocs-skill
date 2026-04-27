# AI PPT（aippt）工具完整参考文档

本文件包含金山文档 Skill 中 **AI PPT** 相关工具的 MCP API 调用方法、脚本模板和中间文件约定。

**适用范围**：`aippt.theme_*`（主题生成 PPT）与 `aippt.doc_*`（文档生成 PPT）两类场景。

---

## 推荐调用链路

### 主题生成 PPT

```
aippt.theme_questions → aippt.theme_deep_research → aippt.theme_outline → 本地格式转换 → aippt.theme_generate_html_pptx → upload_file
```

详细工作流见 `references/workflows/topic-ppt.md`

### 文档生成 PPT

```
aippt.doc_create_session → aippt.doc_outline_options → aippt.doc_outline → aippt.doc_beautify → 本地格式转换 → aippt.doc_generate_ppt → upload_file
```

详细工作流见 `references/workflows/doc-ppt.md`

### 使用建议

| 场景 | 建议 |
|------|------|
| 用户只给一句主题 | 先 `aippt.theme_questions`，补齐受众、场景、风格、重点 |
| 用户已经给出文档 | 使用 `aippt.doc_create_session` → `aippt.doc_outline_options` → `aippt.doc_outline` |
| 需要更可靠的事实资料 | 衔接 `aippt.theme_deep_research` |
| 先看结构再决定生成 | 先停在 `aippt.theme_outline` 或 `aippt.doc_outline` |
| 需要文档直出最终演示文件 | `aippt.doc_beautify` → 本地格式转换 → `aippt.doc_generate_ppt` |
| 需要最终演示文件 | 两条链路最终都需要 `{topic, outlines[]}`，主题链路用 `aippt.theme_generate_html_pptx`，文档链路用 `aippt.doc_generate_ppt` |

---

## MCP 调用规则

### 注意事项

- 通过 `mcporter` 或 `kdocs-cli` 调用本流程中的步骤时，请为每一步单独设置超时时间为 **1800000 毫秒**
- `aippt.theme_deep_research` 的完整研究内容主要通过流式通知产生，最终 tool result 只保留摘要
- `aippt.theme_outline` 返回的 `outline` 结构可能因上游模板而变化，传给 `aippt.theme_generate_html_pptx` 时应优先使用其中正式的 `outlines` 数组
- `aippt.doc_outline` 返回的 `markdown_outline` 可能包含重复内容（大纲出现两次），建议从 `assistant_messages` 中提取最长的含 `{.topic}` 和 `{.end}` 的消息作为清洁版大纲
- `aippt.doc_outline` 的 `resume_info` 必须是**数组**，格式为 `[{type:"follow_up", id:<interrupt_id>, data:{items:[...]}}]`
- **文档链路同样需要本地格式转换**：将 `03_outline.md` + `04_beautify.json` 合并为 `{topic, outlines[]}` 后再传入 `aippt.doc_generate_ppt`
- `aippt.doc_generate_ppt` 接收 `{topic, outlines[]}` 格式，与 `aippt.theme_generate_html_pptx` 底层接口一致
- 生成速度约每页 60 秒，20 页以上的 PPT 生成可能耗时 20-30 分钟
- `aippt.theme_generate_html_pptx` 返回的下载链接通常带时效性，建议尽快消费
- `deep_research` 的 `references`、`outline` 结果、Markdown 大纲、格式转换后的 `outlines` 等内容可能很长，**必须先写入本地文件**，后续步骤再读取文件内容，避免命令行参数超过长度限制

### mcporter短参数直接调用

当参数 JSON 较短（< 2000 字符）时可直接命令行调用：

```bash
mcporter call kdocs aippt.theme_questions --args '{"input":"长颈鹿科普PPT"}' --output json --timeout 1800000
```

### mcporter长参数脚本调用

当参数 JSON 超过 2000 字符，或包含 `references`、`outlines`、`content_base64` 等长字段时，**必须使用脚本方式调用**。

#### 脚本生成与调用流程

1. 将工具参数写入 JSON 文件（UTF-8），如 `$AIPPT_WORK_DIR/03_outline_args.json`
2. 在会话临时目录生成调用脚本 `$AIPPT_WORK_DIR/_call_mcp.js`（首次生成后复用）
3. 执行：`node $AIPPT_WORK_DIR/_call_mcp.js <toolName> <argsFile> [outputFile] [timeoutMs] [serverName]`

#### 脚本模板 `_call_mcp.js`

```javascript
const { spawnSync, execSync } = require('child_process');
const fs = require('fs');
const path = require('path');
const os = require('os');

const toolName = process.argv[2];
const argsFile = process.argv[3];
const outputFile = process.argv[4] || '';
const timeoutMs = parseInt(process.argv[5] || '1800000', 10);
const serverName = process.argv[6] || 'kdocs';
const STDOUT_LIMIT = 8000;
const TIMEOUT_BUFFER = 1800000;

if (!toolName || !argsFile) {
  console.error('用法: node _call_mcp.js <toolName> <argsFile> [outputFile] [timeoutMs] [serverName]');
  process.exit(1);
}

function findMcporterCli() {
  const candidates = [];
  if (process.env.MCPORTER_CLI) candidates.push(process.env.MCPORTER_CLI);
  const nodeDir = path.dirname(process.execPath);
  candidates.push(path.join(nodeDir, 'node_modules', 'mcporter', 'dist', 'cli.js'));
  const nvmDir = process.env.NVM_SYMLINK || path.join(os.homedir(), 'AppData', 'Roaming', 'nvm');
  if (process.platform === 'win32' && fs.existsSync(nvmDir)) {
    try {
      fs.readdirSync(nvmDir).filter(d => d.startsWith('v')).forEach(d => {
        candidates.push(path.join(nvmDir, d, 'node_modules', 'mcporter', 'dist', 'cli.js'));
      });
    } catch (_) {}
  }
  try {
    const g = execSync('npm root -g', { encoding: 'utf-8', timeout: 5000 }).trim();
    candidates.push(path.join(g, 'mcporter', 'dist', 'cli.js'));
  } catch (_) {}
  if (process.platform === 'win32') {
    candidates.push('C:\\Program Files\\nodejs\\node_modules\\mcporter\\dist\\cli.js');
  } else {
    candidates.push('/usr/local/lib/node_modules/mcporter/dist/cli.js');
    candidates.push('/usr/lib/node_modules/mcporter/dist/cli.js');
  }
  for (const p of candidates) { if (p && fs.existsSync(p)) return p; }
  return null;
}

const argsPath = path.resolve(argsFile);
if (!fs.existsSync(argsPath)) { console.error('[ERROR] 参数文件不存在:', argsPath); process.exit(1); }
const argsContent = fs.readFileSync(argsPath, 'utf-8').trim();
try { JSON.parse(argsContent); } catch (e) { console.error('[ERROR] JSON 不合法:', e.message); process.exit(1); }

const mcporterCli = findMcporterCli();
let result;
if (mcporterCli) {
  console.error('[INFO] mcporter cli:', mcporterCli);
  result = spawnSync(process.execPath, [
    mcporterCli, 'call', serverName, toolName,
    '--args', argsContent, '--output', 'json', '--timeout', String(timeoutMs)
  ], { encoding: 'utf-8', maxBuffer: 50 * 1024 * 1024, timeout: timeoutMs + TIMEOUT_BUFFER });
} else {
  console.error('[INFO] 尝试通过 PATH 调用 mcporter');
  result = spawnSync('mcporter', [
    'call', serverName, toolName,
    '--args', argsContent, '--output', 'json', '--timeout', String(timeoutMs)
  ], { encoding: 'utf-8', maxBuffer: 50 * 1024 * 1024, timeout: timeoutMs + TIMEOUT_BUFFER, shell: process.platform === 'win32' });
}

console.error('[INFO] Exit code:', result.status);
if (result.error) { console.error('[ERROR]', result.error.message); process.exit(1); }
const output = result.stdout || '';
if (outputFile && output.length > 0) {
  const outPath = path.resolve(outputFile);
  fs.mkdirSync(path.dirname(outPath), { recursive: true });
  fs.writeFileSync(outPath, output, 'utf-8');
  console.error('[INFO] 结果已写入:', outPath, '(' + output.length + ' bytes)');
}
if (output) {
  if (output.length > STDOUT_LIMIT) {
    process.stdout.write(output.substring(0, STDOUT_LIMIT));
    console.error('[INFO] stdout 已截断 (' + output.length + ' -> ' + STDOUT_LIMIT + ')');
  } else { process.stdout.write(output); }
}
if (result.stderr) {
  const cleaned = result.stderr.split('\n')
    .filter(l => !l.includes('ExperimentalWarning') && !l.includes('--experimental'))
    .join('\n').trim();
  if (cleaned) console.error(cleaned);
}
process.exit(result.status || 0);
```

#### 调用示例

```bash
# 调用 aippt.theme_outline（参数含长 references）
node $AIPPT_WORK_DIR/_call_mcp.js aippt.theme_outline $AIPPT_WORK_DIR/03_outline_args.json $AIPPT_WORK_DIR/03_outline.json 1800000

# 调用 aippt.theme_generate_html_pptx（参数含长 outlines）
node $AIPPT_WORK_DIR/_call_mcp.js aippt.theme_generate_html_pptx $AIPPT_WORK_DIR/04_config.json $AIPPT_WORK_DIR/05_ppt_result.json 1800000

# 调用 aippt.doc_outline（参数含长 resume_info）
node $AIPPT_WORK_DIR/_call_mcp.js aippt.doc_outline $AIPPT_WORK_DIR/03_doc_outline_args.json $AIPPT_WORK_DIR/03_outline.md 1800000
```

#### 关键设计要点

| 要点 | 说明 |
| --- | --- |
| 参数从文件读取 | 绕过 OS 命令行长度限制（Windows ~8191 字符） |
| `spawnSync` + `process.execPath` | 直接运行 `cli.js`，避免 PATH 查找失败 |
| 自动探测 mcporter 路径 | 按优先级搜索：环境变量 → Node 同级 → nvm → npm root -g → 默认路径 |
| `maxBuffer: 50MB` | 防止大输出被截断 |
| 超时缓冲 +30 分钟 | mcporter 内部超时与外层进程超时之间留有余量 |
| stdout 截断 8000 字符 | 避免终端刷屏，完整内容写入输出文件 |
| 过滤 ExperimentalWarning | 清除 Node.js 实验性功能噪声日志 |
| 兜底 PATH 调用 | cli.js 未找到时，退化为通过 `mcporter` 命令名调用 |

### upload_file 编程 API 调用

> **`upload_file` 无法通过 `mcporter call` 命令行调用。**
> PPTX 文件转 Base64 后 `content_base64` 字段通常有几十万字符，超出 OS 命令行长度限制。
> **唯一可行方式**：在 Node.js 脚本中直接调用 mcporter 的编程 API `callOnce()`。

#### 下载脚本 `_download_pptx.js`

```javascript
const https = require('https');
const http = require('http');
const fs = require('fs');
const path = require('path');

const workDir = process.argv[2];
const resultFile = path.join(workDir, '05_ppt_result.json');
const result = JSON.parse(fs.readFileSync(resultFile, 'utf-8'));
const mergedUrl = result.data
  ? result.data.merged_url
  : JSON.parse(result.content[0].text).data.merged_url;

const pptxFile = path.join(workDir, 'downloaded.pptx');
const driveId = process.argv[3];
const pptName = process.argv[4] || '演示文稿.pptx';

function download(url, dest) {
  return new Promise((resolve, reject) => {
    const mod = url.startsWith('https') ? https : http;
    const file = fs.createWriteStream(dest);
    mod.get(url, { timeout: 60000 }, (res) => {
      if (res.statusCode >= 300 && res.statusCode < 400 && res.headers.location) {
        file.close();
        fs.unlinkSync(dest);
        return download(res.headers.location, dest).then(resolve).catch(reject);
      }
      if (res.statusCode !== 200) { file.close(); return reject(new Error('HTTP ' + res.statusCode)); }
      res.pipe(file);
      file.on('finish', () => { file.close(); resolve(); });
    }).on('error', reject);
  });
}

async function main() {
  await download(mergedUrl, pptxFile);
  const base64 = fs.readFileSync(pptxFile).toString('base64');
  const uploadArgs = {
    drive_id: driveId,
    parent_id: '0',
    name: pptName,
    content_base64: base64
  };
  fs.writeFileSync(path.join(workDir, '06_upload_args.json'), JSON.stringify(uploadArgs), 'utf-8');
  console.log('OK');
}
main().catch(e => { console.error(e); process.exit(1); });
```

调用：`node $AIPPT_WORK_DIR/_download_pptx.js $AIPPT_WORK_DIR <drive_id> "<主题>.pptx"`

#### 上传脚本 `_upload_cloud.js`

```javascript
const fs = require('fs');
const path = require('path');
const os = require('os');

const argsFile = process.argv[2];
const outputFile = process.argv[3] || '';

async function findMcporterModule() {
  const candidates = [];
  const nodeDir = path.dirname(process.execPath);
  candidates.push(path.join(nodeDir, 'node_modules', 'mcporter', 'dist', 'index.js'));
  const nvmDir = process.env.NVM_SYMLINK || path.join(os.homedir(), 'AppData', 'Roaming', 'nvm');
  if (process.platform === 'win32' && fs.existsSync(nvmDir)) {
    try {
      fs.readdirSync(nvmDir).filter(d => d.startsWith('v')).forEach(d => {
        candidates.push(path.join(nvmDir, d, 'node_modules', 'mcporter', 'dist', 'index.js'));
      });
    } catch (_) {}
  }
  if (process.platform === 'win32') {
    candidates.push('C:\\Program Files\\nodejs\\node_modules\\mcporter\\dist\\index.js');
  } else {
    candidates.push('/usr/local/lib/node_modules/mcporter/dist/index.js');
    candidates.push('/usr/lib/node_modules/mcporter/dist/index.js');
  }
  for (const p of candidates) {
    if (fs.existsSync(p)) {
      return await import('file:///' + p.replace(/\\/g, '/'));
    }
  }
  throw new Error('mcporter module not found');
}

async function main() {
  const args = JSON.parse(fs.readFileSync(path.resolve(argsFile), 'utf-8'));
  const mcporter = await findMcporterModule();

  const result = await mcporter.callOnce({
    server: 'kdocs',
    toolName: 'upload_file',
    args: args
  });

  const output = JSON.stringify(result, null, 2);
  if (outputFile) {
    fs.writeFileSync(path.resolve(outputFile), output, 'utf-8');
  }
  const limit = 8000;
  process.stdout.write(output.length > limit ? output.substring(0, limit) : output);
}

main().catch(e => { console.error('[ERROR]', e.message); process.exit(1); });
```

调用：`node $AIPPT_WORK_DIR/_upload_cloud.js $AIPPT_WORK_DIR/06_upload_args.json $AIPPT_WORK_DIR/06_upload_result.json`

### 各调用方式适用范围

| 工具 / 步骤 | `kdocs-cli … @file` | `mcporter call` 命令行 | `_call_mcp.js` 脚本 | `callOnce()` 编程 API |
| --- | --- | --- | --- | --- |
| `aippt.theme_questions` | 可以（参数短，也可不用 `@file`） | 可以（参数短） | 可以 | 可以 |
| `aippt.theme_deep_research` | 可以（参数短，也可不用 `@file`） | 可以（参数短） | 可以 | 可以 |
| `aippt.theme_outline` | **推荐**（`@file` 绕过长度限制） | **不推荐**（`references` 可能很长） | **推荐** | 可以 |
| `aippt.theme_generate_html_pptx` | **推荐**（`@file` 绕过长度限制） | **不推荐**（`outlines` 可能很长） | **推荐** | 可以 |
| `aippt.doc_create_session` | 可以（参数短，也可不用 `@file`） | 可以（参数短） | 可以 | 可以 |
| `aippt.doc_outline_options` | 可以（参数短，也可不用 `@file`） | 可以（参数短） | 可以 | 可以 |
| `aippt.doc_outline` | **推荐**（`@file` 绕过长度限制） | **不推荐**（`resume_info` 可能较长） | **推荐** | 可以 |
| `aippt.doc_beautify` | **推荐**（`@file` 绕过长度限制） | **不推荐**（`outline` 可能很长） | **推荐** | 可以 |
| `aippt.doc_generate_ppt` | **推荐**（`@file` 绕过长度限制） | **不推荐**（`outline`+`beautify` 很长） | **推荐** | 可以 |
| `upload_file`（含 `content_base64`） | **推荐**（`@file` 从磁盘读取，绕过 OS 限制） | **不可用**（超出 OS 命令行限制） | **不可用**（`spawnSync` 同样超限） | **mcporter必须使用** |
| `get_file_link` / `search_files` 等 | 可以（参数短，也可不用 `@file`） | 可以（参数短） | 可以 | 可以 |

---

## 临时文件管理

所有中间产物必须格式规范，使用 UTF-8 格式，写入基于当前 Agent **conversation ID**（对话标识）的独立临时目录，避免污染工作区或多会话并行干扰。

> ⚠️ 此处 conversation ID 是指 Agent 当前对话的唯一标识，**不是** `aippt.doc_create_session` 返回的 API `session_id`。获取方式见工作流文档步骤 0。**禁止**使用自定义名称、缩写或硬编码字符串。

### 目录规则

| 项目 | 说明 |
|------|------|
| 根目录 | 系统临时目录，即 `os.tmpdir()`（Windows: `%TEMP%`，macOS/Linux: `/tmp`） |
| 会话子目录 | `<系统临时目录>/kdocs_aippt_<conversation_id>/` |
| 命名示例 | `/tmp/kdocs_aippt_a1b2c3d4-e5f6-7890-abcd-ef1234567890/` |

### 中间文件约定

#### 主题生成 PPT

| 步骤 | 推荐文件路径 |
|------|-------------|
| Step 01 用户选择 | `$AIPPT_WORK_DIR/01_selections.json` |
| Step 02 研究结果 | `$AIPPT_WORK_DIR/02_research.json` |
| Step 03 大纲结果 | `$AIPPT_WORK_DIR/03_outline.json` |
| Step 04 转换结果 | `$AIPPT_WORK_DIR/04_config.json` |
| Step 05 PPT 结果 | `$AIPPT_WORK_DIR/05_ppt_result.json` |
| Step 06 云文档结果 | `$AIPPT_WORK_DIR/06_cloud_result.json` |

#### 文档生成 PPT

| 步骤 | 推荐文件路径 |
|------|-------------|
| Step 01 会话信息 | `$AIPPT_WORK_DIR/01_session.json` |
| Step 02 选项确认（含 checkpoint_id, interrupt_id） | `$AIPPT_WORK_DIR/02_options.json` |
| Step 03 大纲结果 | `$AIPPT_WORK_DIR/03_outline.md` + `$AIPPT_WORK_DIR/03_extra.json` |
| Step 04 美化结果 | `$AIPPT_WORK_DIR/04_beautify.json` |
| Step 05 本地格式转换结果 | `$AIPPT_WORK_DIR/05_config.json`（`{topic, outlines[]}` 结构） |
| Step 06 PPT 结果 | `$AIPPT_WORK_DIR/06_ppt_result.json` |
| Step 07 云文档结果 | `$AIPPT_WORK_DIR/07_cloud_result.json` |

### 清理方式

流程结束时递归删除整个会话目录：

```javascript
const fs = require('fs');
fs.rmSync(sessionDir, { recursive: true, force: true });
```

```python
import shutil
shutil.rmtree(session_dir, ignore_errors=True)
```

```bash
# Linux / macOS
rm -rf "${TMPDIR}/aippt_${SESSION_ID}"

# Windows PowerShell
Remove-Item -Recurse -Force "$env:TEMP\aippt_${SESSION_ID}"
```

> 清理操作应放在 `try...finally` 中执行，无论流程成功或失败都必须尝试清理。

### JSON 格式约束

中间 JSON 文件必须通过编程语言的标准 JSON 序列化函数生成（**禁止手动拼接**）：

```javascript
// ✅ 正确
fs.writeFileSync(filePath, JSON.stringify(data, null, 2), 'utf-8');
// 写入后验证
JSON.parse(fs.readFileSync(filePath, 'utf-8'));
```

```python
# ✅ 正确
import json
with open(file_path, 'w', encoding='utf-8') as f:
    json.dump(data, f, ensure_ascii=False, indent=2)
```

编码要求：UTF-8 无 BOM，不要在 JSON 前后添加非 JSON 内容。

---

---

## 一、主题生成 PPT

> 从一句话主题到最终 PPT 的链路（问卷 → 深度研究 → 大纲 → 生成）

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| [`aippt.theme_questions`](aippt/theme_ppt.md) | 根据主题描述生成补充问卷 | `input` |
| [`aippt.theme_deep_research`](aippt/theme_ppt.md) | 围绕主题与问答执行联网深度研究 | `input`, `question_and_answers` |
| [`aippt.theme_outline`](aippt/theme_ppt.md) | 根据主题、问答和资料生成 PPT 大纲 | `input`, `question_and_answers` |
| [`aippt.theme_generate_html_pptx`](aippt/theme_ppt.md) | 根据主题与大纲生成 HTML PPTX | `topic`, `outlines` |

## 二、文档生成 PPT

> 从现有文档到 PPT 的链路（会话 → 大纲 → 美化 → 生成）

| 工具 | 功能 | 必填参数 |
|------|------|----------|
| [`aippt.doc_create_session`](aippt/doc_ppt.md) | 创建文档转 PPT 的 AI 会话 |  |
| [`aippt.doc_outline_options`](aippt/doc_ppt.md) | 解析文档并返回大纲选项问题 | `session_id`, `input` |
| [`aippt.doc_outline`](aippt/doc_ppt.md) | 生成完整 Markdown 大纲 | `session_id`, `checkpoint_id`, `input`, `resume_info` |
| [`aippt.doc_beautify`](aippt/doc_ppt.md) | 为 Markdown 大纲生成整套风格描述 | `topic`, `outline`, `user_intention` |
| [`aippt.doc_generate_ppt`](aippt/doc_ppt.md) | 根据主题与大纲生成最终 PPT | `topic`, `outlines` |
