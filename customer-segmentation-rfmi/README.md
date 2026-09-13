# Customer Segmentation with RFM-I and ROI Simulation

## 项目概览

本项目基于用户消费、活跃和浏览行为构建 RFM-I 用户价值分层框架。项目在经典 RFM 的基础上引入浏览意向、决策摩擦和收入层级，并通过 ROI 情景模拟对比传统 RFM 触达策略与优化后的 RFM-I 触达策略。

## 业务 / 分析问题

- 哪些用户在最近活跃、购买频率和消费金额上具有较高历史价值？
- 哪些用户具备较高浏览意向或潜在价值，但没有被经典 RFM 充分识别？
- 在固定预算假设下，哪些用户群体更适合作为优惠券触达对象？
- 传统 RFM 与优化 RFM-I 策略在模拟边际 ROI 上有何差异？

## 数据集 / 数据来源

Notebook 引用的数据文件路径为：

```text
data/user_personalized_features.csv
```

当前工作区未包含该原始 CSV 文件。

Notebook 输出可验证的数据规模为 `1000 行 x 14 列`，缺失值总数为 0，重复用户数为 0，年龄异常值为 0。Notebook 中展示的字段包括：

- `User_ID`
- `Age`
- `Gender`
- `Location`
- `Income`
- `Interests`
- `Last_Login_Days_Ago`
- `Purchase_Frequency`
- `Average_Order_Value`
- `Total_Spending`
- `Product_Category_Preference`
- `Time_Spent_on_Site_Minutes`
- `Pages_Viewed`
- `Newsletter_Subscription`

## 方法

- 对必要字段、缺失值、重复用户和年龄异常值进行数据质量检查。
- 构建用户行为特征：
  - `I_Score`：基于标准化站内停留时长和浏览页数计算浏览意向分。
  - `Friction`：使用浏览页数除以购买频率加一，识别“浏览较多但购买较少”的用户。
  - `Income_Level`：按收入的 33% 和 66% 分位点划分低、中、高收入层级。
- 对消费、收入、活跃、浏览行为、购买频率和高摩擦用户进行 EDA。
- 基于最近活跃、购买频率和总消费构建经典 RFM 评分。
- 结合 RFM 分层、意向分、摩擦系数和收入层级生成营销分层标签。
- 输出用户数量、分层画像和 R/F/M/I 雷达图。
- 通过 ROI 情景模拟比较传统 RFM 策略和优化 RFM-I 策略。

## 技术栈

- Python
- Pandas
- NumPy
- Matplotlib
- Jupyter Notebook

## 关键发现

- Notebook 识别出 15 类最终营销分层，包括 `核心VIP`、`重要价值用户`、`高潜流失用户`、`犹豫型潜力用户`、`高意向待转化用户` 和 `高潜沉睡用户`。
- 输出结果中用户数最多的分层为 `重要价值用户`，共 135 人；其次为 `一般保持用户` 129 人和 `一般价值用户` 128 人。
- ROI 情景模拟中，传统 RFM 策略触达 200 人，模拟边际 ROI 为 4.04%。
- 优化 RFM-I 策略触达 274 人，其中核心用户 149 人、新挖掘潜力用户 125 人，模拟边际 ROI 为 18.47%。
- Notebook 明确说明 ROI 结果属于基于假设参数的情景模拟，真实业务增益仍需通过正式 A/B 实验验证。

## 项目结构

```text
customer-segmentation-rfmi/
├── README.md
└── customer-segmentation-rfmi.ipynb
```

当前工作区中的原始 Notebook：

```text
customer-segmentation-rfmi.ipynb
```

## 如何运行

将数据文件放置在以下路径后，打开 [`customer-segmentation-rfmi.ipynb`](customer-segmentation-rfmi.ipynb)，按 Notebook 单元格顺序执行：

```text
data/user_personalized_features.csv
```

Notebook 会创建 `outputs` 目录，并将 ROI 对比图保存至：

```text
outputs/ROI_result/roi_comparison.png
```

## 输出

- 用户数据质量检查结果。
- 用户衍生特征：`I_Score`、`Friction`、`Income_Level`。
- 标准 RFM 分层汇总。
- 营销扩展标签汇总。
- 用户分层可视化。
- ROI 对比表和 `roi_comparison.png`。
