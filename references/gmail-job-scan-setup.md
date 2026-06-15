# Gmail 求职邮件扫描 — 架构与配置参考

## 方案对比

| 方案 | 复杂度 | 功能范围 | 适用场景 |
|------|--------|---------|---------|
| **A. Himalaya + App Password** | ★☆☆ 5分钟 | 仅邮件读取/搜索/发送 | 只需读求职邮件，不需要日历/Drive |
| **B. Google Workspace OAuth** | ★★★ 20分钟 | 邮件+日历+Drive+Sheets+Docs | 需要全功能 Google 集成 |

**当前选择**：方案A（Himalaya），因为求职邮件扫描只需 IMAP 读取能力。

## Himalaya 配置模板（Gmail）

配置文件：`~/.config/himalaya/config.toml`

```toml
[accounts.personal]
email = "your.email@gmail.com"
display-name = "Your Name"
default = true

backend.type = "imap"
backend.host = "imap.gmail.com"
backend.port = 993
backend.encryption.type = "tls"
backend.login = "your.email@gmail.com"
backend.auth.type = "password"
backend.auth.cmd = "cat ~/.config/himalaya/gmail-app-password"

message.send.backend.type = "smtp"
message.send.backend.host = "smtp.gmail.com"
message.send.backend.port = 587
message.send.backend.encryption.type = "start-tls"
message.send.backend.login = "your.email@gmail.com"
message.send.backend.auth.type = "password"
message.send.backend.auth.cmd = "cat ~/.config/himalaya/gmail-app-password"

# Gmail folder aliases (必须用 v1.2.0+ 语法)
folder.aliases.inbox = "INBOX"
folder.aliases.sent = "[Gmail]/Sent Mail"
folder.aliases.drafts = "[Gmail]/Drafts"
folder.aliases.trash = "[Gmail]/Trash"
```

## App Password 获取步骤

1. 确认 Google 账号已开启两步验证（2FA）
   - 未开启 → https://myaccount.google.com/security → 开启
   - 报错"您的账号不支持您正在尝试的设置" → 2FA 未开（见下方 Pitfall）
2. 打开 https://myaccount.google.com/apppasswords
3. 应用名填 `Himalaya`，点创建
4. 复制 16 位密码，保存到 `~/.config/himalaya/gmail-app-password`

## Pitfall: Workspace 账号限制

如果 Gmail 是 Google Workspace（企业/学校）账号，管理员可能禁用了 App Password。
此时必须改用 Google Workspace skill（OAuth 方案），无法绕过。

判断方法：用的是 @gmail.com → 个人账号，通常没问题。
用的是 @company.com / @school.edu → Workspace 账号，可能受限。

## 求职邮件搜索命令速查

```bash
# 过去24小时所有求职平台邮件
himalaya envelope list "from:linkedin.com OR from:jobsdb.com OR from:indeed.com newer_than:1d"

# LinkedIn 专属
himalaya envelope list "from:linkedin.com subject:job newer_than:7d"

# JobsDB 专属
himalaya envelope list "from:jobsdb.com newer_than:7d"

# 读取具体邮件
himalaya message read <id>

# JSON 输出（便于脚本解析）
himalaya envelope list --output json "from:linkedin.com newer_than:1d"
```

## 切换到 Google Workspace 的触发条件

当出现以下需求时，从 Himalaya 切换到 Google Workspace skill：
- 需要读取日历安排面试
- 需要访问 Google Drive 存储简历/作品集
- 需要 Google Sheets 做求职追踪表
- 需要 Gmail 高级功能（标签管理、过滤器自动分类）

切换时无需卸载 Himalaya，两者可以共存。
