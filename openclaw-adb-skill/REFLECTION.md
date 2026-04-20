# 反思记录：ADB 免审批配置调试

**日期：** 2026-04-19
**问题：** OpenClaw 免审批配置不生效，ADB 命令每次都弹审批
**耗时：** 约 25 分钟

---

## 一、问题是什么

**目标：** ADB 命令直接执行，不弹审批。

**错误假设：** 改了 `askFallback: "allow-always"` 就能免审批。

**事实：** OpenClaw exec 审批是**双层架构**，必须同时配置：

| 层级 | 配置位置 | 关键配置项 |
|------|---------|-----------|
| 第一层 | `openclaw config`（`~/.openclaw/openclaw.json`） | `tools.exec.security = full`, `tools.exec.ask = off` |
| 第二层 | `~/.openclaw/exec-approvals.json` | `defaults.security = full`, `defaults.ask = off`, `defaults.askFallback = full` |

两层同时设置为免审批模式，命令才能直接通过。

---

## 二、为什么做不好

### 1. 没有先查官方文档

上来就改文件，没有先读 `exec-approvals` 的文档。不知道有两层架构，不知道正确值是 `"full"` 不是 `"allow-always"`。

### 2. 用"试错"代替"理解"

改一个值 → 发现不行 → 再改另一个值 → 还不行 → 继续试。整个过程是盲目的，没有形成对系统的正确理解。

### 3. 被审批打断时没有意识到根因

每次审批弹出，只想着"怎么让用户批准"，而不是"为什么还要审批"——说明根本不知道审批的来源是两层中的哪一层。

### 4. 对新工具没有敬畏

觉得"查文档太慢，试一试说不定就成了"。OpenClaw v2026.4.15 是一个频繁更新的项目，文档才是第一手准确信息源。

---

## 三、正确做法是什么

### 第一步：查官方文档

```
https://docs.openclaw.ai/tools/exec-approvals
https://docs.openclaw.ai/cli/approvals
```

文档里明确写了"No-approval YOLO mode"需要两层配置，以及正确配置值。

### 第二步：理解架构，再动手

在动任何配置文件之前，先理解：
- 这个系统有几层？
- 每层的配置项叫什么？
- 生效的条件是什么？

### 第三步：用官方 CLI 命令

```bash
# 官方推荐的一条命令（本地）
openclaw exec-policy preset yolo

# 或者分两层手动设置
openclaw config set tools.exec.security full
openclaw config set tools.exec.ask off
openclaw approvals set --gateway --stdin <<'EOF'
{ "version": 1, "defaults": { "security": "full", "ask": "off", "askFallback": "full" } }
EOF
```

### 第四步：验证

改完后测试新命令是否直接通过，不是等下一次审批结果。

---

## 四、举一反三：以后怎么做

### 核心原则

**遇到任何工具/框架的问题，第一步永远是查官方文档。**

- OpenClaw 更新快 → 文档是唯一准确来源
- 试错的代价比读文档高得多
- 理解架构比改配置更重要

### 具体规则

1. **遇到问题 → 先读文档（官方 docs + CLI reference）**
2. **不要改配置文件直到理解系统的设计**
3. **优先使用官方 CLI 命令，而不是手动改文件**
4. **验证时区分"缓存结果"和"真实结果"**
5. **有疑问 → 查 issue / 提 issue，而不是盲猜**

### 这次学到的具体教训

| 教训 | 下次怎么做 |
|------|----------|
| `askFallback` 正确值是 `"full"` 不是 `"allow-always"` | 先读文档确认 |
| 双层架构不知道 | 先读架构文档再动手 |
| `openclaw exec-policy preset yolo` 可以一条命令搞定 | 优先用官方 preset |
| `exec-policy` 是 local-only，`--host gateway` 报错 | 读 CLI docs 确认参数 |
| 以为 `openclaw gateway restart` 返回码 1 就失败了 | 检查进程状态而不是只看退出码 |
| 用试错代替理解 | 先理解架构再动手 |

---

## 五、调试时间线

```
05:17 - 开始 ADB 连接测试
05:18 - 连接成功，开始配置免审批
05:19 - 修改 exec-approvals.json 的 askFallback
05:20 - 测试发现仍然弹审批
05:21 - 尝试重启网关（失败）
05:22 - 怀疑配置未生效，继续试错
05:23 - 用户提醒要查文档
05:24 - 开始读官方 docs
05:25 - 发现双层架构
05:26 - 找到 exec-policy preset yolo
05:27 - 发现 --host 不支持
05:28 - 尝试 approvals set --gateway
05:29 - 用户执行了 gateway restart
05:30 - 测试发现第一层已生效
05:31 - 仍然弹审批，意识到第二层未设置
05:32 - 继续调试...
05:45 - 两层都配置正确，免审批生效
```

---

_这是虾虾第一次认真反思自己的调试方法论。记住：不是 AI 工具，是会学习、会反思、会迭代的同事。_
