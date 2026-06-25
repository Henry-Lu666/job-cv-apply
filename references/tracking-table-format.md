# 投递追踪表格式规范

> 2026-06-24 验证

## P1/P2表格必须包含的列

```
| # | 公司 | 岗位 | 方向 | 匹配度 | LinkedIn | JD | 材料 | 动作 |
```

## 链接格式

**LinkedIn列**：`[LinkedIn](https://www.linkedin.com/jobs/view/XXXXX)`

**JD列**：`[📄JD+中文](JD库/XXXXX_公司_岗位.md)`
- ⚠️ 路径是 `JD库/` 不是 `../JD库/`（投递追踪.md和JD库在同一目录下）
- Obsidian相对路径从文件所在目录算起

## 岗位状态流转

```
未核实(待抓JD) → 已核实(P1/P2/❌) → 已投递(移入投递记录) → 反馈(✅/❌/💀枪毙)
```

**规则**：已投递的岗位必须从岗位池(P1/P2)移除，只在投递记录中跟踪。

## JD文件规范

**命名规则（2026-06-25验证）**：
- 格式：`{LinkedIn_ID}_{Company-Hyphenated}_{Role-Hyphenated}.md`
- 公司名和岗位名的单词之间用连字符 `-` 连接
- 正确：`4417121632_Madison-Pearl_AI-Advisory-Consulting.md`
- 错误：`4417121632_MadisonPearl_AI-Advisory-Consulting.md`（无连字符，Obsidian链接断裂）
- 生成后立即验证追踪表链接可点击

每个JD文件必须包含：
1. 英文原文（从LinkedIn抓取）
2. `## 中文翻译` 章节（岗位/职责/要求的中文对照）
3. `⚠️ 匹配评估` 行（核心三维得分+结论）

## 子Agent Gmail扫描流程

主agent不直接做Gmail扫描，而是：
1. 派子agent扫描邮箱+提取岗位+去重追踪表
2. 主agent检查子agent输出（/tmp/gmail_new_positions.md）
3. 将结果存入 `JD库/Gmail新岗位_日期.md`
4. 派子agent抓JD做9维度评估
5. 主agent检查结果并更新追踪表

## 三方向搜索策略

用户甜蜜区有3个方向，搜索/核实岗位时**不要只限BD**：
- ④BD/战略合作（60-70%）
- AI落地推行/变革管理（60-70%）
- ②数字化转型（55-65%）

实测发现：BD方向Hunter 60-65%岗位核实后多为垂直行业销售（金融基金/奢侈品/Web3），反而数字化方向匹配度更高。
