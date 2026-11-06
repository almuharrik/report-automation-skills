---
name: haydecor-monthly-data-collector
description: Collects, cleans, reconciles and calculates the monthly Haydecor Ads Performance Report data (Shopee Ads, Meta Reguler, Meta CPAS, Shopee funnel, customers, product and category). Produces a Data Pack for the report writer. Use when a monthly report folder raw_data/YYYY-MM_ReportData is ready or the user asks for report data for a month.
version: 1.0.0
---

# Haydecor Monthly Data Collector

## Purpose

Turn one month of raw exports into a verified **Data Pack**. The report writer (human or agent) uses the Data Pack and nothing else for numbers. You do not write headlines, insights or recommendations.

## Scope

- Client: Haydecor (Shopee store `haydecor`).
- Channels: Shopee Ads (Iklan Produk = GMV Max; Iklan Toko+ = Shop), Meta Reguler, Meta CPAS, Shopee organic / non-ads.
- Currency IDR, timezone Asia/Jakarta, calendar month.
- Other clients (RHA Gallery) are out of scope for this skill.

## Inputs

Each month lives in `raw_data/YYYY-MM_ReportData/`. Expected files:

| Key | File (as named in the repo) | Delimiter / encoding | Grain |
|---|---|---|---|
| `SHOPEE_OVERVIEW` | `Daily Tracking Haydecor - Shopee Overview.csv` | `,` UTF-8 | daily |
| `META_REG` | `Daily Tracking Haydecor - RAW_META_REG.csv` | `,` UTF-8 | daily |
| `ADS_CAMPAIGN` | `Data+Keseluruhan+Iklan+Shopee-<period>.csv` | `,` UTF-8 with BOM, 7 metadata lines before the header (`skiprows=7`) | per campaign (Iklan Produk) |
| `ADS_GROUP` | `Semua-Data-Grup-Iklan-Shopee-<period>.csv` | `,` UTF-8 with BOM, 6 metadata lines (`skiprows=6`) | per group and per product |
| `CUSTOMER` | `Haydecor_Sales_by_Customer Type.csv` | `,` UTF-8; money as `Rp310,270,000` text | month x customer type |
| `SKU_DETAIL` | `parentskudetail.<YYYYMM01>_<YYYYMM31>.csv` | `;` cp1252; numbers use `.` as thousands separator, `-` = 0 | variation rows; parent row where Kode Variasi = `-` |
| `CATEGORY_MASTER` | `Product_Category_List.csv` | `;` UTF-8 | product (Kode Produk) |
| `CATEGORY_MONTHLY` | `Sales_Product_Category_<Mon>.csv` | `,` | product, GMV for 3 months |

Read from the workbook `raw_data/Daily Tracking Haydecor.xlsx` when a file is not in the folder:

| Need | Workbook tab |
|---|---|
| Shopee Ads Iklan Toko+ (Shop) daily | `RAW_SHOPEE_ADS` where Ads_Type = Shop |
| Shopee Ads Iklan Produk daily (cross-check) | `RAW_SHOPEE_ADS` where Ads_Type = Product |
| Meta CPAS daily spend and purchase value | `RAW_META_CPAS` |
| Meta Reguler revenue by channel (UTM) | `RAW_SHOPEE_UTM` where Platform = Semua |
| Monthly cost and revenue by channel | `Table_Consolidated`, `Table_Meta_Reg`, `Table_Meta_CPAS`, `Table_ShopAds` |
| Monthly targets | `MONTHLY_TARGET` |
| Prior reports (for restatement checks) | `Report <Mon>` and archive tabs, `Dasboard` |

If a required input is missing and no workbook tab covers it, mark it MISSING and stop for that metric (see Rule 9).

## Step 1: Inventory

1. List the folder. Map every file to a key above. Record unknown files and do not use them.
2. Confirm the report month and the period in the file names. Files with a different period (for example Nov 2026 rows in an Oct file) are fine, but **drop rows outside the report month** before any sum.
3. Write the inventory to the QA Log: key, file name, row count, date range, delimiter used.

## Step 2: Parse

