# AI PPT（aippt）工具完整参考文档

本文件包含金山文档 Skill 中 **AI PPT** 相关工具的完整 API 说明、详细调用示例、参数说明和返回值说明。

**适用范围**：本文档中的 `aippt.`* 工具面向 AI 演示生成链路，包括需求补充问卷、联网深度研究、大纲生成，以及根据大纲生成 HTML PPTX。

---

## 通用说明

### 推荐调用链路

多数场景建议按以下顺序调用：

1. `aippt.questions`：当主题描述较宽泛时，先生成补充问卷
2. `aippt.deep_research`：结合问答执行联网研究，补充事实资料和案例
3. `aippt.outline`：根据主题、问答和研究资料生成结构化大纲
4. `aippt.generate_html_pptx`：根据最终 `outlines` 生成可下载的演示结果

### 使用建议

| 场景 | 建议 |
|------|------|
| 用户只给一句主题 | 先 `aippt.questions`，补齐受众、场景、风格、重点 |
| 需要更可靠的事实资料 | 衔接 `aippt.deep_research` |
| 先看结构再决定生成 | 先停在 `aippt.outline` |
| 需要最终演示文件 | 使用 `aippt.generate_html_pptx` |

### 注意事项

- `aippt.deep_research` 的完整研究内容主要通过流式通知产生，最终 tool result 只保留摘要
- `aippt.outline` 返回的 `outline` 结构可能因上游模板而变化，传给 `aippt.generate_html_pptx` 时应优先使用其中正式的 `outlines` 数组
- `aippt.generate_html_pptx` 返回的下载链接通常带时效性，建议尽快消费
- 通过 `mcporter` 调用本流程中的步骤时，请为每一步单独设置超时时间为 **3000 秒**
- `deep_research` 的 `references`、`outline` 结果、格式转换后的 `outlines` 等内容可能很长，**必须先写入本地文件**，后续步骤再读取文件内容，避免命令行参数超过长度限制

### MCP 调用规则（非常重要）

在通过 `mcporter` 调用 aippt 工具时，长参数应该采取读文件的方式解决。

当参数 JSON 超过 2000 字符，或包含 `references`、`outlines`、`content_base64` 等长字段时，**必须使用脚本方式调用**。脚本模板和详细规则见下文「MCP 长参数脚本调用」。

短参数场景（如 `aippt.questions` 只有一个 `input` 字段）可直接命令行调用：

```bash
mcporter call kdocs aippt.questions --args '{"input":"长颈鹿科普PPT"}' --output json --timeout 3000000
```

### MCP 长参数脚本调用

当通过 `mcporter` 调用工具时，若参数 JSON 超过 2000 字符（如含 `references`、`outlines`、`question_and_answers`、`content_base64` 等长字段），**必须生成辅助脚本**，将参数写入文件后通过脚本调用，避免命令行长度超限。

涉及步骤：Step 03（大纲生成）、Step 04/05（生成 PPTX）、Step 06（上传文件）等。短参数场景（如 `aippt.questions` 只有一个 `input` 字段）可直接命令行调用。

#### 脚本生成与调用流程

1. 将工具参数写入 JSON 文件（UTF-8），如 `$AIPPT_WORK_DIR/03_outline_args.json`
2. 在会话临时目录生成调用脚本 `$AIPPT_WORK_DIR/_call_mcp.js`（首次生成后复用）
3. 执行：`node $AIPPT_WORK_DIR/_call_mcp.js <toolName> <argsFile> [outputFile] [timeoutMs] [serverName]`

#### 脚本模板

生成的 `_call_mcp.js` 必须包含以下核心逻辑：

