# GitHub 发布工作流 — Skill 发布与脱敏指南

> 最后更新: 2026-06-15
> 用途: 将 Hermes Agent Skill 发布到 GitHub 前的脱敏与自检流程

---

## 发布前必做：四轮脱敏

### 第一轮：路径与联系人（基础）

| 原始值 | 替换为 |
|--------|--------|
| `/mnt/d/个人/个人简历/` | `~/你的简历路径/` |
| `[KB_DIR]/` | `~/你的知识库路径/` |
| `D:\桌面\` / `/mnt/d/桌面/` | `~/job-search/inbox/` |
| 手机号 | `+852 XXXX XXXX` |
| 个人邮箱 | `your.email@example.com` |
| 个人网站 | `your-portfolio.com` |

### 第二轮：身份与教育（进阶）

| 原始值 | 替换为 |
|--------|--------|
| 真实中文名（简繁体） | `你的姓名` |
| 英文名 | `Your Name` |
| LinkedIn ID 片段 | `your-linkedin-id` |
| GitHub 用户名 | `your-github-username` |
| 大学全名 | `某211大学` / `某高校` |
| 学校英文名 | 同上 |
| 签证有效期 | `有效工作签证`（去掉具体日期） |

### 第三轮：职业指纹与战例（深度）

| 原始值 | 替换为 |
|--------|--------|
| 公司全名（如[PREVIOUS_COMPANY]） | `某大型汽车制造企业` |
| 内部代号（如柳州模式） | `某成功案例` |
| 薪资数字 | `HKD XXX,XXX` |
| 具体战例公司名（WPP/MiniMax等） | `某媒体公司` / `某区域总经理岗` |
| 招聘联系人邮箱 | `recruiter@example.com` |
| 家庭信息（孩子年龄等） | `有年幼子女` |
| Cron Job IDs | `YOUR_CRON_JOB_ID_1` |
| 飞书群 ID | `YOUR_FEISHU_CHAT_ID` |

### 第四轮：学术/项目指纹（2026-06-23 补充）

> 论文细节、项目代号、特定技术指标可通过学术搜索引擎反向定位到个人。

| 原始值 | 替换为 |
|--------|--------|
| 预训练模型具体名称（如RoBERTa-wwm-ext） | `预训练模型` |
| F1 具体数值（如 0.21→0.67） | `F1大幅提升` |
| 具体任务名称（如ABSA/dual-task） | `NLP双任务` / `业务分析系统` |
| 消融实验具体配置 | `消融实验验证` |
| 项目合作方名称（如中银/某银行） | `某银行` / `某金融机构` |
| 居住地精确到区（如湾仔） | `香港` |
| 配偶/家庭具体信息 | 删除或泛化 |

**为什么单独一轮**：前三轮替换的是"明显个人信息"，但论文+具体F1数值的组合在Google Scholar上几乎是唯一结果，可直接定位到学校→姓名→LinkedIn。

### 批量替换技术（2026-06-23 补充）

当替换项 > 15 个时，用 `find + sed -i` 批量执行比逐个 `patch` 高效：

```bash
find . -name "*.md" -not -path "./.git/*" -exec sed -i \
  -e 's/原始值1/替换值1/g' \
  -e 's/原始值2/替换值2/g' \
  -e 's/原始值3/替换值3/g' \
  {} \;
```

**⚠️ 顺序重要**：先替换长模式（如 `RoBERTa-wwm-ext dual-task four-head ABSA`），再替换短模式（如 `ABSA`），否则短替换会破坏长模式的一部分。

**验证**：替换后必须重新扫描，因为 sed 的正则可能遗漏边界情况（如中文标点、Windows反斜杠路径）。

---

## 自检命令

发布前用以下命令全面扫描残留 PII：

```bash
cd <skill-directory>

# 一次性全量扫描（覆盖所有已知 PII 模式）
grep -rn -i \
  '真实中文名\\|繁体中文名\\|英文名\\|邮箱前缀\\|@.*\\.com' \
  --include='*.md' | grep -v '.git/'