- Strip the BOM (`utf-8-sig`) and any metadata lines before the header.
- Convert text numbers before any math:
  - `Rp633,759` or `Rp310,270,000`: remove `Rp`, remove commas. Log the row count converted.
  - `1.234.567` (SKU file): remove dots. `37.932.868` = 37,932,868.
  - Percent strings (`4.35%`, `1,43%`): divide by 100 only when you need a ratio.
  - `-` or empty = 0 for amounts, MISSING for rates.
- Shopee Ads money in the API is Rupiah × 100,000. The sync already converts this. A value 100,000 times too large is a WARN.
- Duplicate column names (SKU file has `Kode Variasi` twice): use position, not name, for the second one.

## Step 3: Collect metrics

Collect exactly these values per metric. Keep full IDR; do not round.

**A. Shopee funnel (monthly)**
- Pengunjung (unique visitors), Pesanan Dibuat, Pesanan Siap Dikirim, Penjualan Dibuat, Penjualan Siap Dikirim, Pembeli, Tingkat Konversi (both versions).
- Source of truth: one Seller Centre monthly date-range query (Data Bisnis → Tinjauan). Daily overview is only for trend charts.
- Revenue for the report = **Penjualan Siap Dikirim** (called Confirmed in the daily tabs).

**B. Shopee Ads**
- Iklan Produk (GMV Max): Biaya, Omzet Penjualan, Penjualan Langsung (GMV Langsung), Dilihat, Jumlah Klik, Konversi, Produk Terjual, ACOS. Source: `ADS_CAMPAIGN`, sum of the 24 or more campaign rows.
- Iklan Toko+ (Shop): Spend, Revenue, Order, Impr, Click. Source: workbook `RAW_SHOPEE_ADS` Ads_Type = Shop.
- Shopee Ads total cost = Product + Shop. Shopee Ads revenue = Product + Shop.
- Use `ADS_GROUP` only for the group-level breakdown. Never add it to `ADS_CAMPAIGN` (see Rule 5).

**C. Meta Reguler**
- Cost: `META_REG` Spend, summed for the month.
- Revenue: Shopee UTM, channel `Facebook Ads` only (`RAW_SHOPEE_UTM` or the `Penjualan_Shopee` column in `META_REG`). Do not include `Facebook Collaborative Ads - Sales` here; that is CPAS.
- Check that the `Penjualan_Shopee` column and the UTM channel total agree (Rule 6).

**D. Meta CPAS**
- Cost: `RAW_META_CPAS` Spend, or `Table_Meta_CPAS`.
- Revenue: `RAW_META_CPAS` Purchase Value (Meta-attributed). The Sep 2026 report used this value.
- Reference only: UTM `Facebook Collaborative Ads - Sales` (Shopee-side). It is not the same measurement; see Rule 7.

**E. Customers**
- New Cust and Existing Cust: count, value, value per customer for the month, from `CUSTOMER`.
- Retention = Existing Cust count ÷ (New + Existing count).

**F. Product and category**
- GMV per product from `CATEGORY_MONTHLY` (Gmv of the report month).
- Revenue per product and parent SKU from `SKU_DETAIL` (parent rows only).
- Category per product from `CATEGORY_MASTER`. Map on Kode Produk, never on name.
- Exclude `Biaya Packaging` line items from product and category totals. They stay in Shopee revenue.

**G. Targets and context (manual, always ask)**
- Revenue target and spend target from `MONTHLY_TARGET`. Confirm each month; never reuse an old value silently.
- Hari efektif (effective days), store closures, promos, launches. Ask the human if not provided.

## Step 4: Calculate

