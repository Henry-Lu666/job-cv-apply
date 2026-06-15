# JD 抓取方法参考（已验证路径）

> 最后验证: 2026-06-10
> 用途: 从 LinkedIn / JobsDB 获取岗位完整 JD，用于 9 维度核实

---

## 一、LinkedIn JD 抓取

### 方法 A: curl 抓取 JD 全文（有 job ID 时，首选）

**前提**: 有 LinkedIn job ID（10 位数字，如 `4423900995`）

```bash
curl -s -L \
  -H "User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36" \
  "https://www.linkedin.com/jobs/view/{JOB_ID}"
```

**解析 JD 正文**（Python）:
```python
import re
match = re.search(r'<div class="show-more-less-html__markup[^"]*">(.*?)</div>', html, re.DOTALL)
if match:
    jd_text = re.sub(r'<[^>]+>', '', match.group(1)).strip()
```

**⚠️ 已知问题**:
- 返回的 HTML 中 JD 正文可能被截断到 ~3000 字符（LinkedIn guest 限制）
- 如果截断严重影响判断，需用户在本机 Chrome 打开看完整版
- 偶尔返回空页面（LinkedIn 反爬），重试一次通常成功
- 不需要登录，不需要 cookie

### 方法 B: browser 搜索 + console 提取 ID（无 ID 时）

**场景**: 只有公司名+岗位名（如 LinkedIn 0603 文件），需要先找到 job ID

```bash
# Step 1: browser_navigate 打开搜索页
https://www.linkedin.com/jobs/search?keywords={KEYWORDS}&location=Hong+Kong&f_TPR=r2592000

# Step 2: 关弹窗
browser_click(ref="e1")  # "Dismiss" 按钮

# Step 3: console 提取 job IDs
(() => {
    const links = document.querySelectorAll('a');
    const jobs = []; const seen = new Set();
    links.forEach(a => {
        if (a.href && a.href.includes('jobs/view/')) {
            const match = a.href.match(/-(\d{10})(?:\?|$)/);
            if (match && !seen.has(match[1])) {
                seen.add(match[1]);
                jobs.push({id: match[1], title: a.textContent.trim().substring(0, 60)});
            }
        }
    });
    return JSON.stringify({count: jobs.length, ids: jobs.map(j=>j.id)});
})()
```

**LinkedIn URL 格式**: `hk.linkedin.com/jobs/view/slug-TITLE-SLUG-4425452567?position=1&pageNum=0`
- Job ID 是最后一个 `-` 后的 10 位数字
- regex: `/-(\d{10})(?:\?|$)/`

**⚠️ 限制**:
- Guest 只能看到第一页（~15-60 条，取决于搜索结果数）
- 不能翻页（需登录）
- 旧岗位（发布 1 周+）可能搜不到
- LinkedIn Guest API (`/jobs-guest/jobs/api/seeMoreJobPostings`) **2026-06 起不可用**，返回空结果

### 方法 C: 用户本机 Chrome（最可靠，需用户配合）

当 A/B 方法失败或 JD 被截断时，让用户在本机 Chrome 打开链接：
- 完整 JD、无截断
- 可以看到薪资、公司规模等隐藏信息
- 用户贴回文本给 agent 做分析

---

## 二、JobsDB JD 抓取

### 方法 A: 搜索 API（批量发现，首选）

**已验证 URL**（2026-06-10 实测）:

```bash
# 搜索关键词
curl -s "https://hk.jobsdb.com/api/jobsearch/v5/search?siteKey=HK-Main&sourcesystem=houston&keywords=business+development+director&pagesize=30&page=1&sortmode=Relevance" \
  -H "Accept: application/json" \
  -H "User-Agent: Mozilla/5.0"
```

**⚠️ 参数大小写**: 实测 `pagesize`（小写）和 `sortmode`（小写）可以工作。文档中有些地方写 `pageSize`/`sortMode`（camelCase），两种都可能 work，但**小写更可靠**。

**按 job ID 单条查询**:
```bash
curl -s "https://hk.jobsdb.com/api/jobsearch/v5/search?siteKey=HK-Main&sourcesystem=houston&jobid=92625323&pagesize=1&page=1" \
  -H "Accept: application/json" \
  -H "User-Agent: Mozilla/5.0"
```

**JSON 返回结构**:
```python
data = json.loads(response)
jobs = data["data"]  # 数组，不是 "jobs"
for job in jobs:
    job["id"]           # 岗位 ID（去重用）
    job["title"]        # 岗位标题
    job["companyName"]  # 公司名（有时为 null）
    job["salaryLabel"]  # 薪资（经常为空）
    job["teaser"]       # JD 摘要（~200-500 字）
    job["bulletPoints"] # 要点列表
    job["listingDateDisplay"]  # 发布时间
    job["locations"]    # 地点列表 [{label: "..."}]
    job["advertiser"]   # {id, description}（公司名备用）
```

