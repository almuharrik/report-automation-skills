# Verification checks

Each check returns PASS, WARN, MISSING or OPEN. Every result goes into the QA Log of the Data Pack. Tolerances are relative unless stated.

| # | Check | Compare | Tolerance | On fail |
|---|---|---|---|---|
| 1 | Total ads cost | Shopee (Product + Shop) + Meta Reguler + Meta CPAS = Total Ads Cost | exact | WARN |
| 2 | Revenue, three sources | Shopee overview monthly sum = `CATEGORY_MONTHLY` month total = `SKU_DETAIL` parent-row sum (Penjualan Siap Dikirim) | 0.1% | WARN, list gap |
| 3 | Customer value | New Cust value + Existing Cust value = Penjualan Siap Dikirim | 0.1% | WARN |
| 4 | Shopee Ads Product | `ADS_CAMPAIGN` Biaya and Omzet = workbook `RAW_SHOPEE_ADS` Ads_Type Product | 0.5% | WARN |
| 5 | Shopee Ads Shop | Workbook `RAW_SHOPEE_ADS` Ads_Type Shop vs a Shop export in the folder | 0.5% | MISSING if no export |
| 6 | Meta Reguler cost | `META_REG` Spend = workbook `Table_Meta_Reg` | exact | WARN |
| 7 | Meta Reguler revenue | `META_REG` Penjualan_Shopee = UTM channel `Facebook Ads` | exact | WARN |
| 8 | Meta CPAS cost | `RAW_META_CPAS` Spend = `Table_Meta_CPAS` | exact | WARN |
| 9 | Meta CPAS revenue vs Shopee | Meta Purchase Value vs UTM `Facebook Collaborative Ads - Sales` | none. Report the gap | WARN with gap value (attribution, see SKILL Rule 7) |
| 10 | Conversion and AOV | Recompute from monthly query, compare with Seller Centre | 0.01 pt / 1% | WARN |
| 11 | ACR, ROAS, revenue share | Recompute from the Data Pack inputs | 0.01 pt / 1% | WARN |
| 12 | Buyers | New + Existing count = total buyers (Pembeli) | exact or 1% | WARN |
| 13 | Product coverage | Every Kode Produk in `CATEGORY_MONTHLY` maps in `CATEGORY_MASTER` (excluding Biaya Packaging) | 100% | OPEN, ask before assigning |
| 14 | Campaign vs group files | Use one set only; check the group file covers the campaign total | coverage stated | OPEN if coverage < 100% |
| 15 | Daily completeness | Every calendar day present in each daily source; Shop zero-days confirmed | no gaps | MISSING / OPEN |
| 16 | Provisional data | Last 3 UTM days and current day excluded or labelled | n/a | WARN if used unlabelled |
| 17 | Prior-month restatement | Previous month values = last published report | rounding | WARN, explain |
| 18 | Unit check | Shopee Ads money not 100,000x off | n/a | WARN |
| 19 | Numbers stored as text | Count converted cells | n/a | log count |
| 20 | Revenue labels | Each revenue figure states its source: Seller Centre, Akar API, Business Insights or wallet | n/a | WARN if mixed |
| 21 | Meta spend vs billing | Meta spend = invoice billing, tax stated | exact | WARN, state tax treatment |

## Not a check (do not flag)

- Days with Spend = 0 and Purchase > 0 in `RAW_META_CPAS`. These are the attribution window: a click several days earlier converts on a later purchase date. Compare monthly totals only.
