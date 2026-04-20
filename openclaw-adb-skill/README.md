# openclaw-adb-skill

通过 OpenClaw 远程连接并控制 Android 设备。一套完整的 ADB + OpenClaw 免审批配置指南。

## 功能

- 🔌 ADB 远程连接 Android 设备（手机/平板/电视）
- 🎯 坐标点击、滑动、按键模拟
- 📱 App 启动、UI 层级抓取、文件传输
- ⚙️ OpenClaw v2026.4.x 免审批配置（双层架构详解）

## 快速开始

```bash
# 1. 连接设备
adb connect 192.168.31.200:5555

# 2. 配置免审批（一行命令）
openclaw exec-policy preset yolo && openclaw gateway restart

# 3. 验证
adb -s 192.168.31.200:5555 shell echo "OK"
```

详细步骤见 [SKILL.md](./SKILL.md)

## 重要：双层配置说明

OpenClaw exec 审批有**两层**，很多教程只告诉你改一层，导致免审批不生效：

| 层级 | 配置位置 | 必须设置 |
|------|---------|---------|
| 第一层 | `openclaw config` | `tools.exec.security=full`, `tools.exec.ask=off` |
| 第二层 | `~/.openclaw/exec-approvals.json` | `defaults.security=full`, `defaults.ask=off`, `defaults.askFallback=full` |

两层都要改，缺一不可。

## 经验教训

这个 Skill 的诞生来自一次痛苦的调试过程。详见：
[反思记录](https://github.com/DeandreFu/openclaw-adb-skill/blob/main/REFLECTION.md)

**核心教训：遇到 OpenClaw 问题，第一步永远是查官方文档。**

v2026.4.x 更新频繁，网上搜到的旧方案很可能已过时。

## 项目结构

```
openclaw-adb-skill/
├── SKILL.md       # Skill 主文件（OpenClaw 标准格式）
├── README.md      # 本文件
└── REFLECTION.md   # 调试过程反思记录
```

## 安装 Skill

```bash
# 方法 1：克隆到 skills 目录
git clone https://github.com/DeandreFu/openclaw-adb-skill /path/to/skills/openclaw-adb-skill

# 方法 2：从 ClawHub 安装（如可用）
openclaw skill install openclaw-adb-skill
```

## 开源协议

MIT