```javascript
const { spawnSync, execSync } = require('child_process');
const fs = require('fs');
const path = require('path');
const os = require('os');

const toolName = process.argv[2];
const argsFile = process.argv[3];
const outputFile = process.argv[4] || '';
const timeoutMs = parseInt(process.argv[5] || '300000', 10);
const serverName = process.argv[6] || 'kdocs';
const STDOUT_LIMIT = 8000;
const TIMEOUT_BUFFER = 30000;

if (!toolName || !argsFile) {
  console.error('用法: node _call_mcp.js <toolName> <argsFile> [outputFile] [timeoutMs] [serverName]');
  process.exit(1);
}

// 自动探测 mcporter cli.js 路径
function findMcporterCli() {
  const candidates = [];
  if (process.env.MCPORTER_CLI) candidates.push(process.env.MCPORTER_CLI);
  const nodeDir = path.dirname(process.execPath);
  candidates.push(path.join(nodeDir, 'node_modules', 'mcporter', 'dist', 'cli.js'));
  // nvm 场景 (Windows)
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

#### 关键设计要点

| 要点 | 说明 |
| --- | --- |
| 参数从文件读取 | 绕过 OS 命令行长度限制（Windows ~8191 字符） |
| `spawnSync` + `process.execPath` | 直接运行 `cli.js`，避免 PATH 查找失败 |
| 自动探测 mcporter 路径 | 按优先级搜索：环境变量 → Node 同级 → nvm → npm root -g → 默认路径 |
| `maxBuffer: 50MB` | 防止大输出被截断 |
| 超时缓冲 +30 秒 | mcporter 内部超时与外层进程超时之间留有余量 |
| stdout 截断 8000 字符 | 避免终端刷屏，完整内容写入输出文件 |
| 过滤 ExperimentalWarning | 清除 Node.js 实验性功能噪声日志 |
| 兜底 PATH 调用 | cli.js 未找到时，退化为通过 `mcporter` 命令名调用 |

#### 调用示例

```bash
# 调用 aippt.outline（参数含长 references）
node $AIPPT_WORK_DIR/_call_mcp.js aippt.outline $AIPPT_WORK_DIR/03_outline_args.json $AIPPT_WORK_DIR/03_outline.json 3000000

