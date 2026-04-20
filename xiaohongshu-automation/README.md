# xiaohongshu-automation

通过 OpenClaw + ADB 自动化发布小红书帖子。支持中文标题输入、图片安全推送、标签管理等。

## 快速开始

```bash
# 前提：ADB 连接 + 免审批配置（见 openclaw-adb-skill）
openclaw exec-policy preset yolo && openclaw gateway restart

# 1. 推送图片（必须验证）
adb -s <IP> push cover.jpg /sdcard/Pictures/xiaohongshu_cover.jpg
adb -s <IP> shell "md5sum /sdcard/Pictures/xiaohongshu_cover.jpg"

# 2. 打开小红书
adb -s <IP> shell am start -n com.xingin.xhs/.IndexActivity

# 3. 按坐标操作发布流程（详见 SKILL.md）
```

## 核心功能

### 中文输入（最关键痛点）
- **推荐：uiautomator2** — `pip install uiautomator2`，内置中文支持
- **备选：ADBKeyboard** — broadcast 方式，支持中文
- **不推荐：剪贴板** — 兼容性差

### 图片安全
- 推送后必须验证 MD5
- 文件名固定：xiaohongshu_cover.jpg
- 从「最近」选择，不从「全部照片」选

## 经验教训

> **图片是隐私安全红线。** 每次发布前必须验证文件哈希。

> **中文输入是 Android ADB 的经典痛点。** 不要用 `adb input text` 输入中文。

## 项目结构

```
xiaohongshu-automation/
├── SKILL.md        # 技能说明 + 详细操作流程
├── README.md       # 本文件
└── REFLECTION.md   # 调试过程复盘与经验总结
```

## 开源协议

MIT
