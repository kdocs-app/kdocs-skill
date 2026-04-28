# 版本自检

> 何时触发：**首次使用** Skill / **距上次自检 >24h** / **收到 `unknown command` 或不兼容错误**。其他时刻无需重复执行。

## 前置检查：确认 CLI 已安装且版本满足要求

运行 `kdocs-cli version`：

- 若命令不存在 → 先按 `SKILL.md` 的「工具安装」章节运行安装脚本
- 若输出的版本号 **低于** `SKILL.md` 头部 frontmatter 的 `version` → 运行 `kdocs-cli upgrade -y` 升级 CLI
- 若版本 **满足** 要求 → 继续后续步骤

## 第一步：检查远端最新版本

```bash
kdocs-cli upgrade --check
```

返回当前 CLI 版本和远端最新版本。若 CLI 有新版本可用：

```bash
kdocs-cli upgrade              # 交互式升级
kdocs-cli upgrade -y           # 跳过确认直接升级
```

升级过程自动备份旧版本到 `~/.kdocs-cli/backup/`，支持 `kdocs-cli upgrade --rollback` 回滚。

## 第二步：检查 Skill 版本是否匹配

从 `SKILL.md` 头部 frontmatter 的 `version` 字段读取 Skill 版本，与 `kdocs-cli version` 输出的 CLI 版本对比：

- **相等** → 版本同步，自检完成
- **不相等** → Skill 需要更新，继续第三步

## 第三步：获取最新 Skill 包

```bash
kdocs-cli call check_skill_update version=<当前Skill版本号>
```

返回 JSON，关键字段：

| 字段 | 含义 |
|------|------|
| `latest` | 远端最新版本号 |
| `release_note` | 该版本变更摘要 |
| `instruction` | 下载安装指引，包含 Skill 包的 CDN 下载链接 |

按 `instruction` 中的指引下载并解压替换当前 Skill 目录即可完成 Skill 更新。

## 兜底说明

若因权限或环境限制无法同步版本，以 `kdocs-cli` 实际支持的工具集为准。运行 `kdocs-cli --help` 查看当前可用工具。`SKILL.md` 中描述但 CLI 不支持的工具调用将返回 `unknown command`，可安全忽略。