# 调用 aippt.generate_html_pptx（参数含长 outlines）
node $AIPPT_WORK_DIR/_call_mcp.js aippt.generate_html_pptx $AIPPT_WORK_DIR/04_config.json $AIPPT_WORK_DIR/05_ppt_result.json 2000000
```

### 临时文件管理

所有中间产物（问卷、研究结果、大纲、转换配置,临时脚本等）必须格式规范，使用UTF-8格式，方便上下文读取。必须写入基于当前会话 `session_id` 的独立临时目录，以避免污染工作区或在多会话并行时互相干扰数据。

#### 目录规则

| 项目 | 说明 |
|------|------|
| 根目录 | 系统临时目录，即 `os.tmpdir()`（Windows: `%TEMP%`，macOS/Linux: `/tmp`） |
| 会话子目录 | `<系统临时目录>/aippt_<session_id>/` |
| 命名示例 | `/tmp/aippt_a1b2c3d4-e5f6-7890-abcd-ef1234567890/` |

> `session_id` 应取自当前会话上下文中的唯一标识（如 MCP session ID、conversation ID 等），确保每个会话拥有独立的文件空间，多个会话并行运行时互不干扰。

#### 生命周期与清理

1. **创建时机**：首次需要写入中间文件时，按需创建会话临时目录（使用 `recursive: true` / `exist_ok=True` 确保幂等）
2. **使用期间**：所有中间产物均写入该目录下，路径示例：
   - `<tmpdir>/aippt_<session_id>/01_selections.json`
   - `<tmpdir>/aippt_<session_id>/02_research.json`
   - `<tmpdir>/aippt_<session_id>/03_outline.json`
   - `<tmpdir>/aippt_<session_id>/04_config.json`
   - `<tmpdir>/aippt_<session_id>/05_ppt_result.json`
   - `<tmpdir>/aippt_<session_id>/06_cloud_result.json`
3. **清理时机**：
   - **正常结束**：流程最终步骤（Step 06 或用户确认最终结果）完成后，**必须主动删除**整个会话临时目录
   - **异常中断**：在 `finally` / 错误处理逻辑中同样尝试清理，确保即使流程失败也不残留文件
   - **兜底机制**：目录创建在系统临时目录下，即使因进程崩溃未能主动清理，操作系统的 temp 目录自动回收策略也会最终清除

#### 清理方式

推荐在流程结束时递归删除整个会话目录：

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

> **重要**：清理操作应放在 `try...finally` 中执行，无论流程成功或失败都必须尝试清理，做到不残留任何临时文件。

## 标准生成流程

### Step 01: 获取问卷并收集用户选择

调用 `aippt.questions` 获取问卷后，**必须向用户展示问卷并收集回答**，然后将结果整理为 `question_and_answers` 并写入 `01_selections.json`。

> 通过 `mcporter` 调用本步骤时，请单独设置超时时间为 **3000 秒**。

#### 1.1 获取问卷

调用 `aippt.questions`，输入主题描述，拿到问卷题目列表。题型说明：

| type | 含义 | 用户交互方式 |
|------|------|--------------|
| `radio` | 单选题 | 向用户展示选项，用户单选 |
| `checkbox` | 多选题 | 向用户展示选项，用户多选 |
| `input` | 文本题 | 向用户展示问题，用户文本输入或采用默认值 |

#### 1.2 向用户展示问卷并收集回答

拿到问卷后，**不要跳过用户交互**，必须：

1. 将每道题目以友好的方式展示给用户（展示题目标题、选项文案）
2. 等待用户逐题回答或一次性回答
3. 若用户明确表示"使用默认"或未对某题作答，则回退使用该题的 `default_answer`

#### 1.3 整理用户选择并落盘

将用户回答整理为 `question_and_answers` 数组：

- `question`：使用原问卷项的 `title`
- `answer`：优先使用用户实际选择；若用户未回答，则回退到 `default_answer`

整理完成后写入会话临时目录 `<tmpdir>/aippt_<session_id>/01_selections.json`，后续步骤从该文件读取。

> **注意**：本步骤不再单独保存问卷原始结果（`01_questions.json`），只需保存最终的用户选择文件 `01_selections.json`。

### Step 02: 联网研究

调用 `aippt.deep_research`。注意：

> 通过 `mcporter` 调用本步骤时，请单独设置超时时间为 **3000 秒**。

- 最终 tool result 只会返回完成摘要
- 真正要传给下一步的 `references`，应从流式通知中提取完整研究资料后自行拼接
- 建议把完整研究结果落盘为 `<tmpdir>/aippt_<session_id>/02_research.json`
- 后续 `aippt.outline` 所需的 `references`，请从该文件读取，不要把超长文本直接拼进命令行

### Step 03: 大纲生成

调用 `aippt.outline(input, question_and_answers, references)`，返回结构化 `outline` 结果。

> 通过 `mcporter` 调用本步骤时，请单独设置超时时间为 **3000 秒**。
> 建议把大纲结果写入会话临时目录 `<tmpdir>/aippt_<session_id>/03_outline.json`，后续格式转换直接读取该文件内容。

### Step 04: 本地格式转换

`aippt.generate_html_pptx` 需要的是标准化后的 `{ topic, outlines[] }`，因此需要在本地做一次转换。

> 本步骤为本地转换动作；若前后步骤通过 `mcporter` 调用，仍应分别单独设置超时时间为 **3000 秒**。
> 建议把转换结果写入会话临时目录 `<tmpdir>/aippt_<session_id>/04_config.json`，下一步直接从文件读取 `topic` 与 `outlines`。

#### outlines 每项必填字段

每个 outline 项必须**同时包含以下 4 个字段**，缺少任何一个都可能导致 `400 badRequest`：

| 字段 | 类型 | 说明 |
|------|------|------|
| `page_type` | string | 按下方映射表转换后的页面类型 |
| `title` | string | 页面标题 |
| `contents` | string | 页面正文内容，多行用 `\n` 分隔；封面/章节页/结尾页可传空字符串 `""` |
| `design_style` | string | 该页的视觉风格描述（见下方提取规则） |

#### page_type 映射表

| 大纲原始 type | 转换后 page_type |
|---------------|------------------|
| `title` / `cover` | `pt_title` |
| `toc` | `pt_contents` |
| `chapter` | `pt_section_title` |
| `text` / `content` / `section` | `pt_text` |
| `end` / `ending` | `pt_end` |

#### design_style 提取规则

`design_style` 是每页必填的风格描述，应按以下优先级获取：

1. **优先从 `aippt.outline` 流式输出中提取**：若流式通知中每页携带了 `design_style` 字段，直接使用该值
2. **从大纲全局风格派生**：若流式输出仅返回全局 `design_style`，将其拼接到每页，如果有单页携带了，则不使用全局风格。
3. **根据页面内容生成**：若无法获取流式数据（如 CLI 调用），根据问卷中的风格偏好和每页内容特点，为每页独立生成差异化的风格描述，避免所有页面使用同一句话

#### 转换示例

```json
{
  "topic": "长颈鹿——地球上最高的动物",
  "outlines": [
    {
      "title": "长颈鹿——地球上最高的动物",
      "page_type": "pt_title",
      "contents": "",
      "design_style": "自然生态主题封面，非洲草原背景，暖色调"
    },
    {
      "title": "长颈鹿的基本信息",
      "page_type": "pt_text",
      "contents": "现存最高的陆生动物，身高可达5.8米\n主要分布在非洲稀树草原",
      "design_style": "知识卡片风格，清晰的数据展示"
    },
    {
      "title": "感谢观看",
      "page_type": "pt_end",
      "contents": "",
      "design_style": "温馨收尾，夕阳剪影氛围"
    }
  ]
}
```

### Step 05: 生成 PPTX

调用 `aippt.generate_html_pptx(topic, outlines)`，得到：

> 通过 `mcporter` 调用本步骤时，请单独设置超时时间为 **2000 秒**。
> 生成成功后也需要展示阶段成果连接给用户。
> 建议把返回结果写入会话临时目录 `<tmpdir>/aippt_<session_id>/05_ppt_result.json`，后续上传步骤从文件中读取 `merged_url`。

- `merged_url`：合并后的完整 PPTX 下载地址
- `pages[]`：逐页生成结果

### Step 06: 上传为云文档

目标是"给用户一个云端可继续编辑的演示文稿"，需要完成以下子步骤。

> 说明：`upload_file` 的新建模式支持 `.pptx`，因此可作为该流程的最终落盘步骤。

#### 6.1 获取 `drive_id`

`upload_file` 需要 `drive_id` 和 `parent_id` 两个必填参数。`parent_id` 固定为 `"0"`（根目录），但 `drive_id` 必须先通过查询获取：

```bash
mcporter call kdocs search_files --args '{"keyword":"文档","type":"file_name","page_size":1}' --output json --timeout 300000
```

从返回结果中提取 `data.data.items[0].file.drive_id`。

> 也可以用 `list_files`（需同时传 `drive_id` 和 `parent_id`），但首次获取 `drive_id` 时推荐用 `search_files`，因为它只需要关键词即可返回带 `drive_id` 的文件信息。

#### 6.2 下载 PPTX 并转 Base64

从 Step 05 的 `05_ppt_result.json` 中读取 `merged_url`，下载 PPTX 二进制文件并转为 Base64 编码，写入上传参数文件。

建议生成 `$AIPPT_WORK_DIR/_download_pptx.js` 脚本完成此步骤：

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

#### 6.3 上传文件（必须使用编程 API）

> **重要：`upload_file` 无法通过 `mcporter call` 命令行调用。**
>
> 原因：PPTX 文件转 Base64 后 `content_base64` 字段通常有几十万字符。`mcporter call --args <json>` 会将整个 JSON 作为命令行参数传递，而操作系统对命令行长度有严格限制（Windows CreateProcess ~32,767 字符，cmd ~8,191 字符）。同时，如果脚本用 `spawnSync` 绕过 shell，参数仍作为进程参数传递，同样会触发 `ENAMETOOLONG` 错误。`mcporter` 目前不支持 `--args-stdin` 或 `--args-file`。
>
> **唯一可行方式**：在 Node.js 脚本中直接调用 mcporter 的编程 API `callOnce()`，参数在进程内存中传递，不经过命令行。

生成 `$AIPPT_WORK_DIR/_upload_cloud.js` 脚本：

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

`callOnce()` 函数签名：

```javascript
await mcporter.callOnce({
  server: 'kdocs',         // 服务名
  toolName: 'upload_file', // 工具名
  args: { ... }            // 参数对象（直接传入，不经命令行）
});
```

返回结果中的 `content[0].text` 包含 JSON 字符串，解析后可获取 `file_id`。

#### 6.4 获取云文档链接

上传成功后，从返回结果中提取 `file_id`，调用 `get_file_link` 获取在线访问链接：

```bash
mcporter call kdocs get_file_link --args '{"file_id":"<上传返回的file_id>"}' --output json --timeout 300000
```

返回结果中的 `link_url` 即为最终云文档链接，展示给用户。

#### 各调用方式适用范围总结

| 工具 / 步骤 | `mcporter call` 命令行 | `_call_mcp.js` 脚本 | `callOnce()` 编程 API |
| --- | --- | --- | --- |
| `aippt.questions` | 可以（参数短） | 可以 | 可以 |
| `aippt.deep_research` | 可以（参数短） | 可以 | 可以 |
| `aippt.outline` | **不推荐**（`references` 可能很长） | **推荐** | 可以 |
| `aippt.generate_html_pptx` | **不推荐**（`outlines` 可能很长） | **推荐** | 可以 |
| `upload_file`（含 `content_base64`） | **不可用**（超出 OS 命令行限制） | **不可用**（`spawnSync` 同样超限） | **必须使用** |
| `get_file_link` / `search_files` 等 | 可以（参数短） | 可以 | 可以 |


## 中间文件约定

为避免 `mcporter` 命令行参数过长，推荐将每一步中间结果固定写入**会话临时目录**中的本地文件，并在下一步**读取文件内容**后再构造请求参数。文件应该是用UTF-8编码

> 会话临时目录路径：`<系统临时目录>/aippt_<session_id>/`，详见「通用说明 — 临时文件管理」。

| 步骤 | 推荐文件路径 |
|------|-------------|
| Step 01 用户选择 | `<tmpdir>/aippt_<session_id>/01_selections.json` |
| Step 02 研究结果 | `<tmpdir>/aippt_<session_id>/02_research.json` |
| Step 03 大纲结果 | `<tmpdir>/aippt_<session_id>/03_outline.json` |
| Step 04 转换结果 | `<tmpdir>/aippt_<session_id>/04_config.json` |
| Step 05 PPT 结果 | `<tmpdir>/aippt_<session_id>/05_ppt_result.json` |
| Step 06 云文档结果 | `<tmpdir>/aippt_<session_id>/06_cloud_result.json` |

原则：

- **禁止写入 workspace**：所有中间文件必须写入会话临时目录，避免污染工作区
- **会话隔离**：目录名包含 `session_id`，多会话并行运行时互不干扰
- 长文本与长 JSON 一律先落盘，再读取文件内容参与下一步
- 不要把 `references`、`outline`、`outlines` 这类长字段直接内联到 `mcporter call ...` 命令行里
- 若需要脚本拼接参数，应在脚本内读取本地文件后构造请求，而不是手工复制粘贴
- **流程结束后必须清理**：无论成功或失败，都应递归删除整个会话临时目录，不残留任何文件

### JSON 格式严格约束

中间 JSON 文件是步骤间传递数据的核心载体，任何格式错误都会导致解析失败、流程中断。**写入 JSON 文件时必须遵守以下规则：**

#### 禁止出现的字符与写法

| 错误类型 | 错误示例 | 正确写法 | 说明 |
|----------|----------|----------|------|
| 中文/智能引号 | `"title"`, `'title'`, `"value"`, `'value'` | `"title"`, `"value"` | JSON 只允许 ASCII 直引号 `"` (U+0022) |
| 中文冒号 | `"key"："value"` | `"key": "value"` | 键值分隔符只能用 ASCII 冒号 `:` (U+003A) |
| 中文逗号 | `"a": 1，"b": 2` | `"a": 1, "b": 2` | 分隔符只能用 ASCII 逗号 `,` (U+002C) |
| 中文括号 | `【"item"】`、`（"a"）` | `["item"]`、`("a")` | 只能用 ASCII 方括号和圆括号 |
| 尾部逗号 | `{"a": 1, "b": 2,}` | `{"a": 1, "b": 2}` | 最后一个元素后**禁止**逗号 |
| 单引号包裹 | `{'key': 'value'}` | `{"key": "value"}` | JSON 标准只认双引号 |
| 注释 | `{"a": 1 // comment}` | `{"a": 1}` | JSON **不支持**任何注释 |
| 未转义换行 | `"line1↵line2"` (真实换行) | `"line1\nline2"` | 字符串内换行必须用 `\n` 转义 |
| 未转义反斜杠 | `"path: C:\Users"` | `"path: C:\\Users"` | 反斜杠必须写成 `\\` |
| 未转义双引号 | `"he said "hello""` | `"he said \"hello\""` | 值内双引号必须用 `\"` 转义 |
| 未转义制表符 | `"col1	col2"` (真实 Tab) | `"col1\tcol2"` | 制表符必须用 `\t` 转义 |
| 非法值 | `undefined`, `NaN`, `Infinity` | `null`, `0`, `null` | 使用 JSON 合法值替代 |
| BOM 头 | 文件以 `\xEF\xBB\xBF` 开头 | 无 BOM 的 UTF-8 | 写文件时不要带 BOM |

