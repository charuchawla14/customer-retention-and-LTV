# Customer Retention & Lifetime Value Analytics

## 📌 Executive Summary

This project analyzes 800K+ transactions from a UK-based online retailer (Online Retail II, Dec 2009–Dec 2011) to identify which customers drive the business, which are quietly churning, whether that churn pattern is statistically real, and how well it can actually be predicted.

**Headline Findings:**
- **Champions (22.1% of customers) generate 68.35% of total revenue** — extreme revenue concentration in a small, loyal segment
- **"At Risk" customers — despite being just 3.86% of the customer base — carry a statistically significant 3x higher average value (£4,488) than Regular customers (£1,471)**, confirmed via Welch's t-test (t=5.09, **p<0.001**), ruling out random chance despite the segment's small size
- A logistic regression churn model achieves **73% accuracy** and catches **68.75% of churners by headcount** — but critically, **captures only 27.3% of the revenue actually at risk**, systematically missing the business's highest-value churners
- **Purchase frequency is ~21x more predictive of churn than total spend**, suggesting re-engagement (driving another purchase) matters more for retention than increasing order size

➡️ **Recommendation**: Do not rely on the churn model alone to prioritize retention spend. Because the model under-flags high-Monetary customers who are actually churning, it should be paired with a manual, value-weighted override — any customer above a defined lifetime-value threshold should be reviewed for retention risk regardless of model output, since a single missed high-value churner outweighs several correctly-caught low-value ones.

---

## 🎯 Business Objectives

- Segment customers by value and engagement using RFM (Recency, Frequency, Monetary) analysis
- Statistically verify whether differences between customer segments are real or attributable to chance
- Build a churn prediction model using purchase behavior, and honestly evaluate where it succeeds and fails
- Quantify how much revenue is actually at risk from churn — not just how many customers
- Translate all findings into a specific, actionable retention recommendation

---

## 🛠️ Tools & Technologies

