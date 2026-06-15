# GitHub 发布 Skill 工作流

> 将 Hermes Agent skill 发布到 GitHub 的标准流程（脱敏 → 初始化 → 推送）

## 触发词

"放到 GitHub" / "开源这个 skill" / "push 到 GitHub"

## 流程

### Step 1: 扫描敏感信息

```bash
grep -r -i '用户名\|邮箱\|手机号\|/home/\|/mnt/' ~/.hermes/skills/<skill-name>/ --include='*.md'
```

常见敏感项：
- 个人简历路径（如 `~/Documents/resume/` 或 WSL 挂载路径）
- 知识库路径（如 `~/your-knowledge-base/`）
- 手机号/邮箱/个人网站
- LinkedIn ID / GitHub 用户名
- Cron job IDs
- 飞书群 ID

### Step 2: 批量脱敏

用 `execute_code` 批量替换：

```python
replacements = [
    ("~/Documents/resume/", "~/你的简历路径/"),
    ("~/your-knowledge-base/", "~/你的知识库路径/"),
    ("真实姓名", "你的姓名"),
    ("username", "your-username"),
    ("email@example.com", "your.email@example.com"),
    ("+852 XXXX XXXX", "+852 XXXX XXXX"),
    ("cron-job-id", "YOUR_CRON_JOB_ID"),
]
```

**注意**：先处理 SKILL.md，再处理 references/ 目录。最后 grep 验证无残留。

### Step 3: 创建 README.md

包含：
- 功能概述
- 安装方法
- 配置说明（简历路径、脚本路径、Cron 任务）
- 使用示例（触发词 + 预期行为）
- 核心框架（匹配度公式、核实维度等）
- 目录结构
- 依赖项

### Step 4: 创建 .gitignore

排除：
- `*.sanitized`（临时脱敏文件）
- `*.docx`, `*.pdf`（个人简历）
- `投递追踪.md`, `岗位匹配总报告.md`（含个人信息）
- `JD库/`（含个人信息）
- `.env`, `*.key`

### Step 5: Git 初始化 + 推送

```bash
cd ~/.hermes/skills/<skill-name>
git init
git add .
git commit -m "Initial commit: <skill-name>"
gh repo create <skill-name> --public --source=. --remote=origin --push --description "描述"
```

## Pitfall: 脱敏不彻底

- 用 `grep -r` 多轮检查，不要只检查一次
- references/ 目录下的文件也需要脱敏（不是只有 SKILL.md）
- 邮件主题示例中可能包含姓名（如"你的名字，立即申请"）
- LinkedIn URL 中包含用户 ID

## Pitfall: .gitignore 排除过多

不要排除 references/ 目录——那是 skill 的核心知识库。
只排除包含个人数据的文件（简历、追踪表、JD 库）。

## Pitfall: Git 历史仍含敏感信息

若第一版 commit 曾推送过未脱敏内容，仅改当前文件不够；GitHub 上仍可通过 History 查看旧版本。

**处理**：脱敏验证通过后，用单 commit 重写历史再推送：

```bash
git checkout --orphan clean
git add -A
git commit -m "Initial public release (sanitized)"
git branch -M master
git push -f origin master
```

推送前务必 `git log` 与 `git show` 抽查旧 commit 是否已不可达。