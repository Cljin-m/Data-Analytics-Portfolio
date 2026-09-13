# Cljin-m｜Data Analytics Portfolio

数据分析 / 商业智能方向作品集，重点展示 SQL 分析、Python 数据分析、实验评估、用户分层和 Dashboard 表达能力。当前内容来自本工作区中已经完成的 3 个项目，README 中的技术栈、数据规模、字段和结论均依据现有 Notebook 与 Markdown 报告整理。

## 关于我

我关注从业务问题出发的数据分析工作，包括用户行为、转化、留存、客户价值和运营决策支持。当前作品集展示了 Python 分析、SQL 指标构建、Tableau 看板和 A/B 测试统计评估等能力。

## 核心能力

**数据分析**  
Python · Pandas · NumPy · Jupyter Notebook · EDA · 数据质量检查

**业务分析**  
用户分层 · RFM · 留存分析 · 复购分析 · 转化漏斗 · ROI 情景模拟

**实验分析**  
A/B Testing · 样本量规划 · 分流比例检验 · Proportion Z-Test · 置信区间 · 国家维度下钻

**SQL 与 BI**  
MySQL · 条件聚合 · View · Self Join · Window Functions · Tableau Dashboard

## 代表项目

| 项目 | 类型 | 核心问题 | 关键技术 | 入口 |
| --- | --- | --- | --- | --- |
| Customer Segmentation with RFM-I | Python 用户分析 | 如何在经典 RFM 之外引入浏览意向和收入信息，识别更适合运营触达的用户群体？ | RFM-I 评分、用户标签、分层画像、ROI 情景模拟 | [查看项目](https://github.com/Cljin-m/Data-Analytics-Portfolio/tree/main/customer-segmentation-rfmi) |
| Conversion Page A/B Test | 实验分析 | 新版转化页是否显著提升用户转化率？ | 数据清洗、样本量规划、卡方检验、单侧比例 Z 检验、交互效应检验 | [查看项目](https://github.com/Cljin-m/Data-Analytics-Portfolio/tree/main/conversion-ab-test) |
| Beauty E-commerce SQL + Tableau Analysis | SQL / BI 分析 | 美妆电商用户在活跃、转化、留存和商品表现上有哪些关键模式？ | MySQL 聚合、转化漏斗、留存、RFM、Tableau Dashboard | [查看项目](https://github.com/Cljin-m/Data-Analytics-Portfolio/tree/main/ecommerce-sql-tableau-analysis) |

## 项目组织方式

本仓库没有把每个 Notebook 或报告拆成单独仓库，而是采用“能力领域 → 代表项目”的方式组织：

```text
README.md
customer-segmentation-rfmi/
conversion-ab-test/
ecommerce-sql-tableau-analysis/
MySQL&Tableau/
ecommerce-user-segmentation.ipynb
page_ab_test.ipynb
```

保留的原始分析文件：

```text
ecommerce-user-segmentation.ipynb
page_ab_test.ipynb
MySQL&Tableau/
```

## 数据说明

当前工作区不包含 Notebook 引用的原始 CSV 文件。各项目 README 只记录 Notebook 和报告中能够验证的数据路径、字段、方法和输出，不声明仓库已包含完整原始数据。

## 说明

本作品集只展示当前公开文件能够支撑的能力，不额外虚构数据来源、数据规模、模型效果、自动化 Pipeline 或业务结果。
