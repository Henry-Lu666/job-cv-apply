# Chrome CDP 接管配置指南

> 2026-06-25 验证。让 Hermes browser 工具接管用户已登录的 Chrome，保留所有 cookie/登录态。
> 解决 LinkedIn/JobsDB 登录墙、JobsDB Cloudflare 拦截等问题。

---

## 前置条件

- Windows 11 + WSL2（networkingMode=mirrored）
- Chrome 浏览器已安装

## 启动步骤

### 1. 关闭所有 Chrome 窗口

### 2. Win+R 或 PowerShell 执行：
```
"C:\Program Files\Google\Chrome\Application\chrome.exe" --remote-debugging-port=9222 --user-data-dir=[CHROME_USER_DATA_DIR]
```

**⚠️ 必须加 `--user-data-dir`** — 不加的话 Chrome 用默认 profile（临时目录），登录态不保留。`[CHROME_USER_DATA_DIR]` 是专用 profile，登录态持久化。

**⚠️ 不要在浏览器地址栏输入这个命令** — 这是终端命令，不是 URL。用户曾误在地址栏输入导致 DNS_PROBE_FINISHED_NXDOMAIN。

### 3. 在 Chrome 中登录所有求职平台

首次启动后，手动登录以下平台（后续重启 Chrome 不需要重新登录）：
- LinkedIn
- JobsDB
- Glassdoor（与 Indeed 共享登录）
- 獵聘
- CTgoodJobs（需先注册）
- Indeed
- 劳工处（可选，搜索不需要登录）

### 4. 验证 CDP 连通

```bash
curl -s http://localhost:9222/json/version
```

预期返回 Chrome 版本和 WebSocket URL。

### 5. Hermes 连接

在 config.yaml 中设置：
```yaml
browser:
  cdp_url: 'http://localhost:9222'
```

之后 `browser_navigate` 自动连接到该 Chrome 实例。

---

## 已验证的登录状态（2026-06-25）

| 平台 | 登录身份 | 验证方法 |
|------|---------|---------|
| LinkedIn | [YOUR_NAME] | Feed 页标题 "信息流 \| LinkedIn"，侧边栏显示个人资料 |
| JobsDB | [YOUR_NAME] | Dashboard 页显示 "22 searches in last 90 days" |
| Glassdoor | AI Engineer | Community 页，显示 "Post anonymously as AI Engineer" |
| 獵聘 | — | "我的首页"，显示"0 投递/0 收藏" |
| CTgoodJobs | Henry | 导航栏显示 "Henry" 而非 "Login" |
| Indeed | Hongzhang | 首页显示 "Hongzhang，歡迎" + 个性化推荐 |
| 劳工处 | [USERNAME] | 用户自行登录（搜索免费不需要登录） |

---

## WSL2 网络配置

WSL2 使用 `networkingMode=mirrored`（在 `.wslconfig` 中配置）时，`localhost:9222` 直通 Windows Chrome。无需额外端口转发。

验证连通性：
```bash
curl -s --connect-timeout 3 http://localhost:9222/json/version
```

---

## 常见问题

### Chrome 启动后 CDP 不通
- 检查是否关闭了所有 Chrome 窗口后再启动（Chrome 单实例，已有进程会忽略新参数）
- 检查端口占用：`netstat -ano | findstr 9222`

### browser_navigate 超时
- Chrome 可能已关闭或 CDP 端口断开，重新启动 Chrome
- config.yaml 中 `cdp_url` 是否已设置

### 登录态丢失
- 检查是否用了 `--user-data-dir` — 不加则用临时目录
- 不要用 `--incognito` — 无痕模式不保留 cookie

### browser 工具仍用 Browserbase 而非本地 Chrome
- 检查 `browser.engine` 配置 — 设为 `auto` 时会自动检测 CDP
- 确认 `cdp_url` 已设置为 `http://localhost:9222`

---

## Pitfall

1. **🔴 不要替用户输入密码** — 用户分享的凭据仅供自行登录，agent 不在 browser 工具中操作敏感凭据
2. **Chrome 重启后登录态保留** — 只要 `--user-data-dir` 指向同一目录，cookie 持久化
3. **Indeed+Glassdoor 共享登录** — Indeed 收购了 Glassdoor，登录一个自动登录另一个
4. **CTgoodJobs 需先注册** — 未注册只能看标题，看不到 JD
