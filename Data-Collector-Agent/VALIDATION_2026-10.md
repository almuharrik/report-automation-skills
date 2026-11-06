# Validation log: October 2026 (`raw_data/2026-10_ReportData`)

Run against the 8 files in the folder plus `Daily Tracking Haydecor.xlsx`. Report month: October 2026 (rows dated 1–3 Nov 2026 excluded).

## Results

| # | Check | Result | Status |
|---|---|---|---|
| 2 | Revenue, three sources | Shopee overview Oct: Rp422,988,820. `Sales_Product_Category_Oct` GMV: Rp422,988,820 (includes Biaya Packaging Rp2,411,754). SKU parent rows: Rp422,988,820 | PASS |
| 3 | Customer value | New Rp325,840,000 (1,960 buyers) + Existing Rp97,152,930 (284) = Rp422,992,930. Diff Rp4,110 (0.001%) | PASS |
| 4 | Shopee Ads Product | Campaign file Biaya Rp40,815,607 vs workbook Rp40,815,605 (diff Rp2). Omzet Rp359,110,784 = workbook | PASS |
| 5 | Shopee Ads Shop | Workbook Oct Shop: spend Rp472,656, revenue Rp316,600. No Shop export in the folder | MISSING (export) |
| 6 | Meta Reguler cost | `META_REG` Oct Rp3,137,548 = `Table_Meta_Reg` Rp3,137,548 | PASS |
| 6b | Meta Reguler days | Rows for 25 and 26 Oct are absent from the export | OPEN: check Meta Ads Manager |
| 7 | Meta Reguler revenue | `Penjualan_Shopee` Rp4,585,856 = UTM `Facebook Ads` Rp4,585,856 | PASS |
| 8 | Meta CPAS cost | Workbook Oct Rp2,972,355 | PASS (single source, no folder export) |
| 9 | Meta CPAS revenue vs Shopee | Meta purchase value Rp37,304,122 vs UTM `Facebook Collaborative Ads - Sales` Rp7,462,852. Gap Rp29,841,270 | WARN (attribution, expected per Rule 7) |
| 12 | Buyers | 1,960 + 284 = 2,244 buyers | PASS |
| 13 | Product coverage | 308 SKUs in sales file, all mapped in `Product_Category_List`. The only unmapped code is Biaya Packaging (excluded by rule) | PASS |
| 14 | Campaign vs group file | Group file: campaign-total rows Rp29,192,728 = product rows Rp29,192,729 (internally consistent). Covers 71.5% of campaign spend (Rp40,815,607) | OPEN: cause not established. Use campaign file for totals |
| 15 | Daily completeness | Shopee overview: 31 of 31 days. Meta Reguler: 29 of 31 days | see 6b |
| 17 | Prior month | Not run for Oct (no Sep daily file in folder; Sep report figures agree with workbook) | n/a |

## Also recorded

- **Orders:** overview Pesanan Siap Dikirim = 2,868 for the month. SKU-level Pesanan sums to 4,354 because one order can contain several products. Use 2,868 for order count.
- **Visitors:** daily overview visitors sum to 71,947. SKU-level product visitors sum to 149,117. Neither is the monthly unique count. Use the monthly Seller Centre query.
- **Retention:** 284 ÷ 2,244 = 12.66%.
- **Value per customer:** checked. Rp325,840,000 ÷ 1,960 = Rp166,245, matches the file.
- **Category totals (excl. Biaya Packaging):** Custom Poster Rp109,615,792; Nature & Visual Art Rp104,288,924; Retro & Mirror Style Rp85,612,532; Dekorasi Kamar GenZ Rp33,092,175; Sports & Motorsport Rp32,420,737; Film, Music & Pop Culture Rp22,218,446; Bundling & Set Dekorasi Rp19,039,603; Quotes & Typography Rp14,127,192; Nature & Visual Art - Kanvas Series Rp161,665. Total Rp420,577,066.
- **Category naming:** `Nature & Visual Art - Kanvas Series` and `Additional / Accesories` are single-SKU categories in the master. Confirm whether they should merge (OPEN).
- **Master file:** 7 rows have a blank New Category. None are in the October sales file except Biaya Packaging.
- **Meta CPAS zero-spend days:** 4 days in October with Spend = 0 and Purchase > 0. Attribution window, not an error (Rule 8).
- **CPAS ROAS (Meta, monthly):** 12.55x in Oct vs 12.18x in Sep. Compare on monthly totals only.

## Meta Reguler cost: which months differ from the reports

| Month | Report | Workbook | Result |
|---|---|---|---|
| Dec 2025 | 2,540,559 | 2,540,559 | match |
| Jan 2026 | 3,860,458 | 3,860,458 | match |
| Feb 2026 | 1,824,001 | 1,824,001 | match |
| Mar 2026 | 2,181,916 | 2,366,852 | **differs: +184,936 (+8.5%) in workbook** |
| Jul 2026 | 2,894,412 | 2,894,412 | match |
| Aug 2026 | 2,499,555 | 2,499,555 | match |
| Sep 2026 | 3,917,489 | 3,917,489 | match |
| Apr, May, Jun 2026 | no report in project folder | 3,945,778 / 3,702,563 / 4,261,131 | not checkable |

Only March 2026 differs. Which source is correct is still open.