| Tool | Purpose |
|---|---|
| Python (pandas, NumPy) | Data cleaning, RFM calculation |
| SciPy (`scipy.stats`) | Hypothesis testing (Welch's t-test) |
| Scikit-learn | Logistic regression, train/test split, evaluation metrics |
| Matplotlib | Visualization (revenue trends, model evaluation) |
| Jupyter Notebook | Analysis environment |

---

## 📂 Dataset

**Online Retail II** — UK-based online retailer, December 2009–December 2011 
[https://www.kaggle.com/datasets/mashlyn/online-retail-ii-uc](url)
Columns used: `Invoice`, `InvoiceDate`, `Customer ID`, `Quantity`, `Price`, `Country`

---

## 🧹 Data Cleaning & Assumptions

| Step | Rows Affected |
|---|---|
| Original dataset | 1,067,371 |
| Missing Customer ID removed | −243,007 |
| Cancelled orders & negative/zero Quantity/Price removed | −18,815 |
| **Final clean dataset** | **805,549 (75.5% retained)** |

**Rationale:** Rows missing a Customer ID cannot be attributed to any individual for RFM analysis and were dropped. Cancelled orders (Invoice numbers prefixed "C") and rows with zero/negative Quantity or Price represent returns or data errors rather than genuine purchases, and were removed to avoid distorting Monetary calculations.

**Post-cleaning snapshot:** 5,878 unique customers, 36,969 unique orders, £17,743,429.18 total revenue, spanning Dec 1, 2009 – Dec 9, 2011.

---

## 📊 Step 1: Exploratory Data Analysis

Monthly revenue shows clear holiday seasonality, with sharp peaks each **October–November** (2010 and 2011) followed by a steep drop-off — consistent with pre-Christmas retail demand. The 2011 peak (~£1.65M) exceeded the 2010 peak (~£1.4M), indicating year-over-year growth rather than a repeating flat pattern.

---

## 📊 Step 2: RFM Segmentation

Every customer was scored 1–5 on Recency, Frequency, and Monetary value (quintile-based), then grouped into five business segments.

| Segment | Customers | % of Customers | Total Revenue | % of Revenue | Avg. Revenue/Customer |
|---|---|---|---|---|---|
| **Champions** | 1,300 | 22.12% | £12,128,115.56 | **68.35%** | £9,329.32 |
| Regular/Needs Attention | 2,633 | 44.79% | £3,874,165.07 | 21.83% | £1,471.39 |
| **At Risk** | 227 | 3.86% | £1,018,866.68 | 5.74% | **£4,488.40** |
| New/Promising | 443 | 7.54% | £394,638.61 | 2.22% | £890.83 |
| Lost/Hibernating | 1,275 | 21.69% | £327,643.25 | 1.85% | £256.98 |

**Key Insight:** Champions represent under a quarter of customers but generate more than two-thirds of all revenue. More notably, **At Risk customers — the smallest segment by headcount — have the second-highest average value per customer**, nearly 5x higher than Lost/Hibernating and 3x higher than Regular. This segment holds proven historical value and is actively disengaging, making it the single highest-priority group for retention intervention.

---

## 📊 Step 3: Hypothesis Testing

**Question:** Is the value gap between At Risk and Regular customers statistically real, given At Risk's small sample size (227 customers)?

**Method:** Welch's t-test (used instead of a standard t-test due to unequal sample sizes and variances between the two groups)

| Metric | At Risk | Regular/Needs Attention |
|---|---|---|
| Mean Monetary Value | £4,488.40 | £1,471.39 |
| T-statistic | 5.09 | |
| P-value | **0.000001** | |

**Finding:** The probability of observing a gap this large by random chance alone is roughly 1 in 1,000,000. This confirms the At Risk segment's elevated value is a genuine pattern, not a statistical artifact of its small size — providing solid justification for prioritizing this segment despite it being the smallest group in the customer base.

---

## 📊 Step 4: Churn Prediction Model

**Churn definition:** No purchase in the last 180 days (Recency > 180) → Churned. Overall churn rate: **40.83%**.

**Features used:** Frequency and Monetary only. Recency was deliberately excluded, since Churn is defined directly from Recency — including it would cause the model to simply restate the labeling rule rather than genuinely predict behavior (a data leakage risk).

**Model:** Logistic Regression (features standardized via `StandardScaler`), 80/20 train/test split (4,702 train / 1,176 test), stratified by churn status.

| Metric | Value |
|---|---|
| Accuracy | 73.13% |
| Precision (Churned) | 66.53% |
| Recall (Churned) | 68.75% |

**Confusion Matrix:**
|  | Predicted Active | Predicted Churned |
|---|---|---|
| **Actual Active** | 530 | 166 |
| **Actual Churned** | 150 | 330 |

**Feature Importance:** Frequency (coefficient: −3.87) is roughly **21x more influential** than Monetary (coefficient: −0.18) in predicting churn. This suggests purchase habit (how often a customer buys) is a far stronger churn signal than spending power — implying re-engagement campaigns (prompting another purchase) may be more effective than upsell-focused retention tactics.

---

## 📊 Step 5: Revenue-at-Risk — The Critical Finding

Evaluating the model by **customer count** alone paints an incomplete picture. Weighting the same test-set results by actual **revenue** reveals a serious blind spot:

| | Customers | Revenue |
|---|---|---|
| Churners correctly identified | 68.75% | **27.3%** (£158,807.75) |
| Churners missed | 31.25% | **72.7%** (£422,234.35) |

**Why this happens:** Monetary value has a negative relationship with churn in the model — high past spending predicts *lower* churn probability. This creates a systematic blind spot: when a genuinely high-value customer does churn, their historical spend biases the model toward predicting them as "still active," causing exactly the customers who matter most to be under-flagged.

**Business Implication:** A model that looks solid at 68.75% recall is, in financial terms, catching only about a quarter of the revenue genuinely at risk. This model should **not** be used in isolation to guide retention spend — it requires a value-weighted manual override, flagging any customer above a defined lifetime-value threshold for review regardless of the model's prediction.

---

## 💡 Key Insights Generated

- Champions (22.1% of customers) generate 68.35% of total revenue — extreme, concentrated value
- At Risk customers hold a statistically confirmed (p<0.001) 3x higher average value than Regular customers, despite being the smallest segment
- The churn model's 73% accuracy and 68.75% recall look solid by customer count, but capture only 27.3% of at-risk revenue — a significant, easily overlooked blind spot
- Purchase frequency is ~21x more predictive of churn than monetary value, pointing toward re-engagement over upselling as the more effective retention lever
- A model-only retention strategy would systematically under-protect the business's most valuable at-risk customers; a value-weighted override is recommended alongside the model

---

## 🏗️ Project Structure

```
customer-retention-analytics/
│
├── 01_Data_Cleaning.ipynb
├── 02_EDA.ipynb
├── 03_RFM_calculation.ipynb
├── 04_Hypothesis_testing.ipynb
├── 05_Churn_Prediction.ipynb
│
├── online_retail_II.csv
├── cleaned_data.csv
├── customer_rfm_segments.csv
│
└── README.md
```

---

## 🚀 Conclusion

This project moves beyond a standard churn-prediction exercise by testing every finding against a follow-up question: *is this real, and is this actually useful?* RFM segmentation surfaced a concentrated revenue base; hypothesis testing confirmed the At Risk segment's elevated value wasn't a statistical fluke; and the churn model, rather than being reported at face value, was stress-tested by revenue rather than headcount — revealing that a seemingly strong 73%-accuracy model actually fails at the one thing that matters most financially. The resulting recommendation — pairing the model with a value-weighted manual override — reflects the kind of applied, skeptical analytical judgment expected of a data analyst working with real, imperfect business data.

## 👨‍💻 Skills Demonstrated

- Python: pandas, NumPy for data cleaning and feature engineering
- Statistical hypothesis testing (Welch's t-test) and correct interpretation of p-values
- RFM customer segmentation
- Logistic regression: training, evaluation, and coefficient interpretation
- Recognizing and avoiding data leakage
- Model limitation analysis — evaluating by business impact (revenue), not just standard ML metrics
- Business recommendation writing grounded in statistical and financial evidence
