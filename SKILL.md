---
name: job-cv-apply
description: 求职全链路自动化 — 每日Cron扫描(Gmail+Hunter) → 自动抓JD(英文+中文) → 9维度匹配评估 → 简历定制(浅层/深层) → Cover Letter → 投递追踪。覆盖场景1-13。
trigger: 今天继续求职 | 求职日常 | 走一遍固定动作 | 简历 | CV | 帮我改简历 | 已投 | 投了 | applied | 出求职邮件 | 写cover letter | 扫邮箱 | 看看邮件里的岗位 | 核实强推 | 批量抓JD | 做一遍匹配 | 去做2 | 出P1简历 | 把待出简历的搞定 | 记个灵感 | /idea | 查看投递 | 投递记录 | 把V3更新进去 | 合并到XX文档
---

# 求职辅助 Agent

## ⚠️ 前置红线：必须先读简历 + ATS 关键词检查

在对任何岗位/邮件/建议做出分析**之前**，必须先读取个人简历：

```
主简历 (V3 最新): ~/你的简历路径/你的简历.docx
主简历 PDF:        ~/你的简历路径/你的简历.pdf
压缩版 PDF:        ~/你的简历路径/你的简历_under500kb.pdf
旧版 V2 (参考):    ~/你的简历路径/你的简历_V2.docx
原始版 (参考):     ~/你的简历路径/你的简历_原始.docx
2023旧版 V3:       ~/你的简历路径/你的简历_旧版V3.pdf  ← 内容更详细，适合挖细节
2023旧版 V4:       ~/你的简历路径/你的简历_旧版V4.docx
2023旧版 繁体:     ~/你的简历路径/你的简历_繁体.docx  ← 153KB，经历最详细（含「某成功案例」全过程、经销商管理等），用表格存储（doc.tables[0]）
2023旧版 求职信:   ~/你的简历路径/求职信.pdf
```

**读取方式**：
- PDF 用 `read_file` 或 OCR 工具
- .docx 用 `python-docx`（`from docx import Document; doc = Document(path)`），pandoc 在 WSL 中通常未安装
- 对定制版简历（如 V3-公司A-方向A.docx），直接读 .docx 源文件比 PDF 更准确（无格式损失）

**V3 与 V2 的关键差异**：见 `references/resume-v2-structure.md`（待更新为V3）

**求职平台资料填写**：见 `references/jobsdb-profile-setup.md`（JobsDB完整填写指南+用户偏好技能列表）、`references/offertoday-platform.md`（OfferToday平台信息+推荐质量评估）

**🔴 ATS 关键词红线**（详见 `references/ats-keyword-optimization.md`）：
简历中绝对不能出现这些词 — 否则 ATS 会将你误判为 HR/L&D 方向：
- 人才发展 / Talent Development / 慕课 / 培训 / 内训师 / e-learning / trainer / L&D
- 替代为：营运 SOP / 业务赋能 / 业务落地宣导 / enablement / rollout

**为什么**：
- 用户的实际能力组合（某大型汽车制造企业 12年大厂 + AI硕士 + 扭亏实战）远超表面印象
- 不做这个预读，分析就会偏——比如把 Ta 当"普通营销老兵"对待，实际上是"新能源汽车全链条操盘+AI落地的复合人才"
- 之前犯过这个错误：没读简历就给了理论化分析，被用户纠正

**简历核心画像摘要（快速参考）**：
- 12年某大型汽车制造企业（国企2万+员工）
- 高压工程师→用户运营（15000+反馈）→650人团队管理→扭亏为盈→AI 硕士
- 论文：RoBERTa-wwm-ext dual-task ABSA，F1 0.21→0.67
- 本科学的车辆工程，211/双一流
- 粤语：基础听；英语：工作读写+商务沟通；普通话：二级甲等

**🎯 甜蜜区定位（2026-06-11 更新）**：

| 方向 | 匹配度 | 说明 |
|------|--------|------|
| **BD / 战略合作 / 客户管理** | **60-70%** ← 主力 | 12年大客户开发+渠道管理+GTM经验直接匹配，BD岗对行业经验容忍度最高 |
| **AI落地推行 / 变革管理 / 业务赋能** | **60-70%** ← 主力 | SOP推行+区域协调+推动业务部门采用新技术=AI Adoption/Change Enablement岗的核心要求。实证：某保险科技岗 Manager Asia 8维度评分7.5/10 |
| **AI创新管理（中资驻港）** | **55-65%** ← 副线 | 央企/国企驻港的创新岗（如中银Innovation Manager），SOE背景是独特优势，金融科技gap可弥补 |
| **AI方案销售/BD（技术型）** | **55-65%** ← 副线 | 如LexisNexis Market & BD Director，AI背景+BD能力形成差异化，行业gap存在但AI是加分项 |
| 低门槛AI PM（传统PM路线图） | **45-55%** ← 观望 | 仅限要求"AI exposure"而非"3-5yr PM experience"的岗位（如HKT级） |
| Head of AI / AI Director | **25-35%** ← 暂停 | 要求10-15年AI领导经验，1.5年硕士不够 |
| 需特定行业经验的岗 | **30-45%** ← 暂停 | 保险/奢品/航空/加密货币/媒体，行业gap无法弥补 |

**⚠️ 关键教训（2026-06-11 更新）**：
1. AI PM方向不能一刀切砍掉。区分两类：
   - **传统AI PM**（要3-5年PM经验+ship过产品）→ 确实不匹配，投5个废3个
   - **AI落地推行/变革管理**（要把AI战略变成行动、推行SOP、协调区域）→ 完美匹配你的业务赋能经验
2. **判断标准**：JD核心是"管理产品路线图"还是"推动AI落地执行"？前者需要PM经验，后者需要执行力+协调力+变革管理——这正是你的强项
3. **中资驻港创新岗**不要忽略。央企背景+AI知识+管理经验的组合，在BOC/华润/招商局等的创新部门有独特竞争力
4. BD岗的容错率依然最高，但AI变革管理岗的匹配度可能比BD还高（Manulife 7.5 > 多数BD岗）

## 海投阶段规则（2026-06-13 生效）

当用户宣布进入"海投阶段"时，匹配度阈值从70%降至60%，核心逻辑从"精准投递"变为"广覆盖快投递"：

| 匹配度 | 行动 | 简历 | CL |
|--------|------|------|-----|
| ≥70% | 直接投 | 定制版 | 专属CL |
| 60-69% | 投 | 定制版(可浅层) | 通用CL或专属 |
| 55-59% | 观望 | 不出 | 不出 |
| <55% | 不投 | — | — |

**海投阶段行为变化**：
- 用户**不再逐一确认**投递决策，批量推进即可
- 优先复用已有定制简历 > 同方向浅层替换 > 跨方向深层替换
- CL可用通用模板，不必每份单独定制（除非已有定制简历+JD特别匹配）
- 同一岗位24小时内不重复投
- **但追踪表仍需每次更新**（不可省略记录）

**切换回精准模式**：当用户说"不海投了"/"回到精准"/"一个个来"时，恢复70%阈值+逐岗确认。

---

## 求职自动化 Cron 配置（2026-06-15 文档化）

### 现有 Cron 任务

| job_id | 名称 | 频率 | 脚本 | deliver | 用途 |
|--------|------|------|------|---------|------|
| `YOUR_CRON_JOB_ID_1` | 邮箱岗位日报 | `0 10 * * *` (每天10:00) | `gmail_job_scanner.py --days 2 --output text --save` | local | 扫描Gmail求职邮件+JobsDB API搜索 |
| `YOUR_CRON_JOB_ID_2` | hunter-daily | `5 10 * * 1-5` (周一到周五10:05) | `hunter.py` | local | LinkedIn/Indeed岗位搜索 |
| `YOUR_CRON_JOB_ID_3` | 求职反馈邮箱检查 | `every 60m` (每60分钟) | `job_feedback_scanner.py` | origin | 追踪已投岗位的后续响应（面试/拒绝/通知） |

### Cron 配置说明

**邮箱岗位日报**（job_id: `YOUR_CRON_JOB_ID_1`）：
- 脚本路径：`~/.hermes/scripts/gmail_job_scanner.py`
- 输出：`~/你的知识库路径/raw/notes/jobs/YYYY-MM-DD-email-jobs.md`
- 评分逻辑：关键词匹配 → ★★★★★强推 / ★★★★值得投 / ★★★可考虑 / ★★观望 / ★低匹配
- **⚠️ 评分是关键词匹配，不是人岗匹配**，必须走JD核实

**hunter-daily**（job_id: `YOUR_CRON_JOB_ID_2`）：
- 脚本路径：`~/.hermes/scripts/hunter.py`
- 输出：`~/你的知识库路径/raw/notes/jobs/YYYY-MM-DD-jobs.md`
- 搜索源：LinkedIn + Indeed
- **⚠️ 匹配度虚高**（实测75%档位核实后仅22%中位数），必须走JD核实

**求职反馈邮箱检查**（job_id: `YOUR_CRON_JOB_ID_3`）：
- 脚本路径：`~/.hermes/scripts/job_feedback_scanner.py`
- 逻辑：读最近2小时INBOX → 过滤求职平台/猎头/HR邮件 → 分类（🟢积极/🔴拒绝/⚪通知）
- **no_agent模式**：不调LLM，脚本stdout直接推送
- 空输出=静默，非空输出=原样推送给用户

### 如何修改 Cron

```bash
# 查看所有cron
hermes cron list

# 暂停某个cron
hermes cron pause <job_id>

# 恢复
hermes cron resume <job_id>

# 修改deliver目标（如推送到飞书群）
hermes cron update <job_id> --deliver "feishu:oc_xxx"

# 修改频率
hermes cron update <job_id> --schedule "0 9 * * *"

# 手动触发一次
hermes cron run <job_id>
```

### Cron 与场景12的关系

- **自动执行**：cron每天10:00+10:05自动运行邮箱扫描+Hunter搜索
- **手动触发**：用户说"今天继续求职"时，主Agent执行场景12（读取cron结果+补充JD匹配）
- **衔接逻辑**：场景12的Step 1-2读取cron产出的报告文件，Step 5自动进入JD匹配

---

## 求职反馈监控（邮箱应用状态追踪）

与`gmail_job_scanner.py`（发现新岗位）不同，求职反馈监控是**追踪已投岗位的后续响应**。

**工具**：`~/.hermes/scripts/job_feedback_scanner.py`
**触发**：cron job "求职反馈邮箱检查"（每60分钟）
**逻辑**：
1. 用himalaya读最近2小时INBOX
2. 过滤求职平台/猎头/公司HR邮件
3. 排除"新职位推荐"类邮件（job alert, recommended, is hiring）
4. 对剩余邮件分类：🟢积极（面试/offer）/ 🔴拒绝 / ⚪通知
5. 有反馈→推通知给用户 + 记录到追踪表；无反馈→静默

**关键区分**：
| 信号 | 邮件内容 | 处理 |
|------|---------|------|
| 新职位推荐 | "is hiring", "job alert", "推荐" | 跳过（scanner处理） |
| 申请状态更新 | "application", "reviewing", "shortlisted" | 记录+通知 |
| 面试邀请 | "interview", "schedule", "availability" | 🔔紧急通知 |
| 拒绝通知 | "unfortunately", "not moving forward" | 记录 |

