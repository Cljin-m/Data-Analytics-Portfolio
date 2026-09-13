# Conversion Page A/B Test Analysis

## 项目概览

本项目评估新版电商转化页相对于旧版页面是否显著提升用户转化率。分析过程包括实验数据清洗、分流质量检查、最小样本量估计、统计显著性检验和国家维度下钻，最终给出是否上线新版页面的判断。

## 业务 / 分析问题

- 实验数据是否满足 A/B 测试分析的基本质量要求？
- 对照组和实验组的用户分流是否均衡？
- 新版页面是否显著提升用户转化率？
- 新版页面效果在 CA、UK、US 三个国家用户中是否存在差异？
- 基于当前实验结果，是否应全量上线新版页面？

## 数据集 / 数据来源

Notebook 标注的数据来源为 Kaggle 公开数据集：

```text
https://www.kaggle.com/datasets/putdejudomthai/ecommerce-ab-testing-2022-dataset1
```

Notebook 读取的数据文件为：

```text
data/ab_data.csv
data/countries.csv
```

当前工作区未包含上述原始 CSV 文件。Notebook 输出可验证的信息包括：

- `ab_data`：294,480 行，5 列。
- `countries`：290,586 行，2 列。
- `ab_data` 展示字段：`user_id`、`timestamp`、`group`、`landing_page`、`converted`。
- `countries` 展示字段：`user_id`、`country`。

## 方法

- 检查样本规模、实验分组、实际访问页面和重复用户。
- 删除 3,893 条实验分组与实际页面不匹配的记录。
- 按 `user_id` 去重，避免同一用户重复计入统计分析。
- 使用以下实验规划参数估算最小样本量：
  - 历史基准转化率：12%
  - 相对 MDE：5%
  - 目标转化率：12.6%
  - 显著性水平：5%
  - 统计功效：80%
  - 单侧检验
- 使用卡方检验检查分流比例和国家分布。
- 对整体转化率执行单侧双样本比例 Z 检验。
- 计算转化率置信区间。
- 对 CA、UK、US 三个国家进行转化率下钻。
- 使用 Logistic Regression 的似然比检验判断组别与国家是否存在交互效应。

## 技术栈

- Python
- Pandas
- NumPy
- SciPy
- Statsmodels
- Matplotlib
- Jupyter Notebook

## 关键发现

- Notebook 报告实验分组与实际页面不匹配的记录数为 3,893；清洗后存在 2 个重复用户。
- 国家构成未发现显著差异，国家分布卡方检验 p-value 为 0.408431。
- 新版页面转化率为 11.88%，低于旧版页面的 12.04%。
- 单侧比例 Z 检验 p-value 为 0.9052，未达到 0.05 的显著性水平。
- 国家交互效应检验 p-value 为 0.291454，Notebook 结论为当前没有充分证据说明新版页面效果因国家不同而不同。
- 基于 Notebook 总结，当前数据不支持“新版页面显著提升转化率”的假设，不建议直接全量上线。

## 项目结构

```text
conversion-ab-test/
└── README.md
```

当前工作区中的原始 Notebook：

```text
../page_ab_test.ipynb
```

## 如何运行

将数据文件放置在以下路径后，打开 [`page_ab_test.ipynb`](../page_ab_test.ipynb)，按 Notebook 单元格顺序执行：

```text
data/ab_data.csv
data/countries.csv
```

## 输出

- 清洗后的实验数据。
- 样本量和统计功效规划结果。
- 分流比例检验结果。
- 整体转化率对比。
- Z 检验与置信区间结果。
- 国家维度转化率表。
- 国家转化率对比图。
- Logistic Regression 交互效应检验结果。
