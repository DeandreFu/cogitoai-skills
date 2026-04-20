# ClawHub Sync Skill

将 OpenClaw Skills 同步发布到 ClawHub（openclaw.com/skills）平台。

## 快速开始

### 前提条件
- npm 全局安装了 clawhub CLI
- 已在 ClawHub 网站登录并获取 API Token

```bash
npm install -g clawhub
```

### 验证 clawhub CLI
```bash
clawhub --version
```

## 同步本地 Skills 到 ClawHub

### 方式一：发布整个目录

```bash
clawhub publish /path/to/your/skill-folder
```

### 方式二：指定具体 skill

```bash
clawhub publish /path/to/skills/openclaw-adb-skill --name "openclaw-adb-skill"
```

### 方式三：更新已发布的 skill（指定版本）

```bash
clawhub publish /path/to/skills/openclaw-adb-skill --version 1.0.1 --update
```

## 工作流程

推荐在发布前：
1. 确认 SKILL.md 格式符合规范
2. 更新版本号
3. 运行本地测试
4. commit 到 GitHub
5. 再发布到 ClawHub

```bash
# 典型发布流程
cd ~/clawd/skills
# 编辑 skill 内容
git add -A && git commit -m "feat: update openclaw-adb-skill"
git push origin main
# 发布到 ClawHub
clawhub publish ./openclaw-adb-skill --name "openclaw-adb-skill"
```

## 常用命令

```bash
# 登录（获取 API Token 后）
clawhub login

# 查看已发布列表
clawhub list

# 更新已发布 skill
clawhub publish --update ./my-skill --version 1.0.2

# 删除已发布 skill
clawhub delete my-skill

# 搜索 skills
clawhub search "adb"
```

## 相关资源

- ClawHub 官网：https://clawhub.ai
- 官方文档：https://docs.openclaw.ai/cli/skills
