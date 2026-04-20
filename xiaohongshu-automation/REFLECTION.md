# 反思记录：小红书自动发帖

**日期：** 2026-04-19
**问题：** ADB 自动化发布小红书帖子
**耗时：** 约 20 分钟（不包括 ADB 免审批配置时间）

---

## 一、结果

**发布成功。** 帖子使用了英文标题 "WildFlowersEra"，图片正确推送到了 `/sdcard/Pictures/xiaohongshu_cover.jpg`。

但暴露了两个严重问题需要解决。

---

## 二、问题一：图片选择隐私安全

### 现象

在「选择图片」界面点击「相册」时，展示了手机里**所有照片**——这台是旧手机，里面有不少生活照片。

如果选错一张，或者 UI 状态不对（比如默认选中了上一张生活照片），就会把私人照片发到网上。

### 根因

**小红书的图片选择逻辑不是「先推文件再选」，而是「从相册选」。**

相册里有这台手机所有照片，发布动作会把选中的照片上传到网络。

### 教训

> **图片发布是隐私安全红线。**
>
> 不能假设计载里只有你想发布的照片。
> 不能相信 UI 默认状态。
> 不能在「全部照片」里操作。

### 正确做法

**每次发图前，必须确认推送到手机的图片路径 + 文件名完全正确。**

```bash
# 1. 推送到固定位置（文件名固定，不混淆）
adb -s <IP:PORT> push cover.jpg /sdcard/Pictures/xiaohongshu_cover.jpg

# 2. 推送后验证 MD5
LOCAL_MD5=$(md5sum cover.jpg | cut -d' ' -f1)
REMOTE_MD5=$(adb -s <IP:PORT> shell "md5sum /sdcard/Pictures/xiaohongshu_cover.jpg" | cut -d' ' -f1)

if [ "$LOCAL_MD5" = "$REMOTE_MD5" ]; then
  echo "✅ 文件验证通过"
else
  echo "❌ 文件不一致，终止发布"
  exit 1
fi

# 3. 在「选择图片」时，确认从「最近」或「相机」等固定位置选择
# 不要从「全部照片」里选
```

---

## 三、问题二：中文输入

### 现象

`adb input text "野花盛开的年代"` 抛出：
```
NullPointerException: Attempt to get length of null array
```

### 根因

**这是 Android 系统设计层面的限制，不是配置问题。**

`adb input text` 本质是模拟键盘输入，底层调用 `InputConnection`——它只接受字符数组，非 ASCII 字符（如中文）在 Java 层会被当作 null 处理。

### 解决方案探索

| 方案 | 原理 | MIUI 兼容 | 工程化 |
|------|------|-----------|--------|
| ADBKeyboard | broadcast 接收文本，绕开 InputConnection | ⚠️ 需手动允许一次 | 低 |
| 剪贴板粘贴 | 通过 Broadcast 设置剪贴板再粘贴 | ✅ 好 | 中 |
| uiautomator2 | 封装了 ADBKeyboard 机制 | ✅ 好 | 低（最优） |

**最终推荐：uiautomator2。** 它专门为 Android 自动化设计，内置中文支持，安装时自动推 ADBKeyboard 到设备。

---

## 四、经验总结

### 隐私安全
- 推送前计算 MD5，验证后发布
- 文件名固定（xiaohongshu_cover.jpg），推送到固定目录
- 在「选择图片」界面，确认从「最近」等确定位置选择

### 中文输入
- 不依赖 ADB input text
- 优先用 uiautomator2（`pip install uiautomator2`）
- 备选：ADBKeyboard broadcast 方式

### UI 自动化
- 每次 UI 变动后重新抓取坐标（`uiautomator dump`）
- 小红书 UI 频繁变动，做好每次重新获取坐标的准备
- 截图确认内容后再发布

---

## 五、举一反三

| 教训 | 下次怎么做 |
|------|----------|
| 图片发布要验证哈希 | 写脚本自动验证，不靠人眼确认 |
| 中文输入是系统限制 | 先想好方案再动手，不要试错 |
| UI 自动化要做好重新抓取坐标的准备 | 把坐标抓取流程标准化 |
| 小红书 UI 频繁变动 | 把操作步骤参数化，方便更新 |

---

_小红书发帖自动化经验已整理为 Skill，分享给社区。_