#### 必须使用 `JSON.stringify()` 序列化

**绝对禁止**手动拼接 JSON 字符串。所有中间文件的 JSON 内容必须通过编程语言的标准 JSON 序列化函数生成：

```javascript
// ✅ 正确：使用 JSON.stringify
const data = { title: "长颈鹿——地球上最高的动物", contents: "身高可达5.8米\n主要分布在非洲" };
fs.writeFileSync(filePath, JSON.stringify(data, null, 2), 'utf-8');

// ❌ 错误：手动拼接 JSON 字符串
fs.writeFileSync(filePath, '{"title": "' + title + '", "contents": "' + contents + '"}', 'utf-8');
```

```python
# ✅ 正确：使用 json.dumps
import json
data = {"title": "长颈鹿——地球上最高的动物", "contents": "身高可达5.8米\n主要分布在非洲"}
with open(file_path, 'w', encoding='utf-8') as f:
    json.dump(data, f, ensure_ascii=False, indent=2)

# ❌ 错误：用 f-string 或字符串拼接构造 JSON
with open(file_path, 'w') as f:
    f.write(f'{{"title": "{title}", "contents": "{contents}"}}')
```

#### 写入后必须验证

每次写入 JSON 文件后，**必须立即读取并解析验证**，确认写入的内容是合法 JSON：

