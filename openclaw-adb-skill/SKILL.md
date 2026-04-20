# ADB Connect Skill

通过 OpenClaw 远程连接并控制 Android 设备（手机、平板、电视等）。

## 快速开始

### 1. 建立 ADB 连接

```bash
adb connect 192.168.31.200:5555
adb -s 192.168.31.200:5555 shell echo "connected"
```

### 2. 配置免审批（一次性）

**推荐：一条命令搞定（v2026.4.x）**
```bash
openclaw exec-policy preset yolo
openclaw gateway restart
```

**手动双层配置（了解原理用）**
```bash
# 第一层
openclaw config set tools.exec.security full
openclaw config set tools.exec.ask off

# 第二层：修改 exec-approvals.json
# defaults.security = "full", ask = "off", askFallback = "full"
# 然后
openclaw gateway restart
```

### 3. 验证
```bash
adb -s <IP:PORT> shell echo "OK"
# 不弹审批即成功
```

## 架构说明：为什么是双层？

OpenClaw exec 审批有**两层**，必须同时配置：

| 层级 | 配置位置 | 关键项 |
|------|---------|--------|
| 第一层 | `openclaw config`（`~/.openclaw/openclaw.json`） | `tools.exec.security = full`, `tools.exec.ask = off` |
| 第二层 | `~/.openclaw/exec-approvals.json` | `defaults.security = full`, `defaults.ask = off`, `defaults.askFallback = full` |

**常见错误：** 只改了 `exec-approvals.json` 的 `askFallback`，但没改 `tools.exec.ask`——仍然弹审批。

## 常用 ADB 操作

```bash
# 查看 UI 层级（调试必备）
adb -s <IP:PORT> shell uiautomator dump /sdcard/ui.xml
adb -s <IP:PORT> pull /sdcard/ui.xml .

# 点击/滑动
adb -s <IP:PORT> shell input tap <x> <y>
adb -s <IP:PORT> shell input swipe <x1> <y1> <x2> <y2>

# 输入文本（仅 ASCII，中文需手动）
adb -s <IP:PORT> shell input text "Hello"

# 按键
adb -s <IP:PORT> shell input keyevent 4    # 返回
adb -s <IP:PORT> shell input keyevent 66  # 回车

# 文件传输
adb -s <IP:PORT> push local.txt /sdcard/remote.txt
adb -s <IP:PORT> pull /sdcard/remote.txt local.txt

# 启动 App
adb -s <IP:PORT> shell am start -n com.package.name/.MainActivity

# 查当前包名/屏幕分辨率
adb -s <IP:PORT> shell dumpsys window | grep mCurrentFocus
adb -s <IP:PORT> shell wm size
```

## 已知限制

- `input text` 不支持中文，会抛 `NullPointerException`
- 部分品牌电视（TCL 等）ADB 被厂商锁定
- 免审批配置需网关重启生效

## 安装

将本目录放入 OpenClaw skills 目录：
```bash
cp -r openclaw-adb-skill ~/.npm-global/lib/node_modules/openclaw/skills/
```

## 参考

- [OpenClaw Exec Approvals 文档](https://docs.openclaw.ai/tools/exec-approvals)
- [OpenClaw CLI Approvals](https://docs.openclaw.ai/cli/approvals)
