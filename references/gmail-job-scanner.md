# Gmail Job Scanner — Technical Reference

## Architecture (2026-06-10 升级为5步)

```
Gmail (IMAP) → himalaya CLI → gmail_job_scanner.py → structured report
         ↓                                        → desktop report (~/job-search/inbox/)
  LinkedIn / JobsDB /                              → archive (raw/notes/jobs/)
  OfferToday / Indeed /
  Glassdoor 邮件解析
         +
  JobsDB API 关键词搜索
  (8组方向关键词 × 15条)
```

## Himalaya Setup

### Installation
```bash
curl -sSL https://raw.githubusercontent.com/pimalaya/himalaya/master/install.sh | PREFIX=~/.local sh
```

### Gmail App Password Config
- File: `~/.config/himalaya/config.toml`
- Password file: `~/.config/himalaya/gmail-app-password` (chmod 600)
- **Prerequisite**: 2FA must be enabled on Google account FIRST
- Without 2FA: App Passwords page shows "您的账号不支持您正在尝试的设置"

### Config template
```toml
[accounts.personal]
email = "user@gmail.com"
display-name = "Name"
default = true

backend.type = "imap"
backend.host = "imap.gmail.com"
backend.port = 993
backend.encryption.type = "tls"
backend.login = "user@gmail.com"
backend.auth.type = "password"
backend.auth.cmd = "cat ~/.config/himalaya/gmail-app-password"

message.send.backend.type = "smtp"
message.send.backend.host = "smtp.gmail.com"
message.send.backend.port = 587
message.send.backend.encryption.type = "start-tls"
message.send.backend.login = "user@gmail.com"
message.send.backend.auth.type = "password"
message.send.backend.auth.cmd = "cat ~/.config/himalaya/gmail-app-password"

# Gmail folder aliases (REQUIRED for v1.2.0+)
folder.aliases.inbox = "INBOX"
folder.aliases.sent = "[Gmail]/Sent Mail"
folder.aliases.drafts = "[Gmail]/Drafts"
folder.aliases.trash = "[Gmail]/Trash"
```

## Pitfalls

### P1: himalaya search syntax is NOT Gmail syntax
- Correct (IMAP lowercase): `from "linkedin"`, `subject "job"`
- Wrong (Gmail): `from:linkedin.com` → parse error
- Combined: `from "linkedin" subject "job"` works but may be flaky; prefer single filter + Python-side filtering

### P2: JSON output has WARN lines mixed in
```python
output = run_himalaya(["envelope", "list", "--output", "json", ...])
start = output.find('[')
data = json.loads(output[start:])  # skip WARN lines
```

### P3: `from` field is dict, not string
```json
{"from": {"name": "领英", "addr": "messages-noreply@linkedin.com"}}
```
Helper:
```python
def get_sender_addr(env):
    frm = env.get("from", "")
    return frm.get("addr", "") if isinstance(frm, dict) else str(frm)
```

### P4: LinkedIn email noise patterns
Must skip in parser: "简单几步，轻松迈向成功", "编辑订阅", "其他订阅", "查看领英", "查看全部职位", "有符合您的搜索偏好", "您已成功订阅", "Subject:", "To:", "From:", "<strong"

### P5: JobsDB email noise patterns
Must skip: "Rate your recent employer", "apple store", "google play", "Edit frequency", "View more jobs", "[https://", lines starting with `* ` (requirement bullets, not company names)

### P6: Hong Kong district names as company
"九龙城区", "中西區", "湾仔区" etc. get misidentified as company names. Post-fix in score_job():
```python
location_names = ["九龙城区", "中西區", "湾仔区", ...]
if job.get("company", "") in location_names:
    job["location"] = job["company"]
    job["company"] = ""
```

### P7: Message IDs are folder-relative
Switching folders invalidates previously listed IDs. Always re-list after folder change.

## Email Format Patterns

### LinkedIn Job Alert Digest (领英职位订阅)
- Sender: `jobalerts-noreply@linkedin.com`
- Body: Multiple jobs separated by `----` lines
- Each block: title, company, location, then "查看职位: URL"
- URL pattern: `https://www.linkedin.com/comm/jobs/view/JOB_ID`

### LinkedIn Saved Job Reminder (立即申请)
- Sender: `jobs-noreply@linkedin.com`
- Subject: "你的名字，立即申请"XXX的YYY岗位""
- Body: Main job + "其他已保存的职位" section with additional jobs