```javascript
// 写入后验证
fs.writeFileSync(filePath, JSON.stringify(data, null, 2), 'utf-8');
try {
  JSON.parse(fs.readFileSync(filePath, 'utf-8'));
} catch (e) {
  console.error('[ERROR] JSON 写入验证失败:', filePath, e.message);
  process.exit(1);
}
```

```python
import json
with open(file_path, 'w', encoding='utf-8') as f:
    json.dump(data, f, ensure_ascii=False, indent=2)
# 写入后验证
with open(file_path, 'r', encoding='utf-8') as f:
    json.load(f)  # 解析失败会抛 JSONDecodeError
```

#### 编码要求

- 文件编码：**UTF-8 无 BOM**
- `ensure_ascii=False`（Python）或默认行为（Node.js）以保留中文可读性
- 不要在 JSON 文本前后添加任何非 JSON 内容（如 markdown 代码围栏、说明文字等）

---

## 一、AI PPT 生成链路

### 1. aippt.questions

#### 功能说明

根据用户输入的 PPT 主题或需求描述，生成一组可继续追问用户偏好的问卷题目。

**适用于**：用户只给了一个较宽泛的主题，还需要澄清受众、场景、风格、内容重点等信息时。

- 服务端会固定以 `stream=true` 调用上游接口，但最终工具返回为一次性 JSON 结果
- 建议将返回的问卷继续转成 `question_and_answers`，供 `aippt.deep_research` 与 `aippt.outline` 使用


