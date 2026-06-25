# 岗位猎手 Agent 记忆

## 服务对象

[YOUR_NAME]（[YOUR_NICKNAME]），12年 SOE 管理 → AI 转型，MSc Applied AI，IANG 签证持有者。

## 核心数据源

- 投递追踪表: `[KB_DIR]/job-search/投递追踪.md`
- JD库: `[KB_DIR]/job-search/JD库/`
- 简历目录: `[RESUME_DIR]/`
- 主简历: `[YOUR_NAME]的简历_2026_V3-内推版.docx`
- 输出目录: `[KB_DIR]/raw/notes/jobs/`

## CDP 配置

- Chrome 启动: `--remote-debugging-port=9222 --user-data-dir=[CHROME_USER_DATA_DIR]`
- config.yaml: `browser.cdp_url: 'http://localhost:9222'`
- WSL2 需 `.wslconfig` 配 `networkingMode=mirrored`
- 验证: `curl -s http://localhost:9222/json/version`

## 已验证平台登录态（2026-06-25）

- LinkedIn ✅ | JobsDB ✅ | Indeed ✅ | Glassdoor ✅
- CTgoodJobs ✅ | 猎聘 ✅ | 劳工处（用户自登录）

## 技术栈

- Gmail MCP: `mcp_gmail_search_emails` / `mcp_gmail_read_email` / `mcp_gmail_batch_modify_emails`
- Hermes browser 工具: CDP 模式操作 6 个平台
- himalaya CLI: 备用邮件扫描

## 搜索配置

- 甜蜜区3方向: BD/AI落地推行/数字化转型
- 排除: PM硬门槛/特定行业/纯销售
- 薪资范围: HKD 25K-40K/月（用户目标）
- 地域: 香港
