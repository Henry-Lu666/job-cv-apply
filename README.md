# job-cv-apply

Hermes Agent 求职辅助 Skill — 覆盖从岗位发现到投递记录的全链路。

> ⚠️ **使用前请先 fork 并修改**：本 Skill 包含作者真实求职方法论，使用前请：
> 1. 修改 `SKILL.md` 中的简历路径（替换 `~/你的简历路径/`）
> 2. 配置 `~/.hermes/scripts/` 下的脚本（见下方「脚本依赖」）
> 3. 创建你自己的投递追踪表和 JD 库（可参考 `examples/`）

## 功能

- **岗位发现**：Gmail 邮箱扫描 + LinkedIn/Indeed 自动搜索
- **JD 抓取**：LinkedIn/JobsDB 自动抓取，保存为英文+中文翻译
- **匹配度评估**：9 维度结构化核实（行业/职能/年限/资质/语言/技术/层级/签证/区域）
- **简历定制**：浅层（求职目标+个人优势）/ 深层（+AI项目+技术栈+工作经历）
- **Cover Letter**：中英文双版本，4 个 bullet 直接映射 JD 要求
- **投递追踪**：自动记录已投岗位，追踪面试/拒绝/通知
- **每日自动化**：Cron 定时扫描 + 自动 JD 匹配

## 安装

```bash
# 复制到 Hermes skills 目录
cp -r job-cv-apply ~/.hermes/skills/productivity/

# 或通过 Hermes CLI 安装
hermes skill install job-cv-apply
```

## 配置

### 1. 简历路径

修改 `SKILL.md` 中的简历路径：

```markdown
主简历 (V3 最新): ~/你的简历路径/你的简历.docx
主简历 PDF:        ~/你的简历路径/你的简历.pdf
```

### 2. 脚本路径

将 `~/.hermes/scripts/` 下的脚本复制到你的 Hermes scripts 目录：

- `gmail_job_scanner.py` - Gmail 邮箱岗位扫描
- `hunter.py` - LinkedIn/Indeed 岗位搜索
- `job_feedback_scanner.py` - 求职反馈邮箱检查
- `apply_handler.py` - 投递记录处理

### 3. Cron 任务

```bash
# 邮箱岗位日报（每天 10:00）
hermes cron create --name "邮箱岗位日报" --schedule "0 10 * * *" --script gmail_job_scanner.py --args "--days 2 --output text --save"

# Hunter 岗位搜索（周一到周五 10:05）
hermes cron create --name "hunter-daily" --schedule "5 10 * * 1-5" --script hunter.py

# 求职反馈邮箱检查（每 60 分钟）
hermes cron create --name "求职反馈邮箱检查" --schedule "every 60m" --script job_feedback_scanner.py --no-agent
```

## 使用

### 每日求职流程

```
用户：今天继续求职
Agent：执行场景12
  → 读取邮箱扫描结果
  → 读取 Hunter 搜索结果
  → 汇总当日新增高匹配岗位
  → 自动抓取 JD + 匹配度评估
  → 呈现结果，等用户决策
```

### 批量 JD 核实

```
用户：帮我核实这些岗位
Agent：执行场景11
  → 关键词初筛
  → 9 维度结构化核实
  → 分类决策（≥65% 可投 / 55-64% 观望 / <55% 不投）
```

### 生成定制简历

```
用户：出 P1 简历
Agent：执行场景13
  → 读取追踪表 P1 section
  → 检查 JD 库
  → 重评匹配度
  → 按方向归类
  → 生成定制简历（浅层/深层）
  → 更新追踪表状态
```

## 核心框架

### 匹配度校准公式

```
Hunter 75%档位 × 0.3 ≈ 真实匹配度（缩水53pp）
Hunter 60-68%档位 × 0.6 ≈ 真实匹配度（缩水20pp）
邮箱★★★★★强推 × 0.5 ≈ 真实匹配度（50%命中率）
```

### 9 维度核实框架

| 维度 | 说明 | 权重 |
|------|------|------|
| 1. 行业经验要求 | 是否要求特定行业经验 | 决定性 |
| 2. 核心职能要求 | 是否匹配核心职责 | 决定性 |
| 3. 硬性年限要求 | 是否满足年限门槛 | 辅助 |
| 4. 专业资质要求 | 是否需要证书/牌照 | 辅助 |
| 5. 语言硬性要求 | 是否要求特定语言 | 辅助 |
| 6. 技术栈要求 | 是否需要特定技术 | 辅助 |
| 7. 管理层级匹配 | 是否匹配管理层级 | 辅助 |
| 8. 签证/永居要求 | 是否需要永居 | 辅助 |
| 9. 目标区域市场经验 | 是否要求特定区域经验 | 决定性 |

### 核实层级

| 层级 | 标记 | 含义 | 可否直接投 |
|------|------|------|-----------|
| L1 关键词粗筛 | `[L1]` | 自动化 scanner 输出 | ❌ 需先走 L2 |
| L2 JD 摘要匹配 | `[L2]` | 基于 teaser/bullets | ⚠️ 可投但建议看全文 |
| L3 JD 全文核实 | `[L3]` | 9 维度逐条对照 | ✅ 可直接投 |
| L4 用户确认 | `[L4]` | 用户看过 JD 并决策 | ✅ 已确认 |

### 投递追踪表格式

```markdown
| 公司 | 岗位 | 方向 | 匹配度 | 链接 | JD链接 | 材料 | 状态 |
```

### JD 保存格式

```markdown
# {公司} - {岗位}

{英文JD原文}

---

## 中文翻译

{中文翻译内容}
```