#### 调用示例

生成儿童科普主题问卷：

```json
{
  "input": "帮我做一份长颈鹿主题的儿童科普 PPT，用于小学课堂讲解"
}
```


#### 参数说明

- `input` (string, 必填): 用户输入的 PPT 主题、目标或需求描述

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "input": "帮我做一份长颈鹿主题的儿童科普 PPT",
    "questionnaire": [
      {
        "key": "properties1",
        "title": "这份长颈鹿主题的PPT主要用于什么场景？",
        "type": "radio",
        "enum": ["A", "B", "C", "D"],
        "enumNames": ["儿童科普教学", "幼儿园活动展示", "生日派对主题分享", "文创产品提案"],
        "required": false,
        "default_answer": ["儿童科普教学"]
      },
      {
        "key": "properties4",
        "title": "如果目标受众是低龄儿童，您是否希望在PPT大纲中规划互动环节模块？",
        "type": "input",
        "required": false,
        "default_answer": "是"
      }
    ]
  }
}

```

| 字段 | 类型 | 说明 |
|------|------|------|
| `code` | number | 0 表示成功 |
| `message` | string | 成功时通常为“成功” |
| `data.input` | string | 原始主题描述 |
| `data.questionnaire` | array | 生成的问卷列表 |
| `data.questionnaire[].key` | string | 问题唯一键 |
| `data.questionnaire[].title` | string | 问题标题 |
| `data.questionnaire[].type` | string | 题型，如 `radio` / `checkbox` / `input` |
| `data.questionnaire[].enum` | array[string] | 选项编码列表，题型为选择题时可能返回 |
| `data.questionnaire[].enumNames` | array[string] | 选项文案列表，顺序与 `enum` 对齐 |
| `data.questionnaire[].required` | boolean | 是否必答 |
| `data.questionnaire[].default_answer` | any | 默认答案，可能为字符串或字符串数组 |


> 如果主题已经足够明确、无需继续澄清，可跳过本工具，直接组织 `question_and_answers`
---

### 2. aippt.deep_research

#### 功能说明

根据用户主题和补充问答执行 AI PPT 联网研究，提炼研究目标完成情况与研究资料。

**适用于**：需要先补充事实资料、案例、背景知识，再继续生成 PPT 大纲时。

- 服务端固定以 `stream=true` 调用上游接口
- 最终工具结果只返回完成摘要，不直接返回完整研究资料文本
- 若调用端传入 MCP `_meta.progressToken`，服务会通过 `notifications/progress` 回传进度
- 若同时传入 `_meta.partialResults=true`，`notifications/progress` 中会追加逐段 partial result chunk
- 服务端还会兼容透传 `notifications/aippt/deep_research` 结构化事件


#### 调用示例

对儿童科普主题做研究：

```json
{
  "input": "帮我做一份长颈鹿主题的儿童科普 PPT",
  "question_and_answers": [
    {
      "question": "这份 PPT 主要用于什么场景？",
      "answer": "小学科学课课堂讲解"
    },
    {
      "question": "希望突出哪些内容模块？",
      "answer": "外形特征、生活习性、趣味冷知识"
    }
  ]
}
```


#### 参数说明

- `input` (string, 必填): 用户输入的 PPT 主题或需求描述
- `question_and_answers` (array[object], 必填): 问答列表，不能为空。每项含 `question`（string，必填）与 `answer`（string，必填）。


#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "streamed": true,
    "done": true,
    "goal_count": 3,
    "goals_done": 3,
    "learning_count": 6
  }
}

```

