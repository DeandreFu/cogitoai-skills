# 小红书自动发帖 Skill（Xiaohongshu Automation）

通过 OpenClaw + ADB 自动化发布小红书帖子，包含图片推送、文字输入（支持中文）、标签选择等操作。

## 快速开始

### 前提条件
- Android 设备已连接 ADB（见 [openclaw-adb-skill](../openclaw-adb-skill/SKILL.md)）
- OpenClaw 免审批配置已生效（`openclaw exec-policy preset yolo && openclaw gateway restart`）
- 待发布的图片已在本地路径（如 `/sdcard/Pictures/xiaohongshu_cover.jpg`）

### 标准发布流程

```bash
# 1. 推送图片到手机
adb -s <IP:PORT> push local_image.jpg /sdcard/Pictures/xiaohongshu_cover.jpg

# 2. 打开小红书并进入发帖页
adb -s <IP:PORT> shell am start -n com.xingin.xhs/.IndexActivity
sleep 2

# 3. 点击「发布」按钮（进入内容选择页）
adb -s <IP:PORT> shell input tap <x> <y>

# 4. 选择图片（从相册）
adb -s <IP:PORT> shell input tap <x_album> <y_album>
sleep 1

# 5. 确认选图（点击确定/完成）
adb -s <IP:PORT> shell input tap <x_confirm> <y_confirm>
sleep 1

# 6. 标题和正文中英文混写时用 ADB input text，纯中文用下面方案一/二
# （详见下方「中文输入解决方案」章节）

# 7. 添加标签（手动或用 tap 模拟键盘搜索）
adb -s <IP:PORT> shell input text "#后室"

# 8. 点击「发布笔记」按钮
adb -s <IP:PORT> shell input tap <x_publish> <y_publish>
```

---

## 核心问题一：图片选择安全

### ⚠️ 隐私安全警告

**必须使用确定路径的文件，避免误发生活照片到网络。**

常见错误：
- 假设相册里只有你想发布的照片
- 随便选一张最近的图
- 手动从相册选了你自己都没注意到的照片

**每次发图前，必须确认：推送到手机的图片路径 + 文件名完全正确。**

### 正确做法

1. 推送到固定位置：`/sdcard/Pictures/xiaohongshu_cover.jpg`（文件名固定，不混淆）
2. 推送后验证文件 MD5：
```bash
adb -s <IP:PORT> shell "md5sum /sdcard/Pictures/xiaohongshu_cover.jpg"
```
3. 在「选择图片」界面，确认从「最近」或「相机」等固定位置选择
4. 不要在「全部照片」里选择——那里面有你的生活照

### 工程化改进方案

```bash
# 用哈希验证文件，防止推错
LOCAL_FILE="/path/to/cover.jpg"
REMOTE_PATH="/sdcard/Pictures/xiaohongshu_cover.jpg"

# 推送前计算本地哈希
LOCAL_MD5=$(md5sum "$LOCAL_FILE" | cut -d' ' -f1)

# 推送到手机
adb -s <IP:PORT> push "$LOCAL_FILE" "$REMOTE_PATH"

# 推送后验证
REMOTE_MD5=$(adb -s <IP:PORT> shell "md5sum $REMOTE_PATH" | cut -d' ' -f1)

if [ "$LOCAL_MD5" = "$REMOTE_MD5" ]; then
  echo "✅ 文件验证通过"
else
  echo "❌ 文件不一致，终止发布"
  exit 1
fi
```

---

## 核心问题二：中文输入解决方案

`adb input text` **不支持中文**，会抛 `NullPointerException`。这是 Android 系统底层限制，与输入法无关。

以下是最优解法：

### 方案一：ADBKeyboard（最推荐，MIUI 除外）

原理：安装特殊输入法，通过 broadcast 接收文本，绕开键盘输入限制。

```bash
# 1. 安装 ADBKeyboard.apk
adb -s <IP:PORT> install ADBKeyboard.apk

# 2. 切换到 ADBKeyboard
adb -s <IP:PORT> shell ime set com.android.adbkeyboard/.AdbIME

# 3. 发送中文（broadcast 方式）
adb -s <IP:PORT> shell am broadcast -a ADB_INPUT_TEXT --es msg '野花盛开的年代'

# 4. 用完切回原输入法
adb -s <IP:PORT> shell ime set com.sohu.inputmethod.sogou/.SogouIME
```

**MIUI 兼容性：** 需在设置里手动允许第三方输入法切换（一次性的）。

### 方案二：剪贴板粘贴（最兼容）

```bash
# 通过 clipper broadcast 设置剪贴板
adb -s <IP:PORT> shell am broadcast \
  -a clipper.set \
  --es text '野花盛开的年代'

# 聚焦输入框后模拟粘贴
adb -s <IP:PORT> shell input keyevent 279  # KEYCODE_PASTE
```

**注意：** 部分 MIUI 版本限制了第三方应用访问剪贴板，需要用户手动允许。

### 方案三：uiautomator2（工程化最佳）

推荐用于 OpenClaw 自动化流程：

```bash
pip install uiautomator2
python -m uiautomator2 init  # 初始化设备
```

```python
import uiautomator2 as u2

d = u2.connect()  # USB 或网络 ADB

# 直接 set_text 支持中文（内部用 ADBKeyboard 机制）
d(resourceId="com.xingin.xhs:id/xxx").set_text("野花盛开的年代")
```

### 推荐优先级

| 方案 | MIUI 兼容 | 工程化难度 | 稳定性 |
|------|-----------|-----------|--------|
| uiautomator2 | ✅ 好 | 低 | ⭐⭐⭐⭐⭐ |
| ADBKeyboard | ⚠️ 需手动允许一次 | 低 | ⭐⭐⭐⭐ |
| 剪贴板粘贴 | ✅ 好 | 中 | ⭐⭐⭐ |

---

## 坐标获取方法

每次 UI 变动后需要重新抓取坐标：

```bash
# 抓取当前 UI 层级
adb -s <IP:PORT> shell uiautomator dump /sdcard/ui.xml

# 下载到本地分析
adb -s <IP:PORT> pull /sdcard/ui.xml /tmp/ui.xml

# 提取所有可点击文本的坐标
python3 -c "
import re
content = open('/tmp/ui.xml').read()
texts = re.findall(r'text=\"([^\"]+)\".*?bounds=\"\[(\d+),(\d+)\]\[(\d+),(\d+)\]\"', content)
for t,x1,y1,x2,y2 in texts:
    if t.strip():
        cx,cy = (int(x1)+int(x2))//2, (int(y1)+int(y2))//2
        print(f'{t}: [{x1},{y1}][{x2},{y2}] -> center({cx},{cy})')
"
```

---

## 已知限制

1. **中文输入**：必须使用 ADBKeyboard 或剪贴板方案
2. **图片选择**：每次必须确认推送到的是正确文件
3. **UI 坐标**：小红书 UI 经常更新，坐标需重新抓取
4. **发布前预览**：建议在发布前截图确认内容

---

## 经验教训

> **图片选择是隐私安全红线。** 发布前必须验证文件哈希值。
>
> **中文输入是 Android ADB 的经典痛点。** ADBKeyboard 是官方推荐解法，uiautomator2 是工程化最优解。
>
> **UI 自动化本质是坐标操作。** 小红书 UI 频繁变动，做好每次重新抓取坐标的准备。