## 甜蜜区定位

| 方向 | 匹配度 | 说明 |
|------|--------|------|
| **BD / 战略合作 / 客户管理** | 60-70% | 12年大客户开发+渠道管理+GTM经验直接匹配 |
| **AI落地推行 / 变革管理 / 业务赋能** | 60-70% | SOP推行+区域协调+推动业务部门采用新技术 |
| **AI创新管理（中资驻港）** | 55-65% | 央企/国企驻港的创新岗，SOE背景是独特优势 |
| **AI方案销售/BD（技术型）** | 55-65% | AI背景+BD能力形成差异化 |

## 海投阶段规则

| 匹配度 | 行动 | 简历 | CL |
|--------|------|------|-----|
| ≥70% | 直接投 | 定制版 | 专属 CL |
| 60-69% | 投 | 定制版(可浅层) | 通用 CL 或专属 |
| 55-59% | 观望 | 不出 | 不出 |
| <55% | 不投 | — | — |

## 人工确认红线

- 简历发送前必须等用户确认
- 投递提交前必须等用户确认
- 网站内容发布前必须等用户确认
- 视频发布前必须等用户确认

## 目录结构

```
job-cv-apply/
├── SKILL.md                    # 主文件（工作流定义）
├── README.md                   # 本文件
├── LICENSE                     # MIT
├── examples/                   # 脱敏样例
│   ├── sample-JD.md
│   ├── sample-投递追踪.md
│   └── sample-cover-letter.md
└── references/                 # 参考文档
    ├── ats-keyword-optimization.md
    ├── cover-letter-template.md
    ├── gmail-job-scanner.md
    ├── hk-job-channels.md
    ├── jd-fetching-methods.md
    ├── jobsdb-api-workflow.md
    ├── linkedin-jd-extraction.md
    ├── multi-version-resume-workflow.md
    └── ...
```

## 脚本依赖

本 Skill 依赖以下脚本（需自行创建或从其他来源获取）：

| 脚本 | 功能 | 依赖 |
|------|------|------|
| `gmail_job_scanner.py` | Gmail 邮箱岗位扫描 | himalaya (IMAP CLI) |
| `hunter.py` | LinkedIn/Indeed 岗位搜索 | JobSpy |
| `job_feedback_scanner.py` | 求职反馈邮箱检查 | himalaya |
| `apply_handler.py` | 投递记录处理 | — |

**脚本接口**（供参考）：

```bash
# 邮箱扫描
python3 ~/.hermes/scripts/gmail_job_scanner.py --days 7 --output text --save

# 岗位搜索
python3 ~/.hermes/scripts/hunter.py

# 反馈检查
python3 ~/.hermes/scripts/job_feedback_scanner.py

# 投递记录
python3 ~/.hermes/scripts/apply_handler.py "已投 XXX 公司 XXX 岗位 LinkedIn"
```

**脚本存放路径**：`~/.hermes/scripts/`（可通过 `hermes config set scripts_dir` 修改）

## 示例文件

| 文件 | 说明 |
|------|------|
| [examples/sample-JD.md](examples/sample-JD.md) | 脱敏后的 JD 样例（英文+中文） |
| [examples/sample-投递追踪.md](examples/sample-投递追踪.md) | 脱敏后的投递追踪表样例 |
| [examples/sample-cover-letter.md](examples/sample-cover-letter.md) | 脱敏后的 Cover Letter 样例 |

## 依赖

- Hermes Agent
- python-docx（简历处理）
- himalaya（Gmail IMAP）
- curl（JD 抓取）

## 许可证

MIT License — 见 [LICENSE](LICENSE)

## 贡献

欢迎提交 Issue 和 Pull Request！

## 作者

海泽 (Henry)
- GitHub: [Henry-Lu666](https://github.com/Henry-Lu666)
- LinkedIn: [your-linkedin-id](https://www.linkedin.com/in/your-linkedin-id-your-linkedin-id)

## 致谢

- Hermes Agent 团队
- 某高校 MSc Applied AI 项目

## 免责声明

- 本 Skill 提供的是求职方法论框架，不保证求职成功
- 匹配度评估基于作者个人经验校准，你的实际情况可能不同
- 使用前请根据自己的背景调整甜蜜区定位和匹配度阈值
- 作者不对因使用本 Skill 导致的任何损失负责

## v2.0 新增：质量管控体系

### 质量门（Quality Gates）
四级验收标准，每阶段必须通过才能进入下一阶段：
- **L1 扫描合规**：渠道覆盖、结果标注、追踪表查重
- **L2 JD核实质量**：完整JD抓取、9维度评估（核心三维+综合分）
- **L3 简历交付质量**：ATS关键词合规、双语同步、标准描述引用
- **L4 流程完整性**：双文件同步、状态真实性、P1续挖

### 结构化对话（Structured Conversation）
- "走一遍"模式：直接执行（甜蜜区定位表已定义）
- 简历定制：必须确认JD/侧重/Cover Letter
- 面试准备：必须确认轮次/形式/面试官/时间
- 薪资谈判：必须确认阶段/竞争offer/底线

### 输出约束（Output Constraints）
- 简历定制：5项必含 + 4项禁止
- 匹配评估：标准格式（核心三维+综合分+JD引用）
- 投递追踪：双文件同步 + 状态真实性

### 输出管线（Output Pipeline）
```
Stage 1 扫描(L1) → Stage 2 核实(L2) → Stage 3 定制(L3)
→ Stage 4 追踪(L4) → Stage 5 汇总交付
```
"走一遍"模式：全自主执行，只在Stage 5一次性汇报。