| 字段 | 类型 | 说明 |
|------|------|------|
| `code` | number | 0 表示成功 |
| `message` | string | 成功时通常为“成功” |
| `data.streamed` | boolean | 固定为 true，表示研究过程通过流式事件执行 |
| `data.done` | boolean | 是否已完成 |
| `data.goal_count` | integer | 研究目标总数 |
| `data.goals_done` | integer | 已完成的研究目标数 |
| `data.learning_count` | integer | 已提取的研究资料条数 |


> 若后续要调用 `aippt.outline`，应从流式通知中消费研究资料文本，并将其作为 `references` 传入
> 最终 tool result 不包含完整 `references`，不要仅依赖最终返回体获取研究内容
---

### 3. aippt.outline

#### 功能说明

根据主题描述、补充问答以及研究资料，生成用于 AI PPT 的结构化大纲结果。

**适用于**：已经明确主题方向，并且希望先拿到可审阅的大纲，再决定是否继续生成 HTML PPTX。

- 服务端会固定以 `stream=true` 调用上游 `image_to_slide/outline` 接口
- `references` 建议直接传入 `aippt.deep_research` 流式阶段产出的研究资料文本
- `question_and_answers` 不能为空，否则服务端会直接报参数错误


#### 调用示例

生成儿童科普主题大纲：

```json
{
  "input": "帮我做一份长颈鹿主题的儿童科普 PPT",
  "question_and_answers": [
    {
      "question": "这份 PPT 主要用于什么场景？",
      "answer": "小学科学课课堂讲解"
    },
    {
      "question": "希望整体风格更偏向哪种？",
      "answer": "插画风、适合儿童"
    }
  ],
  "references": "长颈鹿是现存最高的陆生动物，舌头很长，常以树叶为食。"
}
```


#### 参数说明

- `input` (string, 必填): 用户输入的 PPT 主题或需求描述
- `question_and_answers` (array[object], 必填): 问答列表，不能为空。每项含 `question`（string，必填）与 `answer`（string，必填）。

- `references` (string, 可选): 研究资料文本，建议直接传入 `aippt.deep_research` 产出的研究内容

