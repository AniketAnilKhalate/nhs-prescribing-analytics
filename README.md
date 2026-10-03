# NHS Prescribing Analytics — Inventory Risk Management & Supply Chain Optimization

A real-data analytics project identifying high-risk, volatile NHS medicine categories and engineering a decentralized vs. centralized inventory optimization model — **saving a theoretical 85.6% in buffer safety stock** — using genuine NHS open data, advanced SQL, and Power BI.

## Business Question & Core Objectives

Across England's ~199 drug categories, where should NHS medicines planning teams focus limited forecasting and supply-chain attention: on the categories that cost the most, the ones that are hardest to predict, or the (rare) overlap of both?
1. **Risk Identification:** Locate the rare drug categories that combine high financial spend (Class A) with unpredictable demand volatility (Class Z).
2. **Network Optimization:** Model a regional supply chain transformation to determine how much safety stock the NHS could save by pooling volatile inventory centrally instead of holding it independently across 48 local regions.

## Data Source

* **NHSBSA Prescription Cost Analysis (PCA) — Monthly Administrative Data**, [opendata.nhsbsa.net](https://opendata.nhsbsa.net) (Official NHS Business Services Authority open data)
* **Timeframe:** 7 months (January–July 2026)
* **Scale:** 7 monthly CSV files totaling **3,836,305 raw rows** of real, un-synthesized NHS prescribing activity.

## Data Engineering & Operational Integrity (SQL / SQLite)

* **Engineered Multi-Month Pipeline:** Unified 7 massive monthly files into a core analytical table via staged `INSERT`/`DROP` imports.
* **Resolved Structural Schema Bug:** The CSV import utility misread the table metadata, generating generic columns (`field1`–`field27`) and embedding headers as literal string rows. Overcame this by executing a batch of 27 structural `ALTER TABLE ... RENAME COLUMN` statements and purging the duplicate schema rows via targeted `DELETE FROM prescribing WHERE YEAR_MONTH = 'YEAR_MONTH'`.
* **Handled Live NHS Restructuring Event:** Discovered 72 distinct `ICB_NAME` records instead of the statutory ~42–48. Investigation confirmed a live NHS England reorganization: 12 Integrated Care Boards (ICBs) were consolidated into 6 new entities on 1 April 2026, and the dataset spans both sides of that cutover. Built a programmatic `ICB_NAME_CLEAN` derived column to strip transitional suffixes (e.g., `(C 01-Apr-26)`), mapping the timeline cleanly to 48 standard regions and preventing critical double-counting.
* **Final Clean Dataset:** **3,836,298 rows**, 27 correctly typed/named columns, and 1 derived regional consolidation feature.

---

## Methodology

### Stage 1: ABC/XYZ Portfolio Risk Matrix
* **ABC Segmentation (Spend):** Ranked all 199 `BNF_SECTION` categories by total Net Ingredient Cost (`NIC`) and partitioned them into three equal-sized financial tiers using `NTILE(3)`.
* **XYZ Segmentation (Predictability):** Calculated the Coefficient of Variation (CV = standard deviation ÷ average) of monthly dispensed quantities. Because SQLite lacks a native `STDEV()` function, standard deviation was manually engineered using variance mathematical properties: `SQRT(AVG(x²) - (AVG(x)*AVG(x)))`. Micro-volume categories averaging fewer than 1,000 units/month were filtered out to avoid statistical distortion from unreliable variability readings.
* **Combined Matrix:** Joined both classifications to find the genuine overlap of high-spend *and* unpredictable categories.

### Stage 2: Inventory Safety Stock Optimization (Maister’s Square Root Law)
To solve the risk identified in Stage 1, an inventory centralization model was built. It simulates independent local storage across all 48 clean ICB regions versus a single, pooled national center under a conservative **10-day lead time scenario** (0.333 months) and a **95% service level (z = 1.65)**:

$$	ext{Total Independent Safety Stock} = \sum \left( 1.65 	imes \sigma_{	ext{local}} 	imes \sqrt{LT} 
ight)$$

$$	ext{Centralized Pooled Safety Stock} = 1.65 	imes \sigma_{	ext{pooled}} 	imes \sqrt{LT} = rac{	ext{Total Independent Safety Stock}}{\sqrt{N}}$$

---

## Key Findings

```
[ All 199 Drug Categories: £6.88bn Total Spend ]
       │
       ├── 66 Class-A Categories ───► [ Class X / High Spend, Low Volatility ]
       │                               (98.5% of high-cost spend is highly stable)
       │
       └── 1 Outlier Category ──────► [ Class Z / High Spend, High Volatility ]
                                       "Vaccines and antisera" (£29.8M Spend, CV: 0.50)
```

### 1. Portfolio Analysis: The "Vaccines" Outlier
* **The Matrix:** Of 199 categories, **only ONE** fell into the high-spend, high-volatility danger zone (Class AZ): **"Vaccines and antisera" (£29.8M)**. 
* **Systemic Stability:** 66 of 67 Class-A categories are highly predictable (Class X), proving that standard trailing-average models suffice for 98.5% of major NHS drug spend.
* **Socio-Demographic Distribution:** Vaccine demand scales directly with population rather than regional anomalies. The top 5 regions (including 4 London ICBs) accounted for ~25% of spend. 
* **Seasonal Drops:** Monthly vaccine spend fell sharply from £8M (Jan) to ~£2M (Apr–Jul), consistent with the tail of a winter vaccination campaign.

### 2. Supply Chain Optimization: The Network Pooling Dividend
Applying Maister's Square Root Law to the volatile "Vaccines and antisera" category under an updated 10-day lead time revealed a massive structural optimization runway:

* **Absolute Dose Savings:** Transitioning from 48 independent regional stockpiles to a centralized national distribution model scales absolute dose safety requirements downward by **roughly 1.9×** compared to shorter lead-time baselines.
* **The 85.6% Structural Constant:** Strikingly, while shifting the lead time radically shifts the total volume of absolute doses required, **the percentage reduction remains locked at exactly 85.6%**. 
* **The Mathematical Proof:** Because lead time ($\sqrt{LT}$) acts as a linear multiplier across both independent and pooled frameworks, it entirely cancels out of the efficiency ratio:

$$\% 	ext{ Reduction} = 1 - rac{	ext{Pooled}}{	ext{Independent}} = 1 - rac{1}{\sqrt{48}} pprox 85.6\%$$

This confirms that the 85.6% inventory savings dividend is a fixed, unchanging property of the **network architecture** itself, completely isolated from local operational metrics.

---

## Strategic Recommendations

1. **National Centralization Policy:** Rather than attempting localized regional interventions or complex algorithmic forecasting fixes at the ICB level, the NHS should centrally pool "Vaccines and antisera" safety stocks. This structurally guarantees an **85.6% reduction in buffer inventory holding costs** while maintaining a 95% service level.
2. **Campaign-Calendar Calibration:** Because the baseline portfolio variance was driven by predictable seasonal drop-offs—collapsing from £8M in January to a steady ~£2M per month from April onwards—the centralized model should utilize calendar-aware campaign parameters rather than standard moving averages.

## Project Limitations

* **Truncated Horizon:** The 7-month data window cannot capture a full 12-month cyclical view, meaning late-autumn or early-winter surges are omitted.
* **Demographic Granularity:** Regional spending analysis relies on absolute financial totals rather than population-adjusted per-capita spending, which masks whether variations are driven by regional healthcare behavior or simple scale.
* **Reorganization Friction:** The 12 ICBs caught in the April 2026 merger feature broken historical time series, making long-term baseline trending inherently noisier.

## Enterprise Tech Stack Used

* **SQL (SQLite / DB Browser):** High-volume data parsing, cross-table stitching, algorithmic sub-queries (`CTEs`), window functions (`NTILE`, `RANK`), custom standard deviation modeling, and mathematical network simulations (`HAVING` filters).
* **Power BI:** Executive-facing interactive dashboard including high-level KPI blocks, a conditionally formatted portfolio matrix, timeline trend models, and regional Pareto-style breakdowns.

## Inventory Registry & Artifacts

* `nhs_prescribing.db` — Cleaned, schema-corrected SQLite warehouse (1.73 GB).
* `abc_xyz_summary.xlsx` — Classified product tier outputs feeding the core matrix.
* `network_optimization_summary.xlsx` — Core mathematical simulation model evaluating localized safety stocks against centralized infrastructure.
* `vaccines_regional.xlsx` & `vaccines_monthly_trend.xlsx` — Downstream granular datasets powering regional geographic trends.
* `NHS_Prescribing_Dashboard.pbix` — Interactive dashboard file ready for deployment.