| Metric | Formula |
|---|---|
| Total Ads Cost | Shopee Ads (Product + Shop) + Meta Reguler + Meta CPAS cost |
| Total Ads Revenue | Shopee Ads revenue + Meta Reguler revenue (UTM Facebook Ads) + Meta CPAS purchase value |
| ACR | Total Ads Cost ÷ Penjualan Siap Dikirim. Ceiling 10% |
| Ads revenue share | Total Ads Revenue ÷ Penjualan Siap Dikirim |
| Non-ads revenue | Penjualan Siap Dikirim − Total Ads Revenue |
| ROAS per channel | Channel ads revenue ÷ channel cost |
| ROAS Meta (Mix) | (Reguler + CPAS revenue) ÷ (Reguler + CPAS cost) |
| ROAS All | Total Ads Revenue ÷ Total Ads Cost |
| Conversion rate | Pembeli ÷ Pengunjung (monthly query) |
| AOV | Penjualan ÷ Pesanan (state which: dibuat or siap dikirim) |
| Daily revenue | Revenue ÷ hari efektif. Also output revenue ÷ calendar days, labelled |
| MoM, QoQ, YoY | (Current − Previous) ÷ Previous. Also the absolute delta |
| Retention | Existing Cust ÷ total buyers (customer file) |
| CPAS share of spend | CPAS cost ÷ Total Ads Cost |
| Channel waterfall | Channel ads revenue delta vs last month. Subtotal must equal total MoM delta |
| Category share | Category GMV ÷ total product GMV (excluding Biaya Packaging) |
| Category increment | Category GMV this month − last month, from `CATEGORY_MONTHLY` |
| Keyword burning share | Keyword cost ÷ total Shopee Ads cost. Flag 100% burning with zero revenue |
| BCG quadrant | CR = Pembeli ÷ Pengunjung per product. Traffic and CR thresholds from the BCG sheet (`RUMUS`). Quadrant is set in that sheet, never in code |

## Step 5: Verify (run all checks, then release)

Each check gets PASS, WARN or MISSING in the QA Log. Full list in `docs/CHECKS.md`. Required checks:

1. Sum of 3 channel costs = Total Ads Cost.
2. Revenue cross-source: Shopee overview monthly sum = `CATEGORY_MONTHLY` month total = `SKU_DETAIL` parent-row sum (tolerance 0.1%).
3. Customer value (new + existing) = Penjualan Siap Dikirim (tolerance 0.1%).
4. Shopee Ads Product cost and revenue: campaign file = workbook `RAW_SHOPEE_ADS` Product (tolerance 0.5%).
5. Meta Reguler monthly cost: `META_REG` = workbook `Table_Meta_Reg` (exact).
6. Meta Reguler revenue: `Penjualan_Shopee` = UTM Facebook Ads channel (exact).
7. Every Kode Produk in the sales file maps to a category (100%).
8. Every day of the month present in each daily source (see Rule 8).
9. Previous month values match the last published report (within rounding). Explain any restatement.

## Step 6: Flags and questions

Flag but do not interpret. Use `WARN` for a mismatch, `OPEN` for a question for the human.

| Signal | Rule |
|---|---|
| ACR above 10% | ACR > 0.10 |
| CPAS ROAS outside 6–7x benchmark | Report as *deviation from benchmark*. Do not say "breakout" or "deterioration" |
| Possible attribution overlap | CPAS ROAS and total ACR both up in the same month |
| Traffic vs conversion divergence | Traffic and CR move in opposite directions by more than 5 points |
| Keyword waste | 100% burning share and zero revenue |
| Calendar effect | Total revenue down while daily revenue up (effective days) |
| Hypothesis | Any seasonal explanation without prior-year data = UNVALIDATED |

## Step 7: Output: the Data Pack

Deliver one workbook `Data_Pack_YYYY-MM.xlsx` with these tabs, plus `Data_Pack_YYYY-MM.json` (same numbers):

1. **Summary**: headline metrics with MoM, QoQ, YoY, target vs actual.
2. **Channel**: cost, revenue, ROAS per channel; last 6 months trend.
3. **Shopee Funnel**: visitors, buyers, orders, CR, AOV (monthly query).
4. **Buyers**: new, existing, retention, value per customer.
5. **Category & Product**: category share and increment, top 5 products, BCG table.
6. **Ads Detail**: campaign table, group breakdown (labelled as group-level), keyword table, CPAS by creative if provided.
7. **QA Log**: every check from Step 5 with PASS / WARN / MISSING / OPEN, the inventory from Step 1, and the question list.

Every metric record:

```json
{
  "metric": "ACR",
  "period": "2026-10",
  "value": 0.0,
  "unit": "ratio",
  "formula": "total_ads_cost / revenue_siap_dikirim",
  "inputs": {"total_ads_cost": 0, "revenue_siap_dikirim": 0},
  "sources": ["Shopee Ads campaign export", "META_REG", "RAW_META_CPAS"],
  "status": "OK | WARN | MISSING | OPEN",
  "note": ""
}
```