#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "input": "帮我做一份长颈鹿主题的儿童科普 PPT",
    "question_and_answers": [
      {
        "question": "这份 PPT 主要用于什么场景？",
        "answer": "小学科学课课堂讲解"
      }
    ],
    "references": "长颈鹿是现存最高的陆生动物，主要生活在非洲稀树草原。",
    "outline": {
      "type": "outline",
      "title": "长颈鹿主题演示",
      "slides": [
        { "title": "封面" },
        { "title": "习性介绍" }
      ]
    }
  }
}

```

| 字段 | 类型 | 说明 |
|------|------|------|
| `code` | number | 0 表示成功 |
| `message` | string | 成功时通常为“成功” |
| `data.input` | string | 原始主题描述 |
| `data.question_and_answers` | array | 原样回传的问答列表 |
| `data.references` | string | 原样回传的研究资料文本 |
| `data.outline` | object | 上游返回的结构化大纲对象；字段会随模板与场景变化 |


> outline 的具体内部结构由上游模型决定，后续若要生成 HTML PPTX，建议优先从其中提取正式的 outlines 数组再传给 aippt.generate_html_pptx
---

### 4. aippt.generate_html_pptx

#### 功能说明

根据主题与 `outlines` 调用 structppt 接口，生成逐页 HTML PPTX，并返回合并后的演示文件地址。

**适用于**：已经拿到稳定的大纲结构，希望进一步生成可下载的演示文稿结果。

- 服务端固定使用 `json2ppt-banana` 场景
- `outlines` 只做“非空对象数组”校验，推荐直接传入上游生成的标准大纲数组
- 返回值同时包含逐页结果和合并后的完整 PPTX 链接


#### 调用示例

生成儿童科普主题 HTML PPTX：

```json
{
  "topic": "长颈鹿主题演示",
  "outlines": [
    {
      "title": "封面",
      "type": "cover"
    },
    {
      "title": "长颈鹿的生活习性",
      "type": "content"
    },
    {
      "title": "探索永无止境",
      "type": "ending"
    }
  ]
}
```


#### 参数说明

- `topic` (string, 必填): 演示文稿标题
- `outlines` (array[object], 必填): PPT 大纲数组，建议直接传入标准 outlines 结果。每项为 object，至少需为非空对象。


#### 返回值说明

```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "topic": "长颈鹿主题演示",
    "merged_url": "https://meihua-service.ks3-cn-beijing.ksyuncs.com/file/20260401/pptx/b74af503-ef35-445f-8106-2c1deac0dcb3.pptx?Expires=1775027041&AWSAccessKeyId=xxx&Signature=xxx",
    "total_pages": 4,
    "pages": [
      {
        "slide_index": 3,
        "file_url": "https://meihua-service.ks3-cn-beijing.ksyuncs.com/temp/20260401/pptx/3097fecf-bf39-454a-92eb-700b49e0e773.pptx?Expires=1775199540&AWSAccessKeyId=xxx&Signature=xxx",
        "title": "探索永无止境",
        "type": "ending"
      }
    ]
  }
}

```

| 字段 | 类型 | 说明 |
|------|------|------|
| `code` | number | 0 表示成功 |
| `message` | string | 成功时通常为“成功” |
| `data.topic` | string | 演示文稿标题 |
| `data.merged_url` | string | 合并后的完整 PPTX 下载地址 |
| `data.total_pages` | integer | 总页数 |
| `data.pages` | array | 逐页生成结果 |
| `data.pages[].slide_index` | integer | 幻灯片序号 |
| `data.pages[].file_url` | string | 单页 PPTX 文件地址 |
| `data.pages[].title` | string | 对应大纲标题 |
| `data.pages[].type` | string | 对应大纲类型，如 `cover` / `content` / `ending` |


> 返回的 `merged_url` 和 `pages[].file_url` 一般为临时链接，调用后应尽快消费
---


## 工具速查表

| # | 工具名 | 分类 | 功能 | 必填参数 |
|---|--------|------|------|----------|
| 1 | `aippt.questions` | AI PPT 生成链路 | 根据主题描述生成补充问卷 | `input` |
| 2 | `aippt.deep_research` | AI PPT 生成链路 | 围绕主题与问答执行联网深度研究 | `input`, `question_and_answers` |
| 3 | `aippt.outline` | AI PPT 生成链路 | 根据主题、问答和资料生成 PPT 大纲 | `input`, `question_and_answers` |
| 4 | `aippt.generate_html_pptx` | AI PPT 生成链路 | 根据主题与大纲生成 HTML PPTX | `topic`, `outlines` |