**cron配置（no_agent模式）**：
```yaml
schedule: every 60m
script: job_feedback_scanner.py
no_agent: true
deliver: origin  # 推送到用户当前对话
```
- `no_agent: true` = 不调LLM，脚本stdout直接推送
- 空输出 = 静默（不打扰用户）
- 非空输出 = 原样推送给用户

---

## 场景1: CV优化（接收JD）

当用户发送以下内容时，执行CV优化：
- 包含"简历"或"CV"或"帮我改简历"
- 或直接发送一段JD文本（包含"岗位职责"/"requirements"/"qualifications"等关键词）

**处理流程：**
1. 读取 `~/你的知识库路径/wiki/people/个人画像.md` 获取用户经历
2. **读取简历PDF** 确认最新版本（强制操作）
3. **挖掘旧版简历**：当V3缺少某类经历的细节时，检查 `~/你的简历路径/` 下的旧版（2023年V2/V3、早期中文简历），旧版可能包含被新版删减但对特定JD高度相关的内容（如PMO统筹、跨部门协调等）。不要只看最新版。
4. 分析JD的核心要求和关键词
5. **输出JD关键词覆盖率分析**（标准格式）：
   - ✅ 已覆盖：简历中有直接对应的关键词/经历
   - ⚠️ 部分覆盖：有相关经历但措辞未对齐JD，需改写
   - ❌ 缺失：简历中完全无对应（标为"硬伤"或"可忽略"）
   - 📊 总结：ATS关键词覆盖率估算 + 获面试概率评估
6. 输出针对该JD的简历微调建议：
   - 哪段经历重点展开
   - 项目描述的措辞调整
   - 技能排序建议
7. 不生成整份简历，只输出"建议修改的段落+修改理由"

### ⛔ Pitfall: 不要帮用户伪造方法论经历

用户问"我做过X，算不算Agile/Scrum/Lean？"时：
- **有框架实践**（用过Jira跑Sprint、有站会/回顾会）→ 可以写"Agile/Scrum experience"
- **有精神无框架**（迭代交付、快速转向、短周期出成果）→ 写"iterative delivery & rapid pivoting"，**不要**写"精通Agile/Scrum"
- **面试被追问框架细节时穿帮**比没写更减分
- 包装原则：用JD关键词描述**真实经历的实质**，而不是给经历贴一个你没有的标签

### 场景1b: 审查已定制简历的JD匹配度

当用户已完成简历定制（如场景10生成的版本），说"看看这份简历合不合适"/"这份简历能投吗"/"帮我review这份简历"时，执行此场景。

**输入**：定制版 .docx 简历 + 目标JD

**处理流程**：
1. 用 python-docx 读取 .docx 内容（pandoc 在 WSL 中可能未安装）
2. 逐条对照JD的核心要求，输出**结构化匹配度评分表**：

```
| JD要求 | 简历覆盖 | 评分 |
|--------|---------|------|
| 要求1 | ✅/⚠️/❌ + 具体说明 | X/10 |
```

3. **重心偏移检查**（新增，2026-06-12）：当 JD 的核心职能与简历当前侧重点不一致时，逐段标注「重心偏移」并给出调整方向。常见模式：

   | JD 类型 | 简历常见偏移 | 调整方向 |
   |---------|-------------|---------|
   | AI落地推行/变革管理 | 过度强调技术细节（模型架构、F1、消融实验） | 弱化技术，强化：治理协调、adoption指标、playbook维护、跨区域推行 |
   | BD/战略合作 | 过度强调管理流程（SOP、团队搭建） | 弱化管理细节，强化：客户开发、收入贡献、pipeline建设、C-suite关系 |
   | 中资创新岗 | 过度强调外企/国际化经历 | 强调：SOE治理经验、央企文化理解、政策合规、内部创新推动 |
   | 区域总经理 | 过度强调单一职能经验 | 强调：P&L、跨职能整合、区域扩张、扭亏为盈 |

   每个偏移点给出：具体段落位置 + 原文摘要 + 建议改写方向（不是完整改写，用户确认后再出对照表）。

4. 分为三个section：
   - **✅ 优势项**（评分≥7/10）：简历中有直接且充分的对应
   - **⚠️ 可弥补项**（评分4-6/10）：有相关经历但措辞/深度未完全对齐
   - **❌ 硬伤项**（评分≤3/10）：简历中完全无对应，且无法通过措辞调整弥补

4. 最后给出**总匹配度百分比估算**和**结论**：
   - ≥70%：建议投递，可做微调
   - 50-69%：有机会但需要强storytelling，说明具体怎么包装
   - 30-49%：匹配度低，建议找更对口的岗位或准备好"为什么跨界是优势"的叙事
   - <30%：不建议投，浪费时间和机会成本

5. **必须给出明确行动建议**（投/不投/有条件投），不要只给分析不给结论

6. **输出「需改 N 处」汇总表**（标准收尾格式）：

   ```
   | # | 位置 | 改动 | 优先级 |
   |---|------|------|--------|
   | 1 | 求职目标 | 3方向→1方向，聚焦JD核心 | 高 |
   | 2 | 个人优势 | 弱化XX，加JD关键词 | 高 |
   | 3 | 技术栈 | 前置JD关心的工具，后移深度学习 | 中 |
   | 4 | 工作经历 | 补governance/metrics/playbook语言 | 高 |
   ```

   然后等用户确认「改哪些」再出完整对照表——不要一次性输出完整改写（用户可能只同意部分改动）。

**风格要求**：
- 表格化，不要大段文字
- 评分用 X/10 量化，不要模糊表述
- 硬伤直接标"硬伤"，不要美化
- 行动建议放在最后，一行结论 + 2-3行理由

**⚠️ Pitfall: 不要因为简历写得好就高估匹配度**
简历的包装质量（措辞精炼、数据充足）和岗位匹配度是两回事。一份写得很好的简历，如果核心经验不对口，匹配度照样低。分开评估"简历质量"和"JD匹配度"。

### 简历版本选择指引

当前有两份活跃简历基础版本 + 多份定制版本：

**基础版本**：

| 版本 | 求职目标 | 适合投 | 不适合投 |
|------|---------|--------|---------|
| V3（通用版） | 运营管理/业务拓展/团队管理 | 传统行业、管理岗、中资企业 | — |
| 涂鸦版（V3基础上） | 海外业务拓展/B2B/科技行业 | 科技公司、IoT、海外业务 | 传统贸易、非科技行业 |

**定制版本**（基于V3，替换求职目标+摘要，2026-06-09生成）：

| 文件名 | 方向侧重 | 适合投 |
|--------|---------|--------|
| V3-WPP-Strategy-Growth | 增长体系搭建+跨区域规模化 | 战略增长/运营优化岗 |
| V3-MiniMax-Regional-GM | 跨区域扩张+扭亏为盈 | 区域总经理/区域运营 |
| V3-Manulife-Brand-Marketing | 消费者洞察+品牌建设 | 品牌营销/市场负责人 |
| V3-Manulife-AI-Adoption-v2 | AI落地推行/变革管理 | AI Adoption/Change Enablement |
| V3-Bupa-Head-Branding | 品牌标准化+消费者体验 | 品牌管理/零售品牌 |
| V3-LexisNexis-BD-AI-v2 | 企业级BD/AI方案销售 | 企业级BD（C-suite导向） |
| V3-Adecco-BD-Tech-v2 | 业务拓展/战略合作/GTM | BD Manager/Director（科技行业） |
| V3-ByteDance-Partnership | 生态合作/战略伙伴管理/GTM | BytePlus Partnership/Ecosystem Manager |
| V3-Wing-AI-PM | AI产品经理/用户增长/Playbook | AI创业公司PM（如Wing Assistant） |
| V3-TradeDesk-BD | 企业级BD/C-suite/战略客户 | AdTech/MarTech BD Director |
| V3-RollingPeople-FMCG | 产品管理/品牌营销/P&L | FMCG/消费品PM |
| V3-AXA-PMO | 运营治理/PMO/战略项目管理 | 保险/金融Ops Head/PMO |
| V3-HashKey-BD-Crypto | B2B客户开发+战略合作 | 商务拓展/战略合作 |

**v2版本与v1版本的关键差异**（2026-06-12验证）：
- v1（06-09生成）：只改了求职目标+个人优势摘要（浅层替换）
- v2（06-12生成）：深层替换——求职目标+个人优势+AI项目bullet+技术栈排序+工作经历bullet（中英文全部重写），并从旧CV（`你的简历_繁体.docx`）提取了更多B2B/经销商管理/跨部门协调细节融入工作经历
- **投递时优先用v2版本**，v1版本保留作参考
- **已验证的批量深层定制**：06-12一次性生成5份（ByteDance/Wing/TradeDesk/RollingPeople/AXA），全部基于同一模板的不同段落替换，输出文件大小稳定在539-541KB

**规则**：
- 默认用V3通用版
- 只有当JD明确涉及科技/IoT/B2B/海外业务时，才用涂鸦版方向的调整
- 如果有匹配的定制版本，优先用定制版
- 每次新投递，根据JD决定用哪个版本——不要用涂鸦版投传统行业（求职目标写着"科技行业"给贸易公司看，对方会觉得找错门了）
- 新方向需要新定制版时，走场景10的批量生成流程

### ⛔ 简历修改的两种方式（重要 Pitfall）

**方式A：修改对照表（单岗位微调，推荐）**

当只投1-2个岗位、改动较小时，输出对照表让用户手动改：
1. 输出**「修改对照表」**，格式：`| # | 位置 | 原文 | 改成 | 理由 |`
2. 用户在 Word 中手动修改（3分钟搞定，排版零风险）
3. 用户改完后说"改好了"，再继续下一步（出 Cover Letter 或投递）

修改对照表的粒度要求：
- 必须给出**精确的原文**（用户能直接 Ctrl+F 查找）
- 必须给出**完整的替换文本**（不是"建议改成更专业的表述"这种废话）
- 按优先级分组：ATS红线必改 → 岗位对齐建议改 → 可选排序调整

**方式B：代码批量生成多版本（3+岗位，已验证可行）**

当需要同时投递多个方向（如运营+品牌+BD），每个方向需要不同简历版本时：
1. **复制** V3 base docx（`shutil.copy2`），**不修改原文件**
2. 用 python-docx 在副本上做文本替换
3. 验证替换生效后输出

**替换深度分两档：**

| 档位 | 替换内容 | 适用场景 | 风险 |
|------|---------|---------|------|
| 浅层 | 求职目标 + 个人优势摘要（中英文） | 同方向多公司（如3个BD岗） | ✅ 低 |
| 深层 | 上述 + AI项目bullet + 技术栈排序 + 工作经历bullet（中英文全改） | 跨方向（如AI推行 vs BD vs 战略合作） | ✅ 中（需验证） |

**浅层替换（run.text 直接替换）：**
```python
# 适用于同一段落内文本完整在一个 run 中的情况
shutil.copy2(SRC, out_path)
doc = Document(out_path)
for para in doc.paragraphs:
    for run in para.runs:
        if old_text in run.text:
            run.text = run.text.replace(old_text, new_text)
doc.save(out_path)
```

