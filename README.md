# Customer Value & Churn Analytics: RFM Segmentation for a UK Gift Retailer

**Stack:** SQL Server · Python (pandas) · Power BI · DAX
**Domain:** E-commerce / Customer Retention Analytics

> **Note:** "Giftware Co." is a fictional company used for this portfolio project. All figures come from a public, anonymized dataset and do not represent any real business.

---

## 1. Executive Summary

A UK online gift retailer lacks a structured view of customer value, so retention activity is reactive and untargeted. This project builds a pipeline from raw transaction data to an interactive dashboard that:

- Identifies the customers who drive the most revenue
- Flags customers at risk of lapsing
- Recommends a distinct action for each customer segment

**Headline results**

| Metric | Result |
|---|---|
| Champions' share of customers | ~22% |
| Champions' share of revenue | ~68% |
| Overall churn (90-day inactivity) | 50.86% |
| Customers segmented | 5,881 |
| Seasonality | Pronounced Q4 revenue peak |

Revenue is heavily concentrated in a small customer group, and half the base is inactive. Both support a shift from broad campaigns to segment-level retention.

---

## 2. Business Context

**Problem:** Leadership cannot see which customers matter most or which are slipping away.

**Questions the analysis answers**

1. Who are the highest-value customers, and what share of revenue do they represent?
2. Which customers have gone inactive and are at risk of churning?
3. How do customer value and retention change by month, quarter, and year?
4. How do value and churn differ across countries?
5. What action should the business take for each segment?

I wrote a Business Requirements Document first, defining scope, metric definitions, and success criteria. Every later step traces back to one of the questions above.

---

## 3. Data

- **Source:** Online Retail II, UCI Machine Learning Repository (donated by Daqing Chen)
- **Coverage:** Dec 2009 – Dec 2011, UK-based non-store online retailer
- **Volume:** 1,067,371 raw rows across two sheets
- **License:** CC BY 4.0

I used the raw data rather than a pre-cleaned version, so the project includes real data-quality work:

| Issue | Handling |
|---|---|
| Mixed date formats (over half of rows) | Parsed with `TRY_CONVERT` under multiple style codes via `COALESCE` |
| Invalid type conversions | Safe casts to correct numeric and datetime types |
| 32,907 exact duplicate rows | Detected and removed with SQL window functions |
| Missing customer IDs | Excluded from customer-level analysis |
| Non-sale row types | Filtered out |

Row counts were reconciled against the published UCI total at each stage.

---

## 4. Approach

### Step 1: Data cleaning and validation (SQL Server)
Imported both sheets, standardized types and dates, deduplicated, and produced a single cleaned table. Validated counts after every transformation.

```sql
SELECT
    Invoice, StockCode, Description, Quantity,
    COALESCE(
        TRY_CONVERT(datetime2, InvoiceDate, 120),
        TRY_CONVERT(datetime2, InvoiceDate, 105)
    ) AS InvoiceDate,
    TRY_CONVERT(decimal(10,2), Price) AS Price,
    Customer_ID, Country
INTO dbo.RetailCombined
FROM dbo.[Year 2009-2010];
```

### Step 2: RFM scoring and segmentation (Python)
Computed Recency, Frequency, and Monetary value per customer, scored each on a 1–5 quintile scale, and mapped score combinations to six segments: **Champions, Loyal, At Risk, New, Needs Attention, Lost**. Added a churn flag for customers inactive 90+ days.

```python
rfm['R_Score'] = pd.qcut(rfm['Recency'], 5, labels=[5, 4, 3, 2, 1]).astype(int)
rfm['F_Score'] = pd.qcut(rfm['Frequency'].rank(method='first'), 5, labels=[1, 2, 3, 4, 5]).astype(int)
rfm['M_Score'] = pd.qcut(rfm['Monetary'], 5, labels=[1, 2, 3, 4, 5]).astype(int)
```

Design notes:
- Recency labels are reversed so a higher score always means a better customer.
- Frequency is ranked before binning because heavy ties would otherwise break `qcut`.

### Step 3: Modeling and dashboard (Power BI)
Merged segment output back onto transaction-level data to form one analysis-ready table. This lets every report page share a single source without extra relationships. Measures for customer count, revenue, churn %, and average customer value are written in DAX.

---

## 5. Dashboard

| Page | Content |
|---|---|
| **Home** | Navigation and a glossary of Recency, Frequency, Monetary, and Churn % |
| **Business Overview** | KPIs (customers, revenue, churn %, avg. customer value), segment breakdowns, month/quarter/year trends, revenue/customer toggle. Filters: date, country, segment, churn status |
| **RFM Analysis** | Segment summary (customers, revenue, avg. R/F/M), avg. value by segment, churn % over time |

---

## 6. Key Findings and Recommendations

| Segment | Observation | Suggested action |
|---|---|---|
| Champions | Small group, ~68% of revenue | Early access, loyalty rewards, referral programs |
| Loyal | Consistent repeat buyers | Upsell, cross-sell, bundles |
| New | Recent first purchase | Onboarding emails, second-purchase incentive |
| Needs Attention | Declining engagement | Reminder campaigns, personalized offers |
| At Risk | Previously valuable, now quiet | Win-back outreach before they lapse |
| Lost | Low value, long inactive | Low-cost reactivation only |

Revenue peaks in Q4, so retention campaigns should go out before the holiday season.

---

## 7. Limitations

- The dataset ends in 2011, so findings show method, not current market behavior.
- The 90-day churn threshold is a simplifying assumption. Purchase cycles in a gifting business are seasonal, so it likely overstates churn for customers who buy once a year.
- Quintile scoring is relative to this customer base, not an absolute benchmark.
- Customers without IDs are excluded, so results cover identified customers only.

**Possible extensions:** CLV modeling, a seasonality-adjusted churn definition, or a predictive churn model.

## 8. Acknowledgments

- Dataset: Daqing Chen, UCI Machine Learning Repository (CC BY 4.0)
- This project was built following [Mansi Goel's tutorial series](https://www.youtube.com/watch?v=LQa0SiuZfhA) on YouTube (replace with the actual link). Analysis, write-up, and any modifications are my own.

> **Note:** This project was built by following Mansi Goel's end-to-end SQL, Python, and Power BI tutorial series (link in Acknowledgments). I reproduced the full pipeline to practice the workflow and tools.