**关键**: 顶层 key 是 `data`（数组），不是 `jobs`。`totalCount` 可信。

### 方法 B: 浏览器打开详情页（❌ 不可用）

JobsDB 网页层有 Cloudflare Turnstile 防护：
- `browser_navigate` → "Just a moment..." 验证页
- `curl https://hk.jobsdb.com/job/{id}` → HTTP 403
- Google/Bing 搜 URL → 同样被挡

**唯一绕过方案**: 用户本机 Chrome（已登录/已过 Cloudflare）

### 方法 C: API 摘要 → 粗筛 → 用户手动核实（务实路径）

```
JobsDB API 搜索 → teaser + bulletPoints → 粗筛 70% 判断
  ↓ 匹配度 ≥ 50%
用户本机 Chrome 打开链接 → 看完整 JD → 贴回 agent → 9 维核实
```

API 摘要包含的信息足够判断：
- 行业方向（从 teaser 看）
- 管理层级（从 title 看）
- 薪资范围（salaryLabel，但经常为空）
- 核心职责（bulletPoints）

不够判断的（需完整 JD）：
- 硬性资质要求（证书、语言、行业年限）
- 具体技术栈要求
- 公司规模和文化

---

## 三、Gmail 邮件中的 JD 链接

邮件中的 LinkedIn/JobsDB 链接可以直接用上述方法抓取：
- LinkedIn 邮件链接: `https://www.linkedin.com/comm/jobs/view/{JOB_ID}` → 同方法 A
- JobsDB 邮件链接: `https://hk.jobsdb.com/job/{JOB_ID}` → 被 Cloudflare 挡，用 API 方法 A 单条查询

---

## 四、路径选择决策树

```
有 LinkedIn job ID?
  ├─ 是 → curl 抓取 JD 全文（方法 A）
  └─ 否 → 有公司名+岗位名?
           ├─ 是 → browser 搜索补 ID（方法 B）→ curl 抓取
           └─ 否 → 邮箱/Gmail 扫描找链接

有 JobsDB job ID?
  ├─ 是 → API 单条查询获取摘要
  └─ 否 → API 关键词搜索 → 从结果中匹配

JD 被截断 / 摘要不够?
  └─ 用户本机 Chrome 打开 → 贴回文本
```

---

## 五、批量抓取模板（execute_code）

```python
import subprocess, re, time, json

def fetch_linkedin_jd(job_id):
    """抓取 LinkedIn JD 全文"""
    result = subprocess.run(
        ["curl", "-s", "-L", "-H",
         "User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36",
         f"https://www.linkedin.com/jobs/view/{job_id}"],
        capture_output=True, text=True, timeout=20
    )
    match = re.search(r'<div class="show-more-less-html__markup[^"]*">(.*?)</div>', 
                      result.stdout, re.DOTALL)
    return re.sub(r'<[^>]+>', '', match.group(1)).strip() if match else None

def fetch_jobsdb_summary(job_id):
    """获取 JobsDB 摘要"""
    result = subprocess.run(
        ["curl", "-s",
         f"https://hk.jobsdb.com/api/jobsearch/v5/search?siteKey=HK-Main&sourcesystem=houston&jobid={job_id}&pagesize=1&page=1",
         "-H", "Accept: application/json", "-H", "User-Agent: Mozilla/5.0"],
        capture_output=True, text=True, timeout=15
    )
    data = json.loads(result.stdout)
    jobs = data.get("data", [])
    return jobs[0] if jobs else None

def jobsdb_keyword_search(keywords, page=1, pagesize=30):
    """JobsDB 关键词搜索"""
    kw = keywords.replace(" ", "+")
    result = subprocess.run(
        ["curl", "-s",
         f"https://hk.jobsdb.com/api/jobsearch/v5/search?siteKey=HK-Main&sourcesystem=houston&keywords={kw}&pagesize={pagesize}&page={page}&sortmode=Relevance",
         "-H", "Accept: application/json", "-H", "User-Agent: Mozilla/5.0"],
        capture_output=True, text=True, timeout=20
    )
    data = json.loads(result.stdout)
    return data.get("data", []), data.get("totalCount", 0)

# 批量抓取示例
job_ids = ["4423900995", "4415327220", "4273469658"]
for jid in job_ids:
    jd = fetch_linkedin_jd(jid)
    print(f"{jid}: {'OK' if jd else 'FAIL'} ({len(jd) if jd else 0} chars)")
    time.sleep(0.5)  # 防限流
```