**深层替换（set_para_text — 跨run安全替换）：**
当修改范围覆盖整个段落（如重写工作经历bullet），文本可能跨多个 run。此时用 `set_para_text()` 更安全：

```python
def set_para_text(para, new_text):
    """替换段落全文，保留首 run 的格式（bold/size/font/color）。"""
    if not para.runs:
        para.text = new_text
        return
    first_run = para.runs[0]
    fmt = {
        'bold': first_run.bold, 'italic': first_run.italic,
        'font_size': first_run.font.size, 'font_name': first_run.font.name,
    }
    if first_run.font.color and first_run.font.color.rgb:
        fmt['color'] = first_run.font.color.rgb
    for r in para.runs:
        r.text = ''
    para.runs[0].text = new_text
    para.runs[0].bold = fmt['bold']
    para.runs[0].italic = fmt['italic']
    para.runs[0].font.size = fmt['font_size']
    para.runs[0].font.name = fmt['font_name']
    if 'color' in fmt:
        para.runs[0].font.color.rgb = fmt['color']
```

**深层替换工作流（2026-06-12 验证，3份批量生成）：**
1. 先用 `python-docx` 读取模板，打印每个段落的索引、样式、bold/font_size（size单位是EMU，如133350=10.5pt, 177800=14pt, 152400=12pt）
2. 对每份定制版，构建 `cn_changes = {段落索引: 新文本, ...}` 和 `en_changes = {...}` 两个 dict
3. 用 `create_cv(template, output, cn_changes, en_changes)` 一次性生成（内部用 shutil.copy2 + set_para_text）
4. **验证**：读取输出文件，检查关键段落（求职目标para[2]、摘要para[4]、英文目标para[56]、英文摘要para[58]）是否生效
5. 输出文件大小应与模板相近（±10KB）

**批量生成的改写策略**（每份简历改什么）：

| 段落 | 改什么 | 对齐什么 |
|------|--------|---------|
| 求职目标（中para[2]/英para[56]） | 3方向→1方向，聚焦JD核心职能 | JD title/Responsibilities |
| 个人优势（中para[4]/英para[58]） | 核心能力描述+差异点+关键词 | JD Required Qualifications |
| AI项目bullet（para[7-10]/para[61-63]） | 精简技术细节，放大业务语言 | JD对AI项目的期望深度 |
| 技术栈（para[12-13]/para[65-66]） | 前置JD关心的工具，后移深度学习 | JD Preferred Qualifications |
| 工作经历bullet（para[17-36]/para[69-93]） | 补JD关键词（governance/metrics/playbook等） | JD每条Responsibility |

**旧CV细节挖掘**：当V3缺少某类经历的细节时，检查2023旧版（`你的简历_繁体.docx`用`doc.tables[0]`读取，因为它是表格存储不是段落存储）。旧版包含更详细的"某成功案例"全过程、经销商管理实战等，可提取后融入定制版的工作经历bullet。

**⚠️ 代码编辑红线**：
- ✅ 安全：`shutil.copy2` + `set_para_text()` 或 run.text 替换
- ✅ 安全：按段落索引批量替换（只要不改段落结构）
- ❌ 危险：`doc.add_paragraph()` / 改字体 / 改间距 / 改布局 / 操作 XML
- ❌ 危险：直接修改原始 V3 文件（必须先 copy）

## 场景4: 岗位推荐分析（接收求职邮件/截图）

当用户转发求职邮件（Jobsdb/LinkedIn/猎头推荐）或询问"这些岗位怎么样"时：

**处理流程：**
1. **先读简历PDF**（强制，见前置红线）
2. 识别"舒适区陷阱"——平台算法基于旧简历推荐的岗位（L&D/HR/培训类），与用户目标方向（AI/数字化转型/新能源）严重偏离
3. 逐岗位分析匹配度，用⭐打分（5星制）
4. 分析必须务实，考虑以下用户真实约束（见下节）
5. 输出格式：表格（岗位 | 公司 | 方向 | 匹配度 | 理由）
6. 明确建议：哪些投、哪些不投、为什么

### 用户真实约束（分析时必须考虑）

| 约束 | 影响 |
|------|------|
| 1.5年无收入 | 紧迫性高，但要避免"什么岗位都投"的慌不择路 |
| 英语51周但不够纯英语职场 | 排除国际大行的英语-only 岗位 |
| 粤语仅基础听 | 排除粤语为主的本地中小公司 |
| HK无人脉无圈子 | 内推渠道关闭，靠公开投递+中资渠道 |
| IANG签证仅限香港 | 深圳/内地岗位不适用IANG，需另办工作签。猎聘等平台会混入深圳岗，必须在location维度过滤 |
| 某高校本地认可度低 | 拼学校没戏，必须拼"12年大厂+AI硕士"的组合拳 |
| **但** 某大型汽车制造企业 是某大型汽车制造企业（合资大厂） | 中企驻港视角看，这是加分项 |

### 💎 用户真实牌面（不是你以为的）

| 你以为 | 实际 |
|--------|------|
| "就是个搞营销的" | 高压工程师→用户运营→管650人→扭亏→AI硕士，**技术+管理+AI 三重经验** |
| "某高校水硕没人看" | 论文 ABSA 双任务 F1 0.67，有产品化潜力，面试不虚 |
| "香港不要营销老兵" | 中资车企/科技驻港要的就是这种**懂汽车+懂AI+能落地**的人 |
| "愿意从初级岗开始" | 37岁管过650人，投初级岗被当 overqualified |
| "AI PM完全不能投" | **要区分传统PM vs AI落地推行**（2026-06-11修正）：传统PM(roadmap/sprint)需PM经验不匹配；AI落地推行(SOP/rollout/adoption)完美匹配 |
| "BD是唯一甜蜜区" | **BD + AI落地推行 双主力**（某保险科技岗 7.5/10 > 多数BD岗） |

### 💎 投递方向优先级（2026-06-11 更新）

| 优先级 | 方向 | 证据 |
|--------|------|------|
| 🥇 主力 | BD / Business Development / Partnership / 战略合作 / Sales Director | P1岗位100%是BD方向；BD岗对行业经验容忍度最高 |
| 🥇 主力 | AI落地推行 / 变革管理 / 业务赋能 / AI Adoption / Change Enablement | 某保险科技岗 Manager 8维度7.5/10，SOP推行+区域协调直接匹配 |
| 🥈 副线 | 中资驻港创新岗（银行/央企Innovation Manager） | 中银Innovation Manager 7.1/10，SOE背景独特优势 |
| 🥈 副线 | AI方案销售/BD（技术型，如LexisNexis） | AI背景+BD能力形成差异化 |
| 🥈 副线 | 低门槛AI PM（只要"exposure"不要求"deep experience"） | HKT级别（55%），仅此一个 |
| 🚫 暂停 | Head of AI / Director级AI岗 | 要求10-15年AI领导经验，匹配度<35% |
| 🚫 暂停 | 需特定行业经验（保险/奢品/航空/加密货币） | 行业硬门槛无法弥补 |

**⚠️ 关键判断标准（2026-06-11 新增）**：
区分"传统AI PM"和"AI落地推行"：
- JD写"manage product roadmap/user stories/sprint" → 传统PM → 需要PM经验 → 大概率不匹配
- JD写"turn strategy into actions/rollout/adoption/change enablement/SOP" → 落地推行 → 你的汽车行业经验直接匹配
- JD写"promote business departments to adopt innovative technologies" → 业务赋能 → 直接匹配

### 风格要求：直球现实

- **不要理论化阐述**（"从长远来看，建议您考虑..." ❌）
- **不要绕弯子**，直接说结论（"这些岗全不投" ✅）
- **用表格/列表**，不要大段文字
- **基于事实数据**，不画饼（"F1 0.21→0.67"比"算法优异"更有说服力）
- **接受用户自我认知偏差**，但用简历事实温柔纠正

## 场景2: 投递记录

当用户发送"已投"/"投了"/"applied"开头的消息时：
1. **检查追踪文件是否存在**（优先 `job-search/投递追踪.md`，备选 `job-search/job-search-2026.md`）
2. 运行：`python3 ~/.hermes/scripts/apply_handler.py "用户原文"`
3. 检查输出，如有解析错误（如渠道识别为"未知"、岗位名混入多余文字），**手动修正追踪文件**
4. 回复确认：`已记录: 公司 | 岗位 | 渠道 | 日期`

### ⛔ Pitfall: apply_handler.py 容易把渠道/日期混入岗位名

用户说"已投 Uber APAC Senior Regional Operations Manager LinkedIn 2026-06-03"时，脚本可能把"LinkedIn 2026-06-03"当作岗位名的一部分。**投递记录后必须 grep 追踪文件验证字段对不对**，不对就用 patch 手动修正。

当用户发送**不打算投但想记录的岗位**（如"观望"/"记录一下"）时：
1. 手动追加到追踪文件的「岗位池」表格，不走 apply_handler.py（它只支持"已投"格式）
2. 标注来源平台和观望原因

### ⛔ Pitfall: apply_handler.py 需要追踪文件预先存在

脚本会检查 `TRACKING_FILE` 是否存在，不存在直接报错退出。
如果用户首次投递（文件还没建），必须先创建文件：

```bash
mkdir -p ~/你的知识库路径/wiki/projects/
```

然后创建 `~/你的知识库路径/job-search/job-search-2026.md`，格式：

```markdown
# 求职追踪 2026

## 投递记录

| 日期 | 公司 | 岗位 | 渠道 | 状态 | 回复 |
|------|------|------|------|------|------|
|------|------|------|------|------|------|

## 岗位池（待投/观望）

| 公司 | 岗位 | 来源 | 备注 |
|------|------|------|------|
| — | — | — | — |

## 面试记录

| 日期 | 公司 | 轮次 | 反馈 |
|------|------|------|------|
| — | — | — | — |
```

### ⛔ Pitfall: 快速交付物放桌面，整合后删除

用户要求：岗位扫描报告、checklist等**快速交付物**保存到 `~/job-search/inbox/`（即 `~/job-search/inbox/`），不放知识库。收件箱报告命名格式：`{平台}岗位扫货_{YYYYMMDD}.md`。

**但收件箱只是临时中转站**。扫描完成后必须：
1. 去重（对比 `投递追踪.md` 中已有条目）
2. 补漏（将有效未跟踪岗位追加到追踪文件）
3. **删除收件箱源文件**

知识库 `08-找工作/` 是唯一长期存放地。

### ⛔ Pitfall: 纯文本模型不要请求截图确认

当前主力模型是 mimo-v2.5-pro（纯文本LLM），无法处理图片。**不要要求用户截图确认**——直接用文字描述验证即可。

### ⛔ JobsDB API 关键词+地区过滤 Pitfall（2026-06-11 实测）

JobsDB 搜索 API (`/api/jobsearch/v5/search`) **没有独立的地区过滤参数**。如果在关键词中加"Hong Kong"，反而会因为关键词太精确返回0结果。