### JobsDB Recommendations
- Sender: `noreply@e.jobsdb.com`
- Subject: "岗位名 + N new jobs"
- Body: Structured listings with title, company, location, salary

### OfferToday (BOSS直聘香港版) — 2026-06-10 新增
- Sender: `offertoday.com` 或 `zhipin` 相关域名
- Subject: 通常含"推荐"关键词
- Body: 岗位链接含 `offertoday.com` 或 `zhipin`
- ⚠️ 推荐质量偏低，容易推保险/理财马甲岗

### Indeed — 2026-06-10 新增
- Sender: `donotreply@match.indeed.com`
- Subject: 含"推荐"或"recommend"或"match"
- Body: `viewjob` 链接

### Glassdoor — 2026-06-10 新增
- Sender: `noreply@glassdoor.com` 或 `info@glassdoor.com`
- Subject: 含"推荐"/"recommend"/"alert"
- Body: `glassdoor.*job` 链接

## JobsDB API Keyword Search (Step 4)

Scanner 在邮件解析之后，额外用 JobsDB 搜索 API 做关键词补充搜索（覆盖 CTgoodjobs/劳工处同类岗位）：

```python
# 8组方向关键词
api_keywords = [
    "operations director",
    "business development director",
    "general manager Hong Kong",
    "partnership manager",
    "digital transformation manager",
    "AI product manager",
    "growth manager",
    "marketing director",
]
# 每组搜15条，去重后合并到 all_jobs
# 来源标记为 "JobsDB-API"（区别于邮件中的 "JobsDB"）
```

⚠️ 中文公司名在 JobsDB API 不返回结果，必须用英文名。

## Scoring Logic
- high keywords (AI, product manager, digital transformation, etc.): 3 points each
- medium keywords (strategy, director, operations, etc.): 1 point each
- EXCLUDED: blacklist match (insurance, trainee, assistant, L&D)
- ★★★★★ ≥6, ★★★★ 4-5, ★★★ 2-3, ★★ 1, ★ 0

## Output Locations
- Desktop report: `~/job-search/inbox/Gmail岗位扫货_YYYYMMDD.md` (format matches existing `LinkedIn岗位扫货_*.md`)
- **输出方式**：用 shell 重定向 `> "~/桌面/Gmail岗位扫货_YYYYMMDD.md"`（不用 `--save`，`--save` 只存到知识库 archive）
- stderr 重定向到临时文件查看进度：`2>/tmp/scan_stderr.txt`
- Archive（知识库备份）：`~/你的知识库路径/raw/notes/jobs/YYYY-MM-DD-email-jobs.md`（需 `--save` 参数）
- Tracking table: append new jobs (deduplicated by job ID) to `job-search-2026.md`

### 实际执行命令
```bash
python3 ~/.hermes/scripts/gmail_job_scanner.py --days 7 --output text 2>/tmp/scan_stderr.txt > "~/桌面/Gmail岗位扫货_YYYYMMDD.md"
cat /tmp/scan_stderr.txt  # 查看扫描进度和统计
```

## Cron
- Job ID: `40292eff1879`
- Schedule: daily 10:00
- Delivery: default local (user can update to feishu)
- **Prompt 已更新（2026-06-10）**：包含5步扫描说明，标注新数据源（OfferToday/Indeed/JobsDB-API）

## 邮件发送者速查表（2026-06-10 更新）

| Sender | 平台 | Scanner是否覆盖 |
|--------|------|---------------|
| jobalerts-noreply@linkedin.com | LinkedIn 订阅 | ✅ Step 1 |
| jobs-noreply@linkedin.com | LinkedIn 提醒 | ✅ Step 1 |
| noreply@e.jobsdb.com | JobsDB 推荐 | ✅ Step 2 |
| noreply@email.jobsdb.com | JobsDB 其他 | ✅ Step 2 |
| offertoday.com / zhipin | OfferToday | ✅ Step 3 |
| donotreply@match.indeed.com | Indeed | ✅ Step 3 |
| noreply@glassdoor.com / info@glassdoor.com | Glassdoor | ✅ Step 3 |
| noreply@research.jobsdb.com | JobsDB 调研 | ❌ 跳过（非岗位推荐） |
| noreply@mail.apply.careers.hsbc.com | HSBC 投递确认 | ❌ 跳过（非推荐） |