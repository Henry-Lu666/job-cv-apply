# JD抓取方法 (2026-06 更新)

## 方法A: LinkedIn JD抓取 ⚠️ curl已失效，browser可用

**curl方式已失效(2026-06-22实测)**: 访问LinkedIn岗位页返回登录墙，无法提取JD。regex匹配不到`show-more-less-html__markup`内容。

**browser方式可用(推荐)**:
1. `browser_navigate` 打开 `https://www.linkedin.com/jobs/view/{JOB_ID}`
2. 页面弹出登录对话框 → `browser_click` 点击 "Dismiss" 按钮关闭
3. 提取JD文本（两种方法）：
   - **方法a**: `browser_snapshot(full=true)` — 对短JD(<5000字)有效，但长JD会被截断到~8000字符
   - **方法b(推荐)**: `browser_console(expression)` 执行JS提取 — 不受截断限制，可分段读取：
     ```javascript
     // 第一段(0-5000字)
     document.querySelector('.show-more-less-html__markup')?.innerText?.substring(0, 5000)
     // 第二段(5000-10000字，如果JD很长)
     document.querySelector('.show-more-less-html__markup')?.innerText?.substring(5000, 10000)
     ```
     如果返回`null`，可能是页面还没加载完或登录弹窗未关闭。
4. 从提取文本中识别JD内容（岗位要求通常在"Qualifications"、"Requirements"、"About you"等标题下）

优点：无需登录，可获取完整JD文本。
缺点：每次只能打开一个页面，批量抓取较慢。**禁止并行(见pitfall #25)**。

## 方法B: JobsDB JD抓取 ❌ 被Cloudflare拦截(2026-06-23实测)

`browser_navigate`访问JobsDB岗位页触发Cloudflare"Just a moment..."安全验证，无法通过。不只是HTML提取不可靠，而是完全无法访问页面。

**替代方案**:
- JobsDB推荐邮件中的岗位摘要做L2粗筛
- 用户手动在浏览器中打开JD后截图/复制
- JobsDB API摘要(如有)做关键词匹配

## 方法C: JobsDB API摘要 ✅ 可用(仅摘要)

```python
url = f"https://hk.jobsdb.com/api/jobsearch/v5/search?siteKey=HK-Main&keywords={urllib.parse.quote(keyword)}&pageSize=20&sortMode=ListedDate&jobType=fulltime&page=1&include=seodata"
```

返回字段: `title`, `companyName`, `id`, `jobUrl`, `salary`(常空), `location`(常空)
注意: `totalCount`可信，单条摘要不足以做L3全文评估。

## 批量抓取模式

**LinkedIn (browser方式)**:
逐个`browser_navigate` → `browser_click`(关闭弹窗) → `browser_snapshot(full=true)`。较慢但可靠。
每次提取后将JD文本存入变量，最后统一评估。

**JobsDB (API方式)**:
用 `execute_code` 批量抓取摘要，间隔1秒避免限流：
```python
for job_id, company in jobs_to_fetch:
    url = f"https://www.linkedin.com/jobs/view/{job_id}"
    req = urllib.request.Request(url, headers=headers)
    with urllib.request.urlopen(req, timeout=15) as resp:
        html = resp.read().decode("utf-8", errors="ignore")
    jd_match = re.search(r'<div class="show-more-less-html__markup[^"]*">(.*?)</div>', html, re.DOTALL)
    # ... parse and store
    time.sleep(1)
```

## Pitfall
- LinkedIn Guest API 2026-06起不可用，返回空结果
- **LinkedIn curl抓取2026-06-22起失效**，返回登录墙，必须用browser方式
- **JobsDB 2026-06-23起被Cloudflare完全拦截**，browser_navigate显示验证页面无法通过
- browser方式抓取LinkedIn时，第一步是关闭登录弹窗(Dismiss按钮)，否则看不到JD内容
- **禁止用子Agent并行抓取LinkedIn(2026-06-23教训)** — 3个子Agent全部触发HTTP 429。必须串行抓取，每次间隔≥3秒
- `browser_snapshot(full=true)`对长JD会被截断(~8000字符)，用`browser_console`的JS提取更可靠
- 抓到的JD可能含HTML实体(`&amp;`等)，需`html.unescape()`处理
