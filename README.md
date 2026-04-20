# cogitoai-skills

武汉市佳哲科技工作室的 OpenClaw Skills 开源项目集。

## 项目列表

### openclaw-adb-skill
通过 OpenClaw 远程连接并控制 Android 设备。

**包含：** ADB 连接、免审批配置（双层架构详解）、常用命令、调试技巧。

**GitHub:** https://github.com/DeandreFu/cogitoai-skills/tree/main/openclaw-adb-skill

### xiaohongshu-automation
通过 OpenClaw + ADB 自动化发布小红书帖子。

**包含：** 图片安全推送、中文输入解决方案（ADBKeyboard/uiautomator2/剪贴板）、标签管理、坐标抓取。

**GitHub:** https://github.com/DeandreFu/cogitoai-skills/tree/main/xiaohongshu-automation

### clawhub-sync
将 OpenClaw Skills 同步发布到 ClawHub 平台。

**包含：** clawhub CLI 使用、发布流程、版本管理。

**GitHub:** https://github.com/DeandreFu/cogitoai-skills/tree/main/clawhub-sync

## 快速开始

```bash
# 安装 skill
git clone https://github.com/DeandreFu/cogitoai-skills ~/cogitoai-skills

# 复制到 OpenClaw skills 目录
cp -r openclaw-adb-skill ~/.npm-global/lib/node_modules/openclaw/skills/
cp -r xiaohongshu-automation ~/.npm-global/lib/node_modules/openclaw/skills/

# 验证安装
openclaw skills list
```

## 经验教训

> **ADB 中文输入是 Android 自动化的经典痛点。**
> 解决方案优先级：uiautomator2 > ADBKeyboard > 剪贴板

> **图片发布前必须验证哈希值。**
> 隐私安全红线：不要从「全部照片」里选图。

> **OpenClaw 问题第一步永远是查官方文档。**
> v2026.4.x 更新快，文档是唯一准确来源。

## 开源协议

MIT