# 必须逐一检查的模式（按风险等级）
grep -rn 'D:\\\\桌面' --include='*.md'       # Windows 桌面路径
grep -rn '/mnt/d/' --include='*.md'          # WSL 挂载路径
grep -rn '+852' --include='*.md'             # 香港手机号
grep -rn '+86' --include='*.md'              # 内地手机号
grep -rn 'IANG.*20[0-9][0-9]' --include='*.md'  # 签证+具体年份
grep -rn 'HKD.*[0-9][0-9][0-9],[0-9][0-9][0-9]' --include='*.md'  # 具体薪资
grep -rn '大学全名\\|英文校名' --include='*.md'  # 学校
grep -rn '公司代号\\|内部案例名' --include='*.md'  # 战例
```

**通过标准**：上述命令输出为空。

### 完整 PII 模式清单（2026-06-15 实战校验）

以下是实测中被用户全文检索抓出的模式，**必须全部覆盖**：

| 类别 | 具体模式 | 替换为 |
|------|---------|--------|
| 真实姓名 | 中文（简繁体）、拼音、英文名 | `你的姓名` / `Your Name` |
| 联系方式 | 手机（+852/+86）、邮箱、个人网站 | `+852 XXXX XXXX` / `your.email@example.com` |
| 第三方联系人 | 招聘联系人邮箱（如 recruiter@company.com） | `recruiter@example.com` |
| 教育背景 | 大学全名（中英文）、学院名 | `某211大学` / `某高校` |
| LinkedIn | ID 片段、编码后的 URL | `your-linkedin-id` |
| 签证信息 | IANG + 具体到期日期 | `有效工作签证` |
| 薪资数字 | HKD + 具体金额范围 | `HKD XXX,XXX` |
| 家庭信息 | 孩子年龄、家庭状况 | `有年幼子女` |
| 文件路径 | `/mnt/d/`、`D:\桌面\` | `~/job-search/inbox/` |
| 知识库路径 | `my-knowledge-base`、`08-找工作` | `job-search` |
| 战例公司名 | 投递过的具体公司（WPP/MiniMax/Manulife等） | `某媒体公司` / `某保险公司` |
| 内部代号 | 柳州模式、SOGMW 等 | `某成功案例` / `汽车行业经验` |
| Cron IDs | job_id 字符串 | `YOUR_CRON_JOB_ID_1` |
| 平台 ID | 飞书群 ID、Telegram chat ID | `YOUR_CHAT_ID` |
| GitHub 用户名 | GitHub handle | `your-github-username` |

---

## 两个脱敏级别

| 级别 | 场景 | 要求 |
|------|------|------|
| **熟人分享** | 同学、朋友、HA 用户 | 基础脱敏即可（路径+手机+邮箱） |
| **公开推广** | 陌生人、论坛、技能市场 | 四轮全做，确保无法还原到本人 |

---

## 发布流程

```bash
# 1. 初始化
cd <skill-directory>
git init

# 2. 脱敏（四轮）
# ... 执行替换 ...

# 3. 自检
grep -rn -i '<所有敏感模式>' --include='*.md' | grep -v '.git/'
# 必须输出为空

# 4. 创建必要文件
# LICENSE (MIT)
# .gitignore (排除 *.docx, *.pdf, 个人信息文件)
# README.md (使用说明 + 脚本依赖 + 免责声明)
# examples/ (脱敏后的样例文件)

# 5. 提交
git add .
git commit -m "Initial commit: <skill-name>"

# 6. 创建 GitHub 仓库并推送
gh repo create <skill-name> --public --source=. --remote=origin --push --description "..."

# 7. 添加 Topics
gh repo edit <owner>/<skill-name> --add-topic hermes-agent --add-topic <relevant-topics>

# 8. 验证
gh repo view <owner>/<skill-name>
```

---

## .gitignore 模板

```gitignore
# 敏感文件
*.docx
*.pdf
*.doc

# 个人信息文件
投递追踪.md
岗位匹配总报告.md
JD库/

# 系统文件
.DS_Store
Thumbs.db

# 编辑器
.vscode/
.idea/
```

---

## Pitfall: 不要只做第一轮就声称"已脱敏"

**教训（2026-06-15）**：第一次只替换了路径和手机号，用户 clone 回来全文检索发现：
- 真实中文名（簡繁體）仍在多处
- 学校全名（中英文）仍在
- 第三方招聘联系人邮箱仍在
- 签证有效期、薪资数字、家庭信息仍在
- 战例公司名仍在

**规则**：必须做四轮，第四轮完成后用 grep 全面扫描确认无残留。用户会 clone 仓库做全文检索验收。

## Pitfall: PII 发现后必须 force push 清理历史

**教训（2026-06-15）**：首次 push 后用户 clone 发现残留 PII，修复后正常 push 会产生第二个 commit——但**旧 commit 中的 PII 仍然存在于 Git 历史中**。

**规则**：PII 修复必须用 force push 覆盖历史，确保旧 commit 从远程移除：

```bash
# 修复 PII 后
git add .
git commit --amend -m "Initial public release (sanitized)"
git push --force origin master
```

**⚠️ 注意**：
- force push 会覆盖远程历史，确认没有其他人基于旧 commit 工作
- 若他人已 fork 旧版，其 fork 中可能仍有旧 commit（无法控制）
- GitHub 缓存/搜索引擎有延迟，几小时后在 Commits 页确认只剩 1 条记录
- 理想情况：**首次 push 前就做彻底脱敏**，避免 force push

## Pitfall: 脱敏后本地 skill 与远程仓库会分叉

发布到 GitHub 后，本地 `~/.hermes/skills/xxx/` 和远程仓库会各自独立演化：
- 本地：你日常使用，会继续添加个人信息、更新工作流
- 远程：公开版本，保持脱敏状态

**不要**试图让两者保持同步——它们本质上是同一 skill 的两个分身。
如果需要更新远程版本，手动 cherry-pick 或重新脱敏后 force push。
