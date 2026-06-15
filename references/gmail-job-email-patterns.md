# Gmail Job Email Patterns

> Quick reference for parsing LinkedIn and JobsDB job alert emails.
> Full technical details: see `gmail-job-scanner.md`

## LinkedIn 职位订阅 (jobalerts-noreply@linkedin.com)

Subject: "公司名+岗位名" or subscription confirmation
Body structure:
```
[separator line ----]
Job Title Line
Company Name
Location
查看职位: https://www.linkedin.com/comm/jobs/view/JOB_ID
[separator line ----]
Next job block...
```

## LinkedIn 保存职位提醒 (jobs-noreply@linkedin.com)

Subject: "你的名字，立即申请"XXX的YYY岗位""
Body structure:
```
Main job title + company + location
查看职位: URL

其他已保存的职位
[separator ----]
Second job title + company + location
查看职位: URL
```

## JobsDB Recommendations (noreply@e.jobsdb.com)

Subject: "Job Title + N new jobs"
Body structure:
```
Hi YourName, we've got new job recommendations...

[logo image]
Job Title
Company Name
Location, District
$XX,000 – $YY,000 per month
* Bullet point requirement 1
* Bullet point requirement 2

[logo image]
Next Job Title
...
```

## Noise Patterns to Skip

### LinkedIn
- "简单几步，轻松迈向成功" (CTA button)
- "编辑订阅" / "其他订阅"
- "查看领英上的全部职位"
- "查看全部职位"
- "有符合您的搜索偏好" / "您已成功订阅"
- "该公司正在热招中"
- "使用简历和职业档案申请"
- Lines containing: Subject:, To:, From:, <strong, midToken, otpToken, trk=, eid=, lipi=

### JobsDB
- "Rate your recent employer"
- "apple store" / "google play"
- "Edit frequency"
- "View more jobs"
- "Recently posted"
- Lines starting with "* " (requirement bullets, not company names)
- Lines starting with "[" or "http" (URL tracking blocks)
- "%%str_to_replace_open_tracking%%"

### Both
- Any line that looks like email metadata (Subject:, To:, From:)
- HTML tags: <strong>, <a href=...>

## Glassdoor Job Alerts (noreply@glassdoor.com) — 2026-06-12

Subject: "{Job Title} at {Company} and N more jobs in Hong Kong for you. Apply Now."
Body: HTML with zero-width joiner characters (‌​‍‎‏﻿) as anti-scraping noise.

Structure per email:
```
Job alert: {Keyword}
Your job listings for {Date}
---
{Company Name} {Rating} ★
{Job Title}
{Location}
{Salary (optional, e.g. "hk$70k - hk$100k (employer est.)")}
{Posted time, e.g. "1d", "6d"}
{Glassdoor partner link with jobListingId}
---
(repeat for 5-8 jobs per email)
```

**Key fields to extract**:
- `jobListingId` (in URL param): unique ID for dedup across days
- Company name + rating
- Job title
- Salary (if present)
- Posted time (relative)

**Noise to skip**:
- Zero-width chars: strip `‌​‍‎‏﻿` (U+200C, U+200B, U+200E, U+200F, U+FEFF)
- "Create job alerts for related roles" section
- "Want more listings" CTA
- Footer: privacy policy, unsubscribe links, address
- `info@glassdoor.com` emails (marketing, not job alerts)

**Dedup**: Same `jobListingId` appears in consecutive days' emails. Dedup on jobListingId.

**JD fetching**: Glassdoor links are Cloudflare-blocked. Use company career sites or LinkedIn as fallback. See `glassdoor-hk-guide.md`.