**正确做法**：
1. 用原始关键词搜索（不加地区）：`keywords=AI%20product%20manager`
2. 在返回结果的 `locations` 字段中过滤HK：检查 `locations[].label` 或 `locations[].seoHierarchy` 是否包含 "Hong Kong"/"Kowloon"/"Central"/"Wan Chai" 等
3. API 字段名映射：
   - 公司名：`companyName`（不是 `company.name`）
   - 薪资：`salaryLabel`（不是 `salary`）
   - 摘要：`teaser`
   - 要点：`bulletPoints`（不是 `bullets`）
   - 地点：`locations[].label`

**⚠️ Pitfall：API 会返回澳洲/新加坡等非HK岗位**（JobsDB是亚太平台），必须做location过滤。

### ⛔ JobsDB 三层防线（2026-06-10 实测）

JobsDB 不是"全死"——网页层死了，搜索 API 还活着：

| 层级 | 状态 | 说明 |
|------|------|------|
| ❌ 网页详情页 `jobsdb.com/job/<id>` | 403 Cloudflare | curl + browser 都被挡 |
| ❌ browser_navigate | Turnstile 拦截 | headless browser 过不去 |
| ✅ **搜索 API** `/api/jobsearch/v5/search` | 200 JSON | 返回标题/公司/摘要/bullets，无需登录 |
| ✅ 邮件订阅 `noreply@e.jobsdb.com` | 正常 | gmail_job_scanner.py 已在用 |
| ✅ 本机 Chrome（CDP） | 正常 | 用户手动打开可过 Cloudflare |

**正确方案**：API 做发现 + 粗筛（teaser/bullets 通常够判断 70%），本机 Chrome 做完整 JD 核实。详见 `references/jobsdb-api-workflow.md`。

### ⛔ LinkedIn 也是半自动化

| 层级 | 状态 |
|------|------|
| ❌ Guest API（旧格式） | 2026-06 起返回空结果 |
| ⚠️ browser 搜索列表（guest） | 关弹窗后能看到 ~10-15 条摘要 |
| ⚠️ browser 详情页 | 需要登录 |
| ✅ 浏览器 console 提取 job IDs | `document.querySelectorAll('a')` + regex `-({10}位数字)(?:\?|$)` |

LinkedIn browser 列表页的 URL 格式是 `hk.linkedin.com/jobs/view/slug-4425452567`，job ID 在最后一个 `-` 后面。详见 `references/jobsdb-api-workflow.md` 中的 browser 提取方法。

### 已知投递渠道

> 完整渠道全景图（含中资/央企/简职HK/CTgoodjobs/劳工处等）: 见 `references/hk-job-channels.md`

| 渠道 | 平台 | 质量定位 |
|------|------|---------|
| JobsDB | jobsdb.com | 中高端白领，主力渠道 |
| CTgoodjobs | ctgoodjobs.hk | 中高端白领 |
| LinkedIn | linkedin.com | 科技/外企，需英文简历 |
| 劳工处互动就业服务 | www2.jobs.gov.hk | 基础岗/蓝领为主，中低质量，偶尔有合适的 |
| 官网直投 | 各公司官网 | 中资驻港机构优先 |
| 猎头 | — | 被动渠道，等推荐 |
| OfferToday | offertoday.com | 新兴平台，类似BOSS直聘，算法匹配质量偏低，容易推保险/理财马甲岗 |
| 邮件投递 | JD中的邮箱 | 需附Cover Letter |

**解析渠道时**，apply_handler.py 只识别有限的关键词列表。如果用户说的是"劳工处""互动就业""jobs.gov.hk"等，脚本会识别为"未知"，需要手动修正。

当用户发送"查看投递"/"投递记录"时：
1. 读取 `~/你的知识库路径/job-search/job-search-2026.md`
2. 输出投递记录表格摘要

## 场景7: 邮箱岗位扫描

当用户说"扫邮箱"/"看看邮件里的岗位"/"邮箱推荐"/"Gmail求职邮件"/"scan job emails"时，执行此场景。

**详细流程**：见 `references/gmail-job-scanner.md`（himalaya pitfalls、邮件格式、评分逻辑、setup步骤）

**快速流程**：
1. 运行 `python3 ~/.hermes/scripts/gmail_job_scanner.py --days 7 --output text --save`
2. 输出结构化报告（★★★★★强推 → ★低匹配 → EXCLUDED）
3. 重点展示强推岗位（≥★★★★），简要列出可考虑岗位
4. 用户选择后，进入场景1（简历微调）或场景5（Cover Letter）

**⚠️ Pitfall**：himalaya JSON 输出混有 WARN 行，`from` 字段是 dict 不是 string。详见 `references/gmail-job-scan-workflow.md`。

## 场景8: 主动求职搜索

当用户说"去XX上跑一圈"/"看看有什么岗位"/"主动出击"/"扫一遍"时，执行此场景。

**详细流程**：见 `references/active-job-search-workflow.md`
**JD 抓取方法**：见 `references/jd-fetching-methods.md`
**Glassdoor HK 注册与使用指南**：见 `references/glassdoor-hk-guide.md`（注册流程、贡献换访问选Create a job alert、Cloudflare不可自动化、与JobsDB配合使用）

**香港渠道全景**：见 `references/hk-job-channels.md`（含劳工处/CTgoodjobs/简职HK/中资官网）

**快速流程**：
1. 用 LinkedIn Guest API（curl，首选）按关键词矩阵批量搜索；备用：browser_navigate + JobsDB API
2. 去重 → 评分 → 过滤黑名单 → 分类（强推/观望/坑）
3. 表格呈现结果，附匹配理由和行动建议
4. 用户选择后，进入场景1（简历微调）或场景5（Cover Letter）

**⚠️ Pitfall**：不要用子Agent的webSearch搜LinkedIn/JobsDB，也不要依赖browser_navigate（易超时）。必须用curl直接调LinkedIn Guest API。详见 `references/active-job-search-workflow.md`。

---

## 场景5: 求职邮件/Cover Letter 生成

当用户确认简历调整完成后，或直接说"出求职邮件"/"写cover letter"/"帮我写申请邮件"时，执行此场景。

**触发条件**：场景1的简历调整完成后用户确认"改好了"，或用户直接要求。

**处理流程：**
1. 读取已确认的简历内容（确保用最新版）
2. 回顾目标JD的核心要求
3. 生成英文求职邮件，结构如下：
   - 主题行：`Application for [职位] (Ref: [编号]) — [姓名]`（带Ref编号方便HR归档）
   - 开头：简洁说明申请什么岗位，一句话概括核心资历
   - 正文：3-4个bullet points，**每个直接映射JD的一条要求**，用简历中的具体数据支撑
   - 签证/IANG状态：明确写出，降低HR顾虑
   - 结尾：附简历、期待面试
4. 提供附件命名建议：`Resume_[姓名]_[公司]_[岗位].pdf`

**Cover Letter 语言选择规则（重要）：**

| 公司类型 | 判断信号 | 邮件语言 | 简历版本 |
|----------|---------|---------|---------|
| 科技/外企/海外业务 | NYSE上市、英文JD、IoT/SaaS、中环办公 | 英文 | 英文简历 |
| 传统内地背景企业 | QQ邮箱、观塘/新蒲岗、进出口贸易、中文JD | 中文 | 中文简历 |
| 中资驻港机构 | 国企/央企驻港、中英文混用 | 视JD语言定 | 中英文简历 |

**错误示范**：给QQ邮箱的内地贸易公司发全英文Cover Letter → 对方觉得你找错门了

**Cover Letter 风格要求：**
- **不要用空话**，每句话都要有数据或事实支撑
- **不要超过一页**（英文约300-400词，中文约400-500字）
- 每个段落/bullet point结构：**JD关键词 → 你的具体经历 → 量化结果**
- 中文邮件风格：简洁商务，用【】做段落小标题突出结构，不用老派格式

**Cover Letter 双语结构模板（2026-06-12 验证，3份CL成功产出）**：

```
# Cover Letter — {公司} {岗位}
```

**LinkedIn 快速申请后的跟进（找recruiter/补CL）**：见 `references/linkedin-recruiter-contact.md`（消息模板中英文短版+长版，查找招聘官的正确方法）

**求职申请表填写模板**：见 `references/application-form-templates.md`（标准字段速查、薪资单位陷阱、US/EST时区判断、粤语要求判断）

**LinkedIn快速申请后不需要补CL**：LinkedIn快速申请只发简历，不支持补材料。系统回执邮箱不收附件。ByteDance等ATS系统入库后按流程筛选，CL在初筛阶段几乎不看。如有心做一步，找recruiter发LinkedIn消息比邮件CL有效10倍。
---

## 中文版

尊敬的招聘团队：

我写信申请{岗位}一职。{一句话概括核心资历+为什么适合}。

在{公司}的{N}年里，{核心能力概述}：

- **{JD关键词1}**：{具体经历}，{量化结果}
- **{JD关键词2}**：{具体经历}，{量化结果}
- **{JD关键词3}**：{具体经历}，{量化结果}
- **{JD关键词4}**：{具体经历}，{量化结果}

{AI/技术差异化段：硕士项目经历如何补充JD需求}

{核心差异总结句：不是XX，而是XX——直接呼应JD核心需求}

期待与您进一步交流。

此致
{姓名}

---

## English Version

Dear Hiring Team,

I am writing to express my interest in the {Position} at {Company}. {Core qualification sentence}.

Throughout my career at {Company}, {competency summary}:

- **{JD keyword 1}**: {specific achievement}, {quantified result}
- **{JD keyword 2}**: {specific achievement}, {quantified result}
- **{JD keyword 3}**: {specific achievement}, {quantified result}
- **{JD keyword 4}**: {specific achievement}, {quantified result}

{AI/tech differentiation paragraph}

{Core differentiator sentence}

I look forward to discussing how my experience can contribute to {Company}'s goals.

Best regards,
{Full Name}

---

