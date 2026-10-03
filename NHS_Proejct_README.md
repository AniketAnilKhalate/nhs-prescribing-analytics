'Full database (1.73GB): see Releases → v1.0.0'
# NHS Prescribing Analytics — ABC/XYZ Inventory Risk Analysis

A real-data analytics project identifying which NHS medicine categories combine
**high spend** with **unpredictable demand** — the segment where forecasting
errors are most financially costly — using genuine NHS open data, SQL, and
Power BI.

## Business Question

Across England's ~199 drug categories, where should NHS medicines planning
teams focus limited forecasting and supply-chain attention: on the categories
that cost the most, the ones that are hardest to predict, or the (rare)
overlap of both?

## Data Source

- **NHSBSA Prescription Cost Analysis (PCA) — Monthly Administrative Data**,
  opendata.nhsbsa.net (official NHS Business Services Authority open data)
- 7 months: **January–July 2026**
- 7 monthly CSV files, **3,836,305 raw rows**, combined into a single table
- Real, genuine NHS prescribing activity — not synthetic

## Data Engineering (SQL / SQLite)

- Combined 7 monthly files into one table via staged `INSERT`/`DROP` imports
- **Diagnosed and fixed a header-import bug**: the CSV import tool failed to
  recognise column headers, leaving 27 columns named `field1`–`field27` with
  header text embedded as literal data rows. Fixed with `ALTER TABLE ...
  RENAME COLUMN` (27 statements) and removed the resulting duplicate header
  rows (`DELETE FROM prescribing WHERE YEAR_MONTH = 'YEAR_MONTH'`)
- **Identified a live NHS organisational event affecting data integrity**:
  72 distinct `ICB_NAME` values were found instead of an expected ~42–48.
  Investigation confirmed this reflects a real, ongoing NHS England
  reorganisation — 12 Integrated Care Boards were merged into 6 new ones
  effective 1 April 2026, and the dataset spans both sides of that cutover.
  Built a cleaned `ICB_NAME_CLEAN` column stripping transitional suffixes
  (e.g. `(C 01-Apr-26)`) to prevent double-counting in regional analysis
- Final clean dataset: **3,836,298 rows**, 27 correctly-typed, correctly-named
  columns, plus one derived region-cleaning column

## Methodology

**ABC classification** (by total Net Ingredient Cost, `NIC`) — ranked all 199
`BNF_SECTION` drug categories into three equal-sized tiers using `NTILE(3)`.

**XYZ classification** (by demand predictability) — calculated the
coefficient of variation (standard deviation ÷ average) of each category's
monthly dispensed quantity across the 7 months, manually computing standard
deviation in SQL (`SQRT(AVG(x²) − AVG(x)²)`, as SQLite has no built-in
`STDEV()`). Categories averaging fewer than 1,000 units/month were excluded
from XYZ classification, since very low volumes produce statistically
unreliable variability readings.

**Combined matrix** — joined both classifications to find the genuine
overlap of high-spend *and* unpredictable categories.

## Key Finding

**Of 199 drug categories (£6.88bn total spend, Jan–Jul 2026), only ONE —
"Vaccines and antisera" (£29.8M) — is both high-spend (Class A) and
highly variable (Class Z, coefficient of variation 0.50).**

- 66 of 67 Class-A categories are demand-stable (Class X) — the vast
  majority of NHS high-cost medicine spend is actually easy to forecast
- Vaccine spend concentrates in no single outlier region — it scales
  roughly with population across all 48 ICBs, with the top 5 regions
  (including 4 London ICBs) accounting for ~25% of spend
- Monthly vaccine spend fell sharply from £8M (Jan) to ~£2M (Apr–Jul),
  consistent with the tail of a winter vaccination campaign

## Recommendation

Because the risk is not regionally concentrated, the recommendation is a
**national forecasting policy change** — campaign-calendar-aware demand
planning for vaccines specifically — rather than a targeted regional
intervention. The other 66 high-spend categories are well-served by
standard trailing-average forecasting.

## Limitations

- 7 months of data (not a full 12-month cycle) — seasonal patterns outside
  this window aren't captured
- Regional comparison uses absolute spend, not population-adjusted
  spend-per-capita, so apparent regional differences may partly reflect
  population size rather than genuine behavioural differences
- The 12 ICBs affected by the April 2026 merger have a shorter, non-continuous
  time series pre/post-merger and are harder to trend reliably

## Tools Used

- **SQL (SQLite / DB Browser)** — data combination, cleaning, ABC/XYZ
  classification, regional and trend analysis (CTEs, window functions
  `NTILE()`/`RANK()`, manual standard deviation, `HAVING` filters)
- **Power BI** — interactive dashboard: KPI card, conditional-formatted
  Matrix, trend line, regional bar chart

## Files

- `nhs_prescribing.db` — SQLite database (full cleaned dataset)
- `abc_xyz_summary.xlsx`, `vaccines_regional.xlsx`, `vaccines_monthly_trend.xlsx`
  — exported query results feeding the dashboard
- `NHS_Prescribing_Dashboard.pbix` — Power BI dashboard


<img width="1206" height="658" alt="image" src="https://github.com/user-attachments/assets/e30a8411-0859-470d-874d-f50703b7173b" />