## Step 8: Stop and ask the human

Stop before release if any of these hold:

- A required file is missing, or a file has a new column layout.
- A product code is unmapped.
- A WARN touches a headline number (revenue, ACR, ROAS, total spend).
- Targets, effective days or client notes are not provided.
- A definition check in Step 5 fails.

## Rules

1. **Period.** Calendar month, Jakarta time. Previous month always pulled for MoM. Same month last year for YoY when available. Full quarter at quarter-end.
2. **Revenue definition.** Report revenue = Penjualan Siap Dikirim. Never mix dibuat and siap dikirim in one calculation.
3. **Source priority.** Raw export or daily feed first, dashboard screenshot second, manual input last. When two sources disagree, report both and flag WARN. Never pick silently.
4. **Visitors and buyers are not additive.** Monthly Pengunjung and Pembeli come from one monthly query. Daily visitors summed over a month give visitor-days, not people. The Oct 2026 daily overview sums to 71,947; that number is not the month's visitors.
5. **Ad groups and campaigns overlap.** `ADS_GROUP` contains campaign-total rows and per-product rows that describe the same spend. Use one set only. `ADS_CAMPAIGN` is the total.
6. **Meta Reguler revenue = UTM Facebook Ads channel.** Not Collaborative. Not Meta's reported value.
7. **Meta CPAS revenue = Meta purchase value.** Meta and Shopee measure CPAS differently (attribution window, view-through, measurement date). The gap is expected; flag it, do not average it away. Report the gap as a number.
8. **Attribution window and date basis.** Meta reports conversions on the purchase date, and a purchase can come from a click several days earlier. So a day with Spend = 0 and Purchase > 0 is expected. Do **not** flag these days as WARN. Compare **monthly totals only**; daily CPAS ROAS is not valid. Daily ROAS charts must say "purchase-date basis".
9. **Missing data.** Write `[DATA NEEDED: what, where to get it]` and mark MISSING. Never carry forward last month's value. Never estimate.
10. **Raw numbers stay raw.** Full IDR. Rounding and decimal style are the writer's job.
11. **Re-pull.** Shopee order statuses and late attribution change after month-end. Pull at least 3–5 days after month-end and log the timestamp. A re-pull that moves total revenue by more than 0.5% = WARN. UTM data lags at least one day and its last 3 days are provisional.
12. **Same definition every month.** Seller Centre labels the Penjualan card "Perubahan Definisi 2026". If the definition in the data differs from this file, stop and flag.
13. **Cleaning.** Exclude full returns (returned qty ≥ ordered qty with approved cancellation), zero-quantity rows and `Biaya Packaging` from product analysis. Deduplicate products per basket. Normalize product names by stripping brand prefix and filler words, with a manual merge list for known variants. Unmapped codes are asked, not guessed.
14. **Rows outside the month are dropped** before any total. Oct 2026 files include 1–3 Nov rows.
15. **Security.** Never copy the Apps Script `/exec` URL, `SPREADSHEET_ID`, Meta access token, HAR files, or client exports into outputs, logs, or commits. Repo must stay private.

## Known source issues (carry into every run until fixed)

- `ADS_GROUP` covers 29.19M of the 40.82M Iklan Produk spend in Oct 2026. Cause not established; use `ADS_CAMPAIGN` for totals.
- Iklan Toko+ (Shop) has no monthly export in the folder. Use the workbook and flag it.
- Shop days with no activity are logged as 0 sometimes and left out other times. Confirm before treating a missing day as 0.
- `RAW_META_REG` has no rows for some days (Oct 2026: 25 and 26). Verify in Meta Ads Manager whether spend was 0 or the importer missed them.
- Meta Reguler cost differs by Rp184,936 between the Mar 2026 report and the workbook (report 2,181,916; workbook 2,366,852). Feb, Jan, Dec 2025 and Jul–Sep 2026 match. Confirm which source is right before the next report.
- Shopee Ads ROAS in the Mar 2026 report appears as 8.20x, 8.42x and 8.47x. Use one source per report and label it.
- The sync extension's `allowOverwrite` setting may revise recent Shopee Ads figures. Compare with the last pull before release.