- **投递渠道**: {LinkedIn/邮件}
- **适用简历**: {V3-{公司}-{方向}.docx}
- **文件路径**: D:\个人\个人简历\Cover_Letter_{公司}_{方向}.md
```

**关键规则**：
- 每份CL的4个bullet point必须**直接映射JD的4条核心要求**，用简历中最相关的经历对应
- Adecco等猎头代招岗位：用邮件格式（Dear Jessica），末尾附联系方式+LinkedIn，附件用定制版简历。同时LinkedIn快速申请+邮件跟进双管齐下，邮件主题带LinkedIn Ref编号方便猎头归档
- 每份CL保存为.md文件到 `D:\个人\个人简历\`，方便后续修改和复用
- 不要一次性写3份CL——先出1份确认风格，再批量出（但本次3份一次性出也OK，因为JD差异足够大）

**已验证的成功模板**：见 `references/cover-letter-template.md`

### ⛔ Pitfall: delegate_task 并行任务容易超时中断

当同时委托多个耗时任务（如"3份CL + 9个JD抓取"）时，`delegate_task` 的子Agent容易因父Agent超时而被中断（本次session两个子任务都被interrupted）。

**对策**：
1. **CL写作直接在主Agent完成**（不需要子Agent，写CL是纯文本生成，主Agent几秒搞定）
2. **JD抓取用 `execute_code` 批量curl**（9个JD < 15秒，不需要子Agent）
3. 只有**真正需要独立上下文的重型任务**才用 delegate_task（如深度研究、多文件分析）
4. 如果必须并行：每个子任务的goal要尽可能自包含，减少子Agent需要的工具调用次数

## 场景3: 灵感归档

当用户发送"记个灵感"/"/idea"开头的消息时：
1. 提取灵感内容（去掉前缀）
2. 运行：`python3 ~/.hermes/scripts/idea_handler.py "灵感内容" "tag1,tag2"`
3. 回复确认：`已收录 → YYYY-WNN-ideas.md`

当用户发送"查看灵感"/"灵感列表"时：
1. 读取 `~/你的知识库路径/raw/notes/idea-inbox/` 本周文件
2. 输出灵感列表

## 场景6: 求职平台资料更新

**LinkedIn 详细指南**：见 `references/linkedin-profile-guide.md`（技能列表、置顶建议、ATS 禁词、Open to Work 设置等）

当用户更新了简历（新版本）后，需要在所有已注册的求职工平台上同步更新个人资料和简历。

**LinkedIn 详细优化指南**：见 `references/linkedin-profile-setup.md`（Headline/Skills/About/Projects 的具体写法、ATS红线词在LinkedIn上的清洗、岗位评估流程、LinkedIn JD提取方法）。

### 已知香港求职平台清单

| 平台 | 网址 | 类型 | 备注 |
|------|------|------|------|
| JobsDB | jobsdb.com | 中高端白领 | 主力渠道 |
| **Glassdoor HK** | glassdoor.com.hk | 中高端白领 | 公司评价+薪资透明为主，注册需WhatsApp绑定，注册时"贡献换访问"选Create a job alert最省事。Cloudflare反爬，browser_navigate不可用 |
| CTgoodjobs | ctgoodjobs.hk | 中高端白领 | — |
| OfferToday | offertoday.com | SPA平台(BOSS直聘架构) | 后端用zhipin.com SDK, API路径`/wapi/geek/*` |
| LinkedIn | linkedin.com | 科技/外企 | 需英文简历 |
| 猎聘 | liepin.com | 中高端 | 中资覆盖好，支持LinkedIn式快速申请，深圳/内地岗也混入需过滤 |
| 劳工处互动就业 | www2.jobs.gov.hk | 基础岗 | 中低质量 |
| Indeed HK | indeed.hk | 综合 | — |
| JIJIS | jijis.org.hk | 应届/实习 | 某高校可注册 |

### 平台资料更新流程

简历新版本发布后，按以下步骤逐一更新：

1. **列清单**：确认用户已注册哪些平台，逐一更新
2. **每平台检查项**：
   - 在线简历/个人资料（平台自带编辑器）
   - 附件简历（上传最新PDF）
   - 求职期望/偏好设置（岗位类型、薪资范围、地点）
   - 求职状态（设为「正在找工作」）
   - 头像（正式职业照）
3. **逐平台操作**：指导用户在自己浏览器操作（浏览器工具对SPA站点可能有编码问题），截图确认
4. **完成后记录**：更新追踪文件，记录各平台最后更新日期

### 探测 SPA 求职平台结构的方法

当浏览器工具不可用时，用 curl 分析 SPA 平台的路由结构：

```bash
# 1. 下载主 JS bundle
curl -s "https://<site>/" | grep -oP 'src="[^"]*\.js"' | head -10

# 2. 从 bundle 提取路由
curl -s "https://<site>/static/js/index.XXXX.js" | grep -oP '"/[a-z][a-z0-9/-]*"' | sort -u

# 3. 提取 API 端点
curl -s "https://<site>/static/js/index.XXXX.js" | grep -oP '"/wapi/[^"]*"' | sort -u
```

这样可以在用户操作前了解平台结构（注册入口、简历管理、求职期望等页面路径）。

## 场景12: 每日求职固定动作（"今天继续求职"）

当用户说"今天继续求职"/"走一遍固定动作"/"求职日常"时，执行此场景。

**标准流程（串行，每步完成后再进下一步）**：

```
Step 1: Gmail邮箱扫描
  python3 ~/.hermes/scripts/gmail_job_scanner.py --days 2 --output text --save
  → 提取★★★★★强推 + ★★★★值得投

Step 2: Hunter岗位搜索
  python3 ~/.hermes/scripts/hunter.py
  → 提取高匹配(≥70%)岗位

Step 3: 读投递追踪表，检查待办
  ~/你的知识库路径/job-search/投递追踪.md
  → 确认昨日P0/P1待投是否已执行

Step 4: 汇总当日扫描结果，呈现给用户
  格式: 数据概览 + 新增高匹配 + 新增强推 + 待办提醒

Step 5: 自动进入JD匹配流程（2026-06-15 生效）
  → 自动抓取今日新增高匹配(≥65%)岗位的JD
  → 保存到JD库（英文原文+中文翻译）
  → 评估匹配度（基于JD全文，不是关键词）
  → 更新追踪表（添加JD链接列）
  → 呈现核实结果，等用户决策投/不投
```

**⚠️ Step 5 自动化规则（2026-06-15 新增）**：

用户明确要求"每日任务更新后直接找JD匹配"，不再等用户手动说"去抓JD"。

自动匹配流程：
1. 从Step 1-2的结果中筛选：
   - 邮箱★★★★★强推 + ★★★★值得投
   - Hunter≥65%高匹配
2. 去重：排除追踪表中已有的岗位
3. 批量抓取JD（curl LinkedIn，API JobsDB）
4. 保存到JD库：`{ID}_{公司}_{岗位}.md`（英文+中文翻译）
5. 匹配度评估（9维度核实）
6. 更新追踪表：添加JD链接、标注核实层级[L3]
7. 呈现结果表格：公司 | 岗位 | 匹配度 | 硬伤 | 建议

**⚠️ 自动匹配的边界**：
- 只自动抓JD+评估，**不自动生成简历**（需用户确认）
- 只自动抓JD+评估，**不自动投递**（红线）
- 匹配度<55%的岗位只记录，不推荐
- 每日新增岗位通常10-30个，批量处理<5分钟

**关键规则**：
- 周日不自动跑cron（cron配置Mon-Sat），需手动执行
- Hunter匹配度虚高（见场景11），**必须JD核实后才能纳入"高匹配"列表**
- 邮箱scanner的★★★★★也需核实（命中率约50%，高于Hunter但仍不完美）
- 每次扫描完必须更新追踪表（不可省略）
- 先检查昨日未完成的P0任务，再看今日新增
- **用户高频操作**：把今日新增高匹配和强推的岗位都抓JD，然后匹配 — 现在自动化了

## 场景11: 批量JD核实 + 匹配度评估

当用户说"帮我核实这些岗位"/"批量抓JD"/"做一遍匹配"/"都抓JD然后匹配"时，执行此场景。

### 核心教训：关键词匹配 ≠ 人岗匹配

**自动化扫描（Gmail scanner / LinkedIn API）的"强推"评级是关键词匹配，不是人岗匹配。** 必须抓取JD全文做人工核实。

**2026-06-10 实测数据**（75个岗位全量核实）：
- 关键词匹配"强推"的13个岗位，核实后只有2个≥65%（15%准确率）
- 昨天标记75%的8个岗位，核实后平均只有40%（大幅缩水）
- **AI PM方向几乎全军覆没**（要求3-5年PM经验，用户零PM经验）
- **BD/战略合作方向才是甜蜜区**（对行业经验容忍度最高）

### JD抓取方法

详见 `references/linkedin-jd-fetch.md`

**快速流程**：
1. 有LinkedIn ID → `curl` 抓取 guest view 页面，提取 `show-more-less-html__markup` div
2. 只有公司+岗位名 → `browser_navigate` 到 LinkedIn 搜索页，提取 ID 后再抓
3. JobsDB → Cloudflare 403，**必须让用户手动打开浏览器查看**

**批量抓取（2026-06-14 更新，推荐delegate_task并行）**：

| 方法 | 适用场景 | 速度 | 备注 |
|------|---------|------|------|
| `delegate_task` 并行（推荐） | 7+个JD，分2批 | ~60-90秒/批 | 每批≤7个，2批并行 |
| `execute_code` 循环curl | 1-6个JD | ~15秒 | 不需要子Agent |

**delegate_task并行法（2026-06-14验证，14个JD分2批≈66秒/批）**：
```python
# 批次1: 7个LinkedIn URLs
delegate_task(
    goal="Fetch full JD from these LinkedIn URLs. For each, curl + regex extract show-more-less-html__markup div. Return full JD text labeled with company+role.",
    toolsets=["terminal", "web"],
    # 列出所有URL
)

# 批次2: 另7个（同时并行）
delegate_task(...)
```
子Agent用curl下载HTML，正则提取`show-more-less-html__markup`或`description__text` div。成功率：LinkedIn≈95%，JobsDB≈0%（Cloudflare）。

**execute_code单批法（1-6个JD，更快）**：
```python
from hermes_tools import terminal
for url in urls:
    r = terminal(f'curl -sL "{url}" -H "User-Agent: ..." | python3 -c "提取markup"', timeout=20)
    jd = r.get("output", "").strip()
```

### 匹配度评估框架

对每个岗位的JD全文做以下检查：

**Step 1: 关键词初筛**（自动，仅用于排序，不可用于决策）
- strong_match: BD/Partnership/Account/Operations/Strategy/Revenue
- medium_match: Digital Transformation/AI/Marketing/Consulting/PM
- red_flags: Insurance/Banking/Luxury/Crypto/日语/韩语/15+年经验

**Step 2: 人工核实**（必须，基于JD全文）
- 行业经验要求（决定性维度）
- 核心职能匹配（决定性维度）
- 硬性年限/资质/语言要求
- 技术栈深度要求
- 目标区域市场经验

**Step 3: 务实评估**
- ≥65%: ✅ 可投，出定制简历
- 55-64%: ⚠️ 观望，需用户判断是否有隐性匹配
- 45-54%: ❓ 低匹配，除非用户坚持否则不投
- <45%: ❌ 不投，简要说明原因

### ⛔ 红线：不要盲目乐观

用户明确要求**务实不画饼**。核实后匹配度比关键词匹配低20-30%是常态。宁可低估也不高估——高估导致无效投递，浪费时间和心理能量。

**2026-06-10 量化校准**：
- 关键词匹配"75%"的岗位，JD全文核实后平均只有**40%**（缩水35个百分点）
- 关键词匹配"★★★★★强推"的13个岗位，核实后仅**2个≥65%**（15%命中率）
- **系统性误判源**：关键词"AI"+"Product Manager" 对无PM经验的转型者是假阳性——大部分AI PM岗要求3-5年PM经验
- **校准公式（经验法则）**：关键词匹配分 × 0.6 ≈ 核实后真实匹配度

**2026-06-14 二次验证（扩大样本）**：
- Hunter标75%的**15个**岗位，JD全文核实后实际分布在8%-30%（中位数22%，缩水53pp）
- **0个≥40%**——这批全部不值得投，无一例外
- **修正系数**：75%档位 × 0.3 ≈ 真实匹配度（不是0.6）
- 邮箱★★★★★强推的6个岗位，核实后3个值得投（50%命中率，高于Hunter）
- **结论**：Hunter自动评分**严重虚高**——75%档位平均缩水53pp，60-68%档位缩水约20pp。邮箱scanner的评分相对更准（因为有薪资/地点等额外信号）
- **实操规则**：Hunter标75%的岗位**必须先JD核实**，不能直接纳入"高匹配"列表
- **核实后最高匹配度分布（15个原标75%岗）**：30%(HKT) > 28%(Alexis) > 25%(Wing/Amadeus) > 22%(Teleport) > 20%(Prudential/Horizon/Coach) > 18%(Binance Chatbot) > 15%(Michael Page/Sompo/Robert Walters/Sia) > 10%(TakeX) > 8%(Binance DeFi)。系统性原因：JD的行业硬门槛（保险/航空/crypto/对冲基金/NGO）是关键词匹配无法捕捉的

### ⛔ 中资传统行业 > 中资科技企业

用户求职信息差认知（2026-06-10 抖音视频）：
- 中资**传统行业**（银行后台、基建、贸易、物流、地产）= 双休多 + 对内地人友好 + 不强制粤语
- 中资**科技企业**（华为/腾讯/阿里）= SPA官网难抓 + 岗位多走内推 + 竞争激烈
- **简职HK公众号** 80%+是传统行业中资/央企，是最高效的中资捡漏渠道
- 不要只盯携程/华为/字节——招商局、华润、中旅、中远、OOCL 等综合央企的管理岗更匹配

### ⛔ subagent 无法抓取外部网页

`delegate_task` 的子Agent只有 Feishu MCP 工具，**无法执行 curl 或抓取外部 URL**。LinkedIn/JobsDB JD抓取必须由主 Agent 直接执行（terminal + curl），不要委托给子Agent。

### 文件组织

用户偏好将求职相关文件统一存放在知识库的 `08-找工作/` 目录：
- `job-search/岗位匹配总报告.md` — 汇总报告
- `job-search/投递追踪.md` — 投递记录（从 `job-search/job-search-2026.md` 同步）
- `job-search/核实进度.md` — 待核实清单
- `job-search/JD库/` — 每个岗位的JD全文（`{ID}_{公司}_{岗位}.md`）

**快速交付物**（扫描报告等）先放 `~/job-search/inbox/`，生命周期为：
1. 扫描产出 → 桌面（`{平台}岗位扫货_{YYYYMMDD}.md`）
2. 核实/评分 → 整合进 `job-search/投递追踪.md` + `岗位匹配总报告.md`
3. **整合完成后删除收件箱源文件**（收件箱仅作临时中转站，不长期存放）

所有求职数据的**唯一权威来源**是 `08-找工作/` 目录。

### ⛔ Pitfall: 追踪文件重写必须保留P0+按匹配度排序（2026-06-14教训）

重写岗位池时（如批量重排序、合并数据）：
1. **P0已投递/材料就绪的岗位必须放在最顶部**，不受排序规则影响
2. **排序规则是匹配度高→低**，不是时间顺序。用户明确要求过"不完全按时间，按强推+匹配度高到低排序"
3. 重写后**必须自检**：P0 ByteDance和Wing是否还在？已投递记录是否完整？（本次重写时两个P0岗位被吞掉，用户10分钟内发现）
4. **已核实和未核实的岗位必须分section**，不能混在一起（用户无法区分哪些分数可信）
5. **merge策略**：用`execute_code`重写整个文件时，先提取header(投递记录)和footer(面试记录)，只替换中间的岗位池section。不要用`write_file`覆盖整个文件——容易丢header/footer中的投递记录
6. **已投递状态列必须保留**：P0岗位的"状态"列（如`🔥今天投`→`✅已投06-14`）不能在重写时丢失

岗位池标准section结构：
```
### 🔥 P0 — 材料就绪，立即投递
### ⭐ P1 — 已核实≥60%，定制简历投递
### ⚠️ P2 — 已核实50-59%
### ❌ 已核实<50%，不投
### 📋 未核实岗位（仅参考，需走L2/L3才能决策）
```

**⚠️ 岗位池表格新增「JD链接」列（2026-06-15 生效）**：

岗位池表格格式更新为：
```
| 公司 | 岗位 | 方向 | 匹配度 | 链接 | JD链接 | 材料 | 状态 |
```

- **链接**：LinkedIn/JobsDB 岗位页面链接
- **JD链接**：本地JD库文件路径（`job-search/JD库/{ID}_{公司}_{岗位}.md`）
- 当抓取JD并保存到JD库后，**必须同时更新追踪表的JD链接列**
- 格式：`[JD](../JD库/4426911071_Morgan_McKinley_AI_Advisory_Lead.md)`（相对路径）

### ⛔ Pitfall: delegate_task 搜外部网站容易卡死

当需要查找LinkedIn招聘官、公司官网信息等**简单网页查找任务**时，不要用`delegate_task`。子Agent会花200+秒尝试各种curl/browser方案最终被interrupted。

**正确做法**：
1. 主Agent直接`curl`尝试（5-10秒内能拿到结果）
2. 拿不到（LinkedIn需登录）→ 直接告诉用户"在LinkedIn登录状态下打开链接，看右侧Posted by"，并**同时给出备用消息模板**
3. 只有真正需要深度研究的任务才用delegate_task

### ⛔ Pitfall: 追踪文件双副本必须同步（2026-06-12 教训）

存在两个追踪文件副本：
- **主文件**：`~/你的知识库路径/job-search/投递追踪.md`（用户日常查看）
- **旧文件**：`~/你的知识库路径/job-search/job-search-2026.md`（早期创建）

apply agent 的 MEMORY (`~/.hermes/profiles/apply/MEMORY.md`) 也维护了一份已投递列表。

**这次出的问题**：apply MEMORY 有 10 条（含 CASETiFY），追踪文件只有 9 条（漏了 CASETiFY），wiki 版有 06-12 新条目但主文件没有。

**规则**：
1. 任何投递记录变更，必须**同时更新**三个位置：`job-search/投递追踪.md` + `job-search/job-search-2026.md` + `apply/MEMORY.md`
2. **排查漏记时**，先读 `apply/MEMORY.md`（它由apply agent独立维护，可能有主追踪文件遗漏的条目，如CASETiFY）。三方对比找差异是最快的补漏方法。
3. 添加待投岗位时，**必须标注核实层级**（见下节），不能让"关键词粗筛"和"JD全文核实"混为一谈
3. 定期（每周）用 `diff` 检查两个文件是否同步

### ⛔ Pitfall: 待投岗位必须标注核实层级（2026-06-12 教训）

用户问"06-12新增的9个已经经过JD匹配了吗？"——答案是没有，只是关键词粗筛。但追踪文件里没有标注区分，导致用户无法判断哪些可以直接投、哪些还需要核实。

**追踪表新增条目时必须标注来源核实层级**：

| 层级 | 标记 | 含义 | 可否直接投 |
|------|------|------|-----------|
| L1 关键词粗筛 | `[L1]` | 自动化scanner输出，仅关键词匹配 | ❌ 需先走L2 |
| L2 JD摘要匹配 | `[L2]` | 基于teaser/bullets的半自动评估 | ⚠️ 可投但建议看全文 |
| L3 JD全文核实 | `[L3]` | 9维度逐条对照，有结构化评分 | ✅ 可直接投 |
| L4 用户确认 | `[L4]` | 用户看过JD并决策 | ✅ 已确认 |

在追踪表的"状态"列中追加层级标记，如：`待投 [L1]` 或 `待投 [L3]`。用户决策后升级为 `[L4]`。

---

## 黑名单

#

## 黑名单

以下岗位直接标记为不匹配（不用分析直接跳过）：
- 地产经纪 / real estate agent
- 纯销售（无管理职责）
- 保险代理 / insurance agent
- 创业公司（<20人）

## 场景7: 邮箱岗位扫描

当用户说"扫邮箱"/"看看邮件里的岗位"/"邮箱推荐"/"scan emails"/"check job emails"时，执行此场景。

**数据源（5步扫描，2026-06-10 升级）**：
1. Gmail LinkedIn 邮件（jobalerts / jobs-noreply）
2. Gmail JobsDB 邮件（noreply@e.jobsdb.com）
3. Gmail OfferToday / Indeed / Glassdoor 邮件
4. JobsDB API 8组关键词搜索（覆盖 CTgoodjobs/劳工处同类岗）
5. 去重 + 匹配评分

**工具**：`~/.local/bin/himalaya` (IMAP CLI) + `~/.hermes/scripts/gmail_job_scanner.py` + `~/.hermes/scripts/job_feedback_scanner.py`（应用反馈追踪）
**配置**：`~/.config/himalaya/config.toml` (Gmail App Password 认证)

**执行命令**：
```bash
python3 ~/.hermes/scripts/gmail_job_scanner.py --days 7 --output text --save
```

**⚠️ 新增数据源说明**：
- OfferToday 邮件：用户已注册，推荐质量偏低（保险马甲多），scanner 自动解析
- Indeed 邮件：`donotreply@match.indeed.com`，已有邮件在 inbox
- Glassdoor 邮件：`noreply@glassdoor.com`，已有邮件在 inbox
- JobsDB API 搜索：8组关键词（operations director / BD director / general manager / partnership / digital transformation / AI PM / growth / marketing director），每组15条，来源标记为 `JobsDB-API`

**输出**：结构化报告到 `~/你的知识库路径/raw/notes/jobs/YYYY-MM-DD-email-jobs.md`
**评分逻辑**：关键词匹配（high=3分, medium=1分）→ ★★★★★ 强推 / ★★★★ 值得投 / ★★★ 可考虑 / ★★ 观望 / ★ 低匹配 / EXCLUDED

**⚠️ Pitfall: himalaya search syntax**
himalaya uses lowercase IMAP-style search, NOT Gmail search syntax:
- Correct: `from "linkedin"`, `subject "job"`
- Wrong: `from:linkedin.com` (Gmail syntax, himalaya rejects with parse error)
- `--output json` returns JSON array; WARN lines from imap_codec may be mixed in — extract JSON with `output.find('[')` then `json.loads()`
- `from` field in JSON is a dict `{name, addr}`, not a string

**⚠️ Pitfall: Email parsing noise**
LinkedIn/JobsDB emails contain lots of non-job content (footer links, CTAs, tracking URLs). The parser must filter:
- LinkedIn: "简单几步，轻松迈向成功", "编辑订阅", "其他订阅", "查看领英", "查看全部职位", "有符合您的搜索偏好", "您已成功订阅"
- JobsDB: "Rate your recent employer", "apple store", "google play", "Edit frequency", "View more jobs", lines starting with `* ` (requirement bullets)
- Both: "Subject:", "To:", "From:", `<strong`, any `[https://` links
- Hong Kong district names misidentified as company names → post-fix in score_job() to move to location field

**⚠️ Pitfall: himalaya 2FA requirement**
Gmail App Password requires 2FA enabled first. Without it, the App Password page shows "您的账号不支持您正在尝试的设置". See himalaya skill for full setup sequence.

**详细邮件解析模式**：见 `references/gmail-job-email-patterns.md`

### Daily cron delivery

已配置 cron job (每日10:00)。delivery target 默认 local，用户可改为飞书群。
设置命令：`cronjob(action='update', job_id='...', deliver='feishu:CHAT_ID')`

### 🔍 Overqualified 快速识别信号

以下信号出现任何一个，该岗位大概率是 overqualified，直接标红警示：

| 信号 | 示例 | 判断 |
|------|------|------|
| 职位名含「助理/Trainee/Assistant/Intern」 | 助理算法工程师 | ❌ 入门岗 |
| 要求「1年经验」而你有12年 | — | ❌ 严重不匹配 |
| 备注提到「中高龄就业计划」/「就业展翅」 | 政府补贴聘用40+员工 | ❌ 针对再就业人群 |
| 月薪 ≤ $20,000 且你目标是 $30k+ | $18,000/月 | ❌ 薪资断崖 |
| 技术方向不对口（如你要NLP但岗要CV） | YOLO/CNN/OpenCV vs RoBERTa/NLP | ❌ 技能栈不匹配 |

**处理方式**：不投，简要说明原因（3行以内），记录到「岗位池」标注「观望-不投」。不要长篇大论分析——浪费用户时间。

### 🎯 JD"硬性要求"的判断框架

当用户问"JD要求X，我只有实战经验/不完全满足，是否投？"时，用以下框架快速判断：

**核心原则：JD的"硬性要求"是筛选门槛，不是天花板。**

| 维度 | 判断规则 | 示例 |
|------|---------|------|
| 要求是工具还是核心职能 | 工具类要求（SQL/Excel/Python）在管理岗 = 数据自服务能力，不是数据工程岗 | OPS Manager要SQL → 分析能力门槛，投 |
| 要求在JD中的位置 | 越靠后越可能是nice-to-have | SQL放在第8条vs放在第1条 |
| 岗位级别 | 高级管理岗的技术要求通常低于同技术栈的初级岗 | Senior OPS Manager的SQL ≠ Data Analyst的SQL |
| 用户实际能力 | "实战中应用过" = 有能力，只是没证书/没专门培训 | 用SQL做过数据管道 → 够用 |

**决策矩阵：**

| 用户水平 | 岗位级别 | 决策 | 理由 |
|----------|---------|------|------|
| 实战应用过 | 管理岗 | ✅ 投 | 管理岗的技术要求是门槛，不是核心 |
| 实战应用过 | 技术岗 | ⚠️ 看具体深度 | 可能需要面试前突击 |
| 完全不会 | 管理岗 | ⚠️ 面试前速学 | 短期可补 |
| 完全不会 | 技术岗 | ❌ 不投 | 核心技能缺失 |

**回复风格**：直接给结论（"投"/"不投"），一句话理由，不要长篇分析。用户问的是决策，不是教育。

### ⛔ 匹配度必须诚实——不能盲目乐观

**核心教训（2026-06-10）**：之前把 多个目标公司 标为"🎯立即投"，实际匹配度只有 35-50%，多份定制简历浪费。

**规则**：
1. **关键词匹配 ≠ 人岗匹配**。"AI" + "Product Manager" 命中不代表能投——大部分 AI PM 岗要求 3-5 年 PM 经验，转型者 0 年。
2. **但"AI PM"不等于一刀切砍掉**（2026-06-11 修正）。区分：
   - 传统AI PM（manage roadmap/user stories/sprint）→ 需PM经验 → 大概率不匹配
   - AI落地推行（turn strategy into actions/rollout/adoption/SOP）→ 需执行力 → 完美匹配
   - AI创新管理（promote adoption of innovative technologies/SOE innovation）→ 需业务理解 → 匹配
3. **简历包装质量 ≠ 匹配度**。写得好的简历照样匹配度低，必须分开评估。
4. **核实必须逐项对照 8-9 维度**，不能只查单一维度就标"可投"。之前 WPP 只查了语言门槛就标了 🎯，漏掉了"必须有媒体行业高级领导经验"。
5. **用户的真实甜蜜区是 BD/战略合作 + AI落地推行/变革管理**，不只是BD。
6. **匹配度 <50% 的岗位不要出定制简历**——浪费时间和简历模板。
7. **"轻微 overqualified"是可接受的**——用户明确说了愿意接受。

**香港市场现实**：竞争激烈，匹配度 65%+ 才有真实面试机会，50-64% 是观望区（需强 storytelling），<50% 是浪费投递额度。

### ⛔ 舒适区陷阱（重点注意）

以下岗位**看起来相关**但实际是陷阱，必须打 ⭐ 低分并明确警示：

| 岗位类型 | 陷阱原因 |
|----------|---------|
| L&D / Training / HR | 基于简历中"人才发展"经历推荐，但会把职业路径锁死在HR方向 |
| 任何初级岗（Trainee/Assistant/Intern） | 用户37岁管过650人，投了被当overqualified，浪费机会 |
| 保险/金融培训 | 方向大幅偏离AI目标，且是保险圈套 |
| 餐饮/零售行业管理 | 非目标行业，经验不转移 |

**为什么是陷阱**：如果用户去面试L&D，面试官会问"你MSc AI学的东西以后不用了？"——面试都过不了。

**🔴 简历端也要同步防误判**：除了避开这类岗位，简历本身也不能出现让 ATS 误判为 HR 的关键词。
详见 `references/ats-keyword-optimization.md`（人才发展→营运负责人、慕课→SOP、培训→业务落地宣导）。

## 场景6: 求职平台资料填写

当用户在求职平台（JobsDB/CTgoodjobs/LinkedIn/OfferToday等）填写个人资料时，执行此场景。

### 风格要求（重要）

用户深夜填表时，**不要解释原因**，直接给可复制粘贴的值：
```
字段名 → 填什么
```
每一步截图确认后再继续下一步。不要一次性给完所有步骤。

### 平台优先级

| 平台 | 优先级 | 质量 | 备注 |
|------|--------|------|------|
| JobsDB | ★★★ 主力 | 高 | 中高端白领，ICT/MGMT类岗位多 |
| **Glassdoor HK** | ★★★ 主力 | 高 | 公司评价+薪资透明，调研公司底细首选。注册需WhatsApp，"贡献换访问"选Create a job alert。Cloudflare反爬不可自动化 |
| **简职HK** | ★★★ 主力 | 高 | 80%+中资/央企，双休多，不强制粤语（微信公众号，需手动查看） |
| CTgoodjobs | ★★☆ | 中高 | 平台体验好，岗位质量稳定，薪资透明 |
| LinkedIn | ★★☆ | 中高 | 科技/外企为主，需英文简历 |
| **劳工处** | ★★☆ | 中 | 政府/公营/大型企业，双休极多（www2.jobs.gov.hk） |
| 猎聘 | ★★☆ | 中高 | 中资覆盖最佳 |
| JIJIS | ★☆☆ | 中 | 应届/实习/不卡粤语，竞争小 |
| OfferToday | ★☆☆ | 低 | 算法匹配差，保险马甲多，资料填好放着 |

### 薪酬期望填写指南（2026-06-12 实测）

当申请表问「总薪酬期望」/「Total Compensation Expectation」时：

| 问题措辞 | 填什么 | 单位 |
|---------|--------|------|
| 总薪酬 / Total Compensation | **年薪**（含base+bonus+福利） | HKD/年 |
| 月薪 / Monthly Salary | 月base | HKD/月 |
| Expected Salary | 视上下文，通常指年薪 | HKD/年 |

**Manulife/宏利参考数据（2026-06-12 实测）**：
- Manager级（区域scope，AI/变革方向）：HKD XXX,XXX/年
- 月base约 60-70K × 13个月 + 绩效奖金2-4个月
- 如果只填一个数字：HKD 900,000
- 备注栏加：*"Flexible and open to discussion based on the full compensation package."*

**⚠️ Pitfall: 不要把年薪填成月薪**
用户第一次填Manulife时，agent建议了月base（60-75K），用户指出「总薪资应该年薪」。正确：HKD XXX,XXX/年。

### 用户核心技能列表（跨平台通用）

第一梯队：Operations Management, Business Development, Team Leadership, Digital Transformation, AI, NLP, Data Analytics
第二梯队：Key Account Management, Sales Strategy, SOP Design, P&L Management, Customer Experience, Marketing Strategy
第三梯队：Python, Deep Learning, RoBERTa, Sentiment Analysis, Data Pipeline, SQL

### 用户语言能力（跨平台通用）

| 语言 | 中文描述 | 英文描述 |
|------|---------|---------|
| 普通话 | 母語 | Native |
| 英语 | 良好工作能力 | Professional Working |
| 粤语 | 基础会话 | Conversational (Basic) |



## 场景9: 合并核实文档到平台文档

当用户创建了合并评审文档（如"V3人工核实版"）并说"把内容更新到LinkedIn/JobsDB文档里"/"合并进去"/"拆分V3"时，执行此场景。

**详细流程**：见 `references/job-doc-management.md`

**快速流程**：
1. 读取合并评审文档，按"来源"列分类（LinkedIn vs JobsDB）
2. 读取对应的平台源文档（`~/job-search/inbox/{平台}岗位扫货_*.md`）
3. 用 `patch` 逐条更新状态（降级/升级/信息更新），不要用 `write_file` 覆盖
4. 在每个平台文档末尾追加变更汇总表（独立section）
5. 更新"下一步建议"（划掉已确认条目）
6. 删除合并评审文档（临时中间产物）

**⚠️ Pitfall**：合并文档的"来源"标注可能不准，以平台源文档中是否存在该岗位为准。详见 `references/job-doc-management.md` P1。

### 场景9b: 收件箱扫描文件整合归档（反向操作）

当桌面有多份扫货文件（`~/job-search/inbox/{平台}岗位扫货_*.md`）需要整合到 `08-找工作/` 并清理桌面时，执行此流程。

**触发词**："整合到知识库"/"收件箱文件清理"/"合并去重"/"归档扫货文件"

**流程**：
1. **读取所有源文件**（桌面 + 08-找工作/），提取公司+岗位对
2. **去重**：以 `投递追踪.md` 为基准，用公司名前5字符+岗位名模糊匹配
3. **补漏**：将收件箱独有的有效岗位追加到 `投递追踪.md` 的岗位池（`## 面试记录` 之前插入）
4. **确认后删除**收件箱源文件（必须先完成步骤3，不可先删后补）

**去重规则**：
- 同一公司+同一岗位（不同来源如LinkedIn vs JobsDB）→ 保留条目更完整/匹配度更高的
- 同一公司不同岗位 → 都保留
- ⭐低匹配/EXCLUDED 标记的岗位 → 不补入追踪文件（已在扫描报告中筛掉的不值得追踪）
- 总报告中已有的P1-P4岗位 → 不重复补入（总报告是权威评分）

**⚠️ 关键Pitfall: 整合前必须查历史状态标记（2026-06-10教训）**
整合收件箱扫描文件时，**不能只做标题关键词匹配就直接复制粘贴**。每条待补入的岗位必须：
1. 先查 `投递追踪.md` 中是否已有该条目（去重）
2. 再查源扫货文件中该条目的**原始状态标记**（❌不投/❓待确认/🚫避坑 → 不得标为"待投"）
3. 最后用 `session_search` 查历史对话中是否对该岗位有过否决/降级决策
4. 只有**无历史否决记录 + 无状态标记**的岗位才能标为"待投"

**失败案例**：某区域总经理岗 SEA 在06-09已由用户明确否决（"无东南亚市场经验"），但06-10整合时因未查历史直接标为"待投 70%"，被用户当场批评"偷懒"。根因：只做了标题匹配（MiniMax + AI + GM → 70%），没有检查源文件的状态标记和session历史。

**⚠️ Pitfall: `read_file` 对 Windows 挂载路径返回空**
`read_file` 工具对 `~/job-search/inbox/` 等 Windows 挂载路径可能返回空内容（0 chars）。遇到此情况改用 `terminal cat` 读取。这是 WSL 文件系统挂载的一致性问题，不是文件本身的问题。

**⚠️ Pitfall: terminal heredoc 中的 `&` 字符**
向文件追加内容时，如果文本包含 `&`（如URL中的查询参数），heredoc（`cat >> file << 'EOF'`）会将 `&` 解释为后台执行符导致命令失败。改用 `patch` 工具或 `write_file` 追加含URL的内容。

## 场景13: P1岗位批量出简历（从场景12衔接）

当场景12汇总后用户说"去做2"（即先出P1定制简历），或直接说"出P1简历"/"把待出简历的搞定"时，执行此场景。

**流程：**
1. 读取追踪表P1 section，提取待出简历的岗位
2. 检查JD库（`job-search/JD库/`）是否有对应JD文件
   - 有 → 直接进入Step 4
   - 无 → curl抓取LinkedIn JD，保存到JD库
3. **⚠️ JD保存格式要求（2026-06-15 生效）**：
   - **英文原文**：保持原样，不修改
   - **中文翻译**：在英文原文下方添加 `---` 分隔线，然后添加中文翻译
   - **格式**：
     ```markdown
     # {公司} - {岗位}
     
     {英文JD原文}
     
     ---
     
     ## 中文翻译
     
     {中文翻译内容}
     ```
   - **翻译要求**：
     - 保留专业术语（如PM、BD、AI、SOP等）
     - 职位名称翻译（如"Business Development Director"→"商务拓展总监"）
     - 要求/资质部分逐条翻译
     - 保持段落结构与原文一致
   - **文件命名**：`{ID}_{公司}_{岗位}.md`（不变）
   - **更新追踪表**：保存后，更新追踪表的"JD链接"列
3. **⚠️ 必须在生成前重评匹配度**：用JD全文逐维度核实，不要盲信P1标签。之前核实的评分可能偏高（实测P1标65%的岗位，重新核实可能只有40-45%）。如果重评<55%，告知用户"这份简历生成后匹配度可能偏低，是否仍要出？"，不要生米煮成熟饭再告诉用户"其实匹配度很低"。
4. 按方向归类（同方向合并，跨方向分别定制）
5. 用`shutil.copy2` + `set_para_text()`生成定制简历
6. 更新追踪表状态（P1待出简历 → P1简历就绪✅）
7. 汇报时**必须附带每份简历的匹配度评估和主要硬伤**，让用户决定投不投

**⚠️ Pitfall: 不要生成简历后才说匹配度低**
用户信任P1标签（已核实≥60%）才让你出简历。如果生成后你说"其实只有40%"，浪费了时间和信任。核实前置，生成后置。

## 场景10: 批量岗位核实 + 多版本简历生成

当用户说"核实一下这些强推岗位"、"帮我做几份针对性简历"、"看看哪些合适"时，执行此场景。

**典型场景**：收件箱中有多份岗位扫货文档（`~/job-search/inbox/{平台}岗位扫货_{日期}.md`），里面标了"强推"的岗位需要二次核实，然后生成定制简历。

**完整流程**：

### Step 1: 读取所有岗位文档 + 简历

```
1. 读取 ~/job-search/inbox/ 下所有岗位扫货*.md
2. 提取所有标记为"强推"/"★★★★★"的岗位
3. 读取 V3 简历（确认用户画像）
```

### Step 2: 过滤虚假强推（关键步骤！）

**⚠️ 自动化扫描（Gmail scanner / LinkedIn API）的"强推"评级是关键词匹配，不是人岗匹配。** 必须用简历画像过滤：

**⚠️ 2026-06-10 实证**：13个★★★★★强推岗位经JD全文核实后，仅2个≥65%匹配（全部是BD方向）。8个此前标记75%的岗位核实后平均仅40%。关键词"AI"+"Product Manager"对无PM经验的转型者是系统性误判源。

**常见误判模式**：

| 用户画像 | 关键词命中 | 误判岗位类型 | 正确判断 |
|----------|-----------|-------------|---------|
| 管理+AI硕士 | "AI" | AI/ML Engineer, ML Lead, Data Scientist | ❌ 技术岗，不是管理岗 |
| 管理+AI硕士 | "Product Manager" | Binance PM, Crypto PM | ⚠️ 需看是否有PM经验 |
| 管理+AI硕士 | "Data" | Data Analytics Lead, Head of Data | ❌ 数据工程岗 |
| 管理+AI硕士 | "AI Product" | AI Product Manager (ERP) | ⚠️ 需看行业和经验要求 |

**过滤规则**：
1. 要求 hands-on ML/DL engineering（PyTorch training, model deployment）→ ❌ 砍
2. 要求 8年+ 深度技术背景（AI Research, Data Science）→ ❌ 砍
3. 要求特定行业经验（银行/保险/金融 mandatory）→ ❌ 砍
4. 要求特定语言（日语/韩语硬性）→ ❌ 砍
5. Manager级但薪资 ≤ 20K → ❌ overqualified
6. 职位含 Assistant/Trainee/Intern → ❌ overqualified

### Step 3: JD核实（对过滤后的岗位）— ⚠️ 必做，不可跳过

**获取JD全文**：
- LinkedIn：`curl -s "https://www.linkedin.com/jobs/view/{id}" | grep description`（无需登录，成功率~80%）
  - 详细命令和批量提取方法见 `references/linkedin-jd-extraction.md`
- JobsDB：Cloudflare保护，browser_navigate会超时，用 curl + User-Agent 尝试
- 核实不到的标注"❓待确认"，不要编造JD内容

**⚠️ 核心教训（2026-06-09）**：之前某媒体公司岗位只核实了"语言门槛"就标了🎯立即投，漏掉了"必须有媒体行业高级领导经验"这个硬性要求。核实必须逐项对照，不能只查单一维度。

**结构化核实清单（每项必须打分，不可跳过）**：

对每个待核实岗位，输出以下表格：

```
| # | 核实维度 | JD原文摘录 | 用户匹配情况 | 判定 |
|---|---------|-----------|-------------|------|
| 1 | 行业经验要求 | "在XX行业担任高级领导职务" | 无该行业经验 / 有相关经验 / 完全匹配 | ✅/⚠️/❌ |
| 2 | 核心职能要求 | "深入了解XX策划/购买/优化" | 无实操经验 / 有相关能力 / 直接匹配 | ✅/⚠️/❌ |
| 3 | 硬性年限要求 | "X年+XX经验" | 具体年限对比 | ✅/⚠️/❌ |
| 4 | 专业资质要求 | "持有XX证书/牌照" | 有/无 | ✅/❌ |
| 5 | 语言硬性要求 | "流利日语/韩语" | 具体语言能力对比 | ✅/⚠️/❌ |
| 6 | 技术栈要求 | "精通XX工具/平台" | 有实操 / 仅了解 / 完全不会 | ✅/⚠️/❌ |
| 7 | 管理层级匹配 | "Senior/Director/VP级" | 用户层级对比 | ✅/⚠️/❌ |
| 8 | 签证/永居要求 | "需香港永久居民" | IANG是否满足 | ✅/❌ |
| 9 | 目标区域市场经验 | "深耕东南亚/北美/欧洲市场" | 有该区域经验 / 无（仅中国大陆） | ✅/⚠️/❌ |
```

**维度9说明（2026-06-09教训）**：区域级岗位（Regional GM / Regional Director / Head of XX Asia）要求对目标市场有深度认知（本地商业文化、Go-to-Market策略、客户网络）。用户仅有中国大陆跨区域经验，对东南亚/北美/欧洲市场零经验。当JD明确指定非中国大陆区域时，维度9为决定性维度（与维度1、2同等权重）。

**判定规则**：
- 维度1（行业经验）、维度2（核心职能）和维度9（目标区域市场经验）是**决定性维度**，任一为❌ = 该岗位总评直接降至❌
- 维度3-8为**辅助维度**，❌不直接否决但降低评分
- **总评计算**：
  - 维度1+2均✅，辅助维度无❌ → 🎯可投
  - 维度1+2均✅，辅助维度有❌ → ⚠️有条件投（说明需要补什么）
  - 维度1或2为⚠️ → ❓待定（需用户判断是否有隐性匹配）
  - 维度1或2为❌ → ❌不投（一句话说明硬伤）

**⚠️ 防止"包装好的简历掩盖匹配度不足"**：简历的措辞优化（如把"门店管理"包装成"增长体系搭建"）不等于岗位匹配。核实必须基于**用户实际经历**，不是基于简历定制后的文字。

### Step 4: 分类决策

```
✅ 合适 + 可投 → 生成定制简历
❓ 待确认     → 列出，等用户看JD
❌ 不合适     → 简要一句话说明原因（必须指出具体是哪个核实维度❌）
🚫 已避坑     → 之前已确认不投的（注明具体原因+核实日期）
```

**⚠️ 红线**：未经Step 3结构化核实的岗位，不得标记为"🎯立即投"。只能标"❓待核实"。

### Step 5: 批量生成定制简历

对"合适"的岗位，按方向归类，每类生成一份。**替换深度取决于方向差异大小**：

| 方向差异 | 替换深度 | 改什么 |
|---------|---------|--------|
| 同方向多公司（3个BD岗） | 浅层 | 求职目标 + 个人优势摘要 |
| 跨方向（AI推行 vs BD vs 战略合作） | 深层 | 上述 + AI项目bullet + 技术栈排序 + 工作经历bullet（中英文全改） |

**深层定制时**，每个段落的改写需对齐目标JD的关键词。详见 `references/multi-version-resume-workflow.md` 的「深层定制模式」和「JD驱动改写策略」。

**归类原则**：不同公司的同一类岗位（如"品牌营销"）用同一份简历，不需要一司一简历。但不同方向（如BD vs AI落地推行）必须分别出定制版。

**命名规范**：`你的姓名的简历_2026_V3-{公司简称}-{岗位方向}.docx`
示例：`你的姓名的简历_2026_V3-公司A-方向A.docx`

### Step 6: 输出核实报告

```
══════════════════════════════════════
  强推岗位核实报告 — {日期}
══════════════════════════════════════

## 一、过滤掉的虚假强推
（表格：岗位 | 公司 | 砍掉原因）

## 二、确认合适的岗位
（表格：# | 岗位 | 公司 | 来源 | 状态 | 对应简历文件名）

## 三、待确认岗位
（表格：# | 岗位 | 公司 | 需确认事项）

## 四、已投/已避坑（不动）
（简要列表）

## 五、下一步建议
（按优先级排列的投递建议）
```

---

## 人工确认红线

- 简历发送前必须等用户确认
- 投递提交前必须等用户确认
- 不自动打开链接、不预填表单、不点击提交

---

## 附录: Skill 发布到 GitHub

脱敏 + README + .gitignore + gh repo create 的完整流程，见 `references/github-publish-workflow.md`。