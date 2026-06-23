# 毕设技术描述（简历标准版）

> 最后更新: 2026-06-22
> ⚠️ 生成简历时必须引用此文件，不要自行改写四头名称

## 中文版

### 技术方案
基于预训练模型构建双任务四头架构（产品BIO + 产品情感分类 + 营销BIO + 营销洞察多标签），处理 8 类产品标签与 6 类营销洞察标签的 BIO 标注

### 核心成果
F1从基线大幅提升（3.2×），消融实验 ×5 种种子验证稳定性，独立测试集 47 条盲评

### 商业价值
输出品牌对比分析（宝骏 vs 五菱），识别口碑拐点与营销活动因果关联，验证小样本场景 AI 可落地性

---

## English Version

### Technical Solution
Built a dual-task four-head architecture based on a pre-trained model (Product BIO + Product Sentiment Classification + Marketing BIO + Marketing Insight Multi-label), handling BIO annotation for 8 product tags and 6 marketing insight tags

### Core Results
Improved F1 from baseline significantly (3.2× improvement). Validated stability through ablation studies with 5 random seeds; independent test set of 47 samples for blind evaluation

### Business Value
Delivered brand benchmarking analysis (Baojun vs. Wuling), identified reputation inflection points and causal links to marketing campaigns, validating AI feasibility in small-sample scenarios

---

## 四头架构说明

| 头 | 任务类型 | 标签数 | 说明 |
|---|---------|--------|------|
| Head 1 | 产品BIO序列标注 | 8类 | 产品方面抽取（BIO格式） |
| Head 2 | 产品情感分类 | 多类 | 针对产品方面的情感判断 |
| Head 3 | 营销BIO序列标注 | 6类 | 营销洞察方面抽取（BIO格式） |
| Head 4 | 营销洞察多标签 | 多标签 | 营销洞察分类（可多选） |

---

## ⚠️ 曾出错记录

- 2026-06-22: 误写为"方面抽取 + 情感分类 + 营销洞察多标签 + 槽位填充"，用户纠正
- 正确描述：**产品BIO + 产品情感分类 + 营销BIO + 营销洞察多标签**
