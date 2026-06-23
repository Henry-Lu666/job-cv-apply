# LinkedIn 个人资料优化指南

> 2026-06-03 首次整理。用户已完成首轮优化。

## 表单填写风格

与 JobsDB 一致：不要解释原因，直接给可复制粘贴的值。格式：
```
字段名 → 填什么
```

## Headline（标题）

**位置**：名字下方，猎头搜索结果里第一眼看到的内容。

**规则**：
- **纯英文**（LinkedIn 搜索单语言匹配，中英混写两边都抓不到）
- **不用放签证状态**（IANG 等放 About 区域，LinkedIn 有专门的 Work Authorization 筛选器）
- **4 个短句，每句一个搜索维度**

**推荐格式**：
```
Operations & BD Leader | 12yr NEV Industry (某大型汽车制造企业) | MSc Applied AI | Digital Transformation · Hong Kong
```

**⛔ 不要**：
- "新能源汽车产业专家" → 太泛，猎头不搜这个词
- 中英文混写
- 把签证有效期放标题里（浪费关键词空间）

## Skills（技能）

LinkedIn 免费版最多 50 个技能，但初始建议先加 5 个核心定位。

**已添加的 5 个（2026-06-03）**：
1. 运营管理 (Operations Management)
2. 业务拓展 (Business Development)
3. 团队领导 (Team Leadership) — ⚠️ 用"领导"不用"领导力"，LinkedIn 技能标签是名词
4. 数字化转型 (Digital Transformation) — ⚠️ 用"转型"不用"改造"，行业通用词
5. 人工智能 (Artificial Intelligence)

**后续待加**（免费版额度允许时扩充）：
- 数据分析 (Data Analytics)
- 大客户管理 (Key Account Management)
- 损益管理 (P&L Management)
- Python / NLP / Deep Learning / Sentiment Analysis
- 新能源汽车 (New Energy Vehicles)
- 渠道管理 (Channel Management)
- 扭亏为盈 (Turnaround Management)

**置顶技能**（Top Skills）：前 5 个即置顶，对搜索排名影响最大。

**⛔ ATS 红线词不能出现在技能里**：
- ~~培训 / Training / L&D~~
- ~~人才发展 / Talent Development~~
- ~~讲师 / Trainer~~

## About（个人简介）

**结构**：定位句 → 经历概述 → 差异化 → 求职目标 → 联系方式

**开头加一句定位总结**（LinkedIn 阅读习惯：先看"你是谁"再看"你做了啥"）：
```
Operations & Business Development leader with 12 years in China's NEV sector, now pivoting into AI-powered business transformation.
```

**签证表述**（英文标准写法）：
```
Eligible to work in Hong Kong (Work visa valid)
```
不要写 "(有效工作签证)"，HR 看不懂中文。

**核心差异化金句（已验证有效）**：
```
Core differentiator: not "business-savvy + AI tool user", but "business-native + capable of building models independently."
```

## Projects（项目）

**排序规则**：猎头扫一眼看第一条，放最硬的经历。按以下顺序：

| 顺序 | 项目 | 理由 |
|------|------|------|
| 1 | MSc Thesis: NLP for Business Analysis | 学术硬通货 + AI 差异化 |
| 2 | 佛山直营店扭亏 | 量化数据最硬（62台/117%/85%） |
| 3 | 全国直营体系扩张 | 650人规模化，管理深度 |
| 4 | 移动机器人 ToB | BD 能力佐证 |
| 5 | 用户洞察 15,000+ | 数据驱动运营 |
| 6 | 个人作品集网站 | 收尾给链接，对猎头价值最低 |

**⛔ ATS 红线词在 Projects 中也要清洗**（与简历一致）：

| 高危词 | 替换为 |
|--------|--------|
| MOOC-style learning platform | Standardized operational SOPs |
| internal trainer system | Frontline enablement system |
| training programs | Business rollout sessions |
| talent development | National direct-sales operations |

**移动机器人 ToB 项目补充建议**（原版太单薄）：
```
Mobile Robot — ToB Business Development (2024.08 – 2025.03)
Explored B2B market entry for commercial mobile robotics in Greater Bay Area.
• Mapped 20+ leads across logistics, hospitality, and manufacturing sectors
• Advanced 3 qualified accounts to solution-design stage with customized proposals
• Outcome: validated product-market fit signals for hospitality vertical
```

**薪酬与晋升**：不要单独一条，内容并入"全国直营体系扩张"即可。

## LinkedIn 岗位评估流程

当用户分享 LinkedIn 推荐岗位时：

1. **提取岗位信息**：LinkedIn 是 SPA 站点，浏览器工具打不开。用 curl 抓 meta 标签：
   ```bash
   curl -sL "https://www.linkedin.com/jobs/view/{JOB_ID}" \
     -H "User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36" \
     | grep -oP '<title>(.*?)</title>|og:title" content="(.*?)"|og:description" content="(.*?)"'
   ```
   可获取：公司名、岗位名、地点、一句话描述。完整 JD 需要用户截图。

2. **逐岗位评估**（5 星制）：对照用户画像 + 真实约束打分
   - ⭐⭐⭐⭐⭐ 80-100%：强投
   - ⭐⭐⭐⭐ 70-79%：推荐投
   - ⭐⭐⭐ 60-69%：可投
   - ⭐⭐ 40-59%：观望
   - ⭐ <40%：不投

3. **评估维度**：
   - 行业匹配度（汽车/科技 vs 金融/医药/食品）
   - 职能匹配度（运营/BD/数字化 vs 纯销售/纯技术）
   - 级别匹配度（管理岗 vs 初级岗 overqualified 风险）
   - 语言要求（纯英语/纯粤语 vs 中英双语）
   - 公司规模和质量（大厂/中资 vs 小微企业）

4. **输出格式**：
   ```
   | 公司 | 岗位 | 匹配度 | 判断 |
   |------|------|--------|------|
   | ...  | ...  | ⭐⭐⭐ | 可投 |
   ```
   然后逐个分析匹配点和风险点，最后给投/观望/不投建议。

5. **投简历需要完整 JD**：meta 标签信息不够做简历微调，需用户在 LinkedIn 登录状态截图完整 JD。