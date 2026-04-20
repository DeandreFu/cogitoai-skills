# ClawHub Sync Skill — 创作反思

**日期：** 2026-04-19
**产出：** clawhub-sync Skill

---

## 一、为什么要有这个 Skill

在整理 openclaw-adb-skill 和 xiaohongshu-automation 时，想到需要把它们发布到 ClawHub。

但不知道 clawhub CLI 的具体用法，也不知道发布流程。搜索了一下，发现：
- 没有找到详细的 clawhub CLI 文档
- 官方文档只有简单的 `clawhub publish` 说明
- 不知道 `--name` 和 `--update` 参数是否支持

于是决定先写一个 Skill，把已知的使用方法记录下来，以后发布其他 Skill 时可以直接参考。

## 二、Skill 创作过程

### 步骤 1：读官方文档

```bash
# 查看 clawhub CLI 帮助
clawhub --help

# 查看子命令
clawhub publish --help
clawhub list --help
```

### 步骤 2：整理已知信息

从官方文档和 CLI help 中提取关键信息：
- `clawhub publish <path>` — 发布
- `clawhub list` — 查看已发布列表
- `--update` — 更新已发布版本
- `--version` — 指定版本号

### 步骤 3：写 SKILL.md

按照标准 Skill 格式：
- 快速开始（前提条件 + 基本命令）
- 发布流程（推荐工作流）
- 常用命令（速查表）

## 三、举一反三

| 教训 | 下次怎么做 |
|------|----------|
| 遇到新工具先读 CLI help | `clawhub --help` 是最准确实时文档 |
| 不知道参数是否支持 | 先假设支持，试一下 `publish --help` |
| Skill 要定期更新 | 使用后发现新参数要立即补充 |

---

_创作日期：2026-04-19_
