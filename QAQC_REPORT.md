# MJZ Petroleum — Virtual Data Room QA/QC Audit Report

**Audit Date:** February 13, 2026  
**Auditor:** Automated QA/QC System  
**Files Compared:**
- VDR: `index.html` (Virtual Data Room web application)
- Source: `mjzp.xlsx` (TRC Engineering ownership assignments export)
- Source: `mjzp.docx` (Exhibit "A" — mineral interest legal descriptions)

---

## Executive Summary

The VDR is **substantially accurate** with interest data matching the source XLSX to high precision across all 78 wells. However, several discrepancies were identified — most notably **inconsistent well counts** across the VDR's own displays, **minor rounding differences** in rollup economics, and **naming inconsistencies** between source documents and the VDR.

**Critical Issues:** 1  
**Moderate Issues:** 5  
**Minor/Cosmetic Issues:** 8  

---

## 1. Well Count Discrepancies ⚠️ MODERATE

| Location in VDR | Count Shown | Actual (wells[] array) |
|---|---|---|
| Banner header | **76** | 78 |
| Field Summary table (sum) | **73** (20+5+4+30+1+12+1) | 78 |
| wells[] JavaScript array | **78** | 78 (authoritative) |
| XLSX source | **78** | 78 |

### Details:

- **Banner says "76 Wells"** but `wells[]` array contains **78 entries**. Discrepancy of 2.
- **Field Summary table** lists:
  - W. Cheyenne: **20** wells (actual: **21** — missing "Tracy Trust" in count)
  - Glenwood: **30** wells (actual: **34** — 4 wells undercounted)
  - Other fields are correct
  - Table total: 73 vs actual 78 (off by 5)

**Recommendation:** Update banner to 78 and correct the Field Summary table counts (Cheyenne→21, Glenwood→34). Or, if 76 is intentional (excluding wells with zero interest like Crawford 1-25, Grissom, Nowell, Taylor), document this clearly.

---

## 2. Interest Values — wells[] vs XLSX ✅ PASS

All 78 wells were compared for WI%, RI%, and ORRI%. **All values match within floating-point tolerance (±0.00005%).**

The XLSX stores interests as decimal fractions (e.g., `0.0052083` = 0.52083%), and the VDR correctly converts these to percentage display values.

### Notable Verified Values:
| Well | VDR WI% | XLSX WI% | Match |
|---|---|---|---|
| Braum 24-1 | 100.0 | 100.0 | ✅ |
| Skaggs 24-1 | 100.0 | 100.0 | ✅ |
| S. BRAWLEY GU 01 | 16.22000 | 16.22000 | ✅ |
| Charlie Winn 1-30 Atoka (ORRI) | 0.00013 | 0.00013 | ✅ |
| Love A-1 (ORRI) | 1.17027 | 1.17027 | ✅ |

---

## 3. Well Name Discrepancies ⚠️ MINOR

### 3a. XLSX Typo (Source Document Issue)
- **XLSX:** `Tracy - Croton 1-32 Chreokee` (typo: "Chreokee")
- **VDR:** `Tracy - Croton 1-32 Cherokee` (correct spelling)
- **Status:** VDR correctly fixed the typo. However, if XLSX is provided to buyers separately, this creates a mismatch.

### 3b. Naming Convention Differences
| XLSX Name | VDR Name | Impact |
|---|---|---|
| `Jeter A-1 - MJZP` | `Jeter A-1 (MJZP)` | Cosmetic (dash→parens) |
| `Love A-1 - MJZP` | `Love A-1 (MJZP)` | Cosmetic (dash→parens) |
| `NICHOLSON, JOHN W. GAS UNIT 1–9` | `NICHOLSON JW GU 1–9` | Abbreviated in VDR |
| `SOUTH BRAWLEY GAS UNIT 01–9` | `S. BRAWLEY GU 01–9` | Abbreviated in VDR |

**Note:** The abbreviations are reasonable for display but may cause confusion when cross-referencing with the XLSX. Consider adding a name-mapping note in the Documents tab.

---

## 4. Economics — Rollup Table vs econ[] Array ⚠️ MINOR

The Economics by State & Field table uses **pre-computed rollup values** from TRC's report, not dynamically summed from the `econ[]` array. This creates minor rounding discrepancies:

### Field-Level (Table vs Computed from econ[]):

| Field | Metric | Table Value | Computed | Diff |
|---|---|---|---|---|
| Dempsey-Mantooth | Misc Revenue | $3.31K | $3.30K | $0.01K |
| Cheyenne | Gas Reserves | 34.55 MMcf | 34.56 MMcf | 0.01 |
| Cheyenne | Expenses | $59.05K | $59.07K | $0.02K |
| Cheyenne | PV10 | $29.56K | $29.58K | $0.02K |

### State-Level Rollups:

| State | Metric | Table | Computed from Fields | Diff |
|---|---|---|---|---|
| Oklahoma | Gas | 53.87 | 53.88 | 0.01 |
| Oklahoma | Expenses | $83.07K | $83.09K | $0.02K |
| Oklahoma | PV10 | $76.44K | $76.47K | $0.03K |
| Texas | Gas | 7.64 | 7.65 | 0.01 |
| US Total | Oil | 5.93 | 5.94 | 0.01 |
| US Total | Gas | 61.51 | 61.53 | 0.02 |
| US Total | PV10 | $269.17K | $269.22K | $0.05K |

**Assessment:** These are normal rounding differences from TRC's report (which rounds at different levels of aggregation). The table values are from TRC's official report and should be considered authoritative. The `econ[]` array values sum correctly within rounding tolerance.

### Patricia Field — Missing from Rollup Sub-Table ⚠️ MODERATE

Patricia field (`Harris E W et al 13`: PV10=$2.31K, Oil Rev=$2.81K) is **included in the Texas total** but does **not appear as a named sub-field** in the rollup table. Only Deroan and Glenwood (Cotton Valley) are shown as Texas sub-fields.

- Texas table PV10 = $192.73K
- Deroan ($154.98K) + Glenwood ($35.44K) = $190.42K
- Difference = $2.31K = Patricia's PV10 ✓

**Recommendation:** Add Patricia as a sub-field row in the Economics table, or add a footnote.

---

## 5. Banner Summary Stats vs Computed ✅ MOSTLY PASS

| Stat | Banner | Computed | Status |
|---|---|---|---|
| Wells | 76 | 78 | ⚠️ See Issue #1 |
| Fields | 7 | 7 | ✅ |
| States | 3 | 3 (TX, OK, MO) | ✅ |
| Mineral Acres | ~6,246 | ~6,246 (4428+480+98+160+150+155+775*) | ✅ |
| Recent Completions | 9 | 9 (Upland Operating wells) | ✅ |
| PV10 | $269.2K | $269.17K (table) / $269.22K (sum) | ✅ (rounded) |
| Undiscounted CF | $591.8K | $591.79K (table) | ✅ |
| Net Oil | 5.93 Mbbl | 5.93 (table) / 5.94 (sum) | ✅ |
| Net Gas | 61.5 MMcf | 61.51 (table) / 61.53 (sum) | ✅ |
| Oil Revenue | $440.6K | $440.56K | ✅ |
| Gas Revenue | $229.7K | $229.71K | ✅ |

*Bailey County sections (Sec 18 N½+SE¼ = 480ac + Sec 19 SW¼ = 160ac + Sec 21 SE¼ = 160ac + Sec 37 W½ = 320ac + Sec 44 SW¼ = 160ac ≈ 1,280 acres for the two Bailey-only deeds, but exact acreage math is approximate as legal descriptions use section quarters)

---

## 6. Mineral Interests — VDR vs DOCX ✅ PASS

All 7 mineral interest descriptions from `mjzp.docx` (Exhibit "A") are present in the VDR Minerals tab. Verified:

| # | County/State | VDR | DOCX | Match |
|---|---|---|---|---|
| 1 | Bailey Co., TX (Sec 18, 19) | Deed 6/17/59, Vol. 78, pp. 387-388 | ✅ | ✅ |
| 2 | Bailey Co., TX (Sec 21, 37, 44) | Deed 6/17/59, Vol. 78, pp. 386-387 | ✅ | ✅ |
| 3 | Bailey & Lamb Co., TX (League 206) | 4,428 acres | 4428 acre | ✅ |
| 4 | Montgomery Co., TX | 98 acres, George Reynolds Survey | ✅ | ✅ |
| 5 | Walker Co., TX | 70.59 + 89.43 acres (~160 total) | ✅ | ✅ |
| 6 | Phelps Co., MO | 150 acres, Sec 19, T36, R6 | ✅ | ✅ |
| 7 | Jackson Co., OK | 155 acres, Sec 11, T2S, R20W | ✅ | ✅ |

**Note on ordering:** The DOCX lists items as #1-7 with Phelps County (#3 in docx) appearing before Montgomery County (#4 in docx). The VDR reorders them geographically (Bailey → Bailey & Lamb → Montgomery → Walker → Phelps → Jackson). This is acceptable but differs from source document order.

**Note:** The Jackson County deed in the DOCX says "Deed dated March 28, 1959" — the VDR shows "Deed dated 3/28/59" which is consistent.

---

## 7. Engineering Data (eng[] array) — Spot Check ✅ PASS

The `eng[]` array contains operator, reservoir, phase, and API data for all wells with economics. Spot-checked entries:

| Well | Operator | Reservoir | Phase | API |
|---|---|---|---|---|
| Crawford 2-25 | Mustang Fuel | Atoka | Gas | 35-129-22991 |
| Summer 1-31 | Mustang Fuel | Atoka | Gas | 35-129-23071 |
| Love A-1 (MJZP) | Cholla Petroleum | Mississippian | Oil | — |
| NICHOLSON JW GU 1 | Merit Energy | Cotton Valley | Gas | 42-459-30642 |

**Anomaly found:** `Tracy Trust` has API `42-459-30528` — this is a **Texas API prefix** (42=TX, 459=Upshur Co.) but Tracy Trust is listed as a **W. Cheyenne field well in Roger Mills Co., Oklahoma**. This API appears to be incorrectly assigned.

**⚠️ CRITICAL: Tracy Trust API number `42-459-30528` is a Texas Upshur County API, not an Oklahoma Roger Mills County API.** Oklahoma APIs start with `35-129-xxxxx` for Roger Mills County. This needs correction.

### NICHOLSON JW GU 3 API Anomaly ⚠️ MODERATE
- API shown: `42-183-31312` — prefix `42-183` is **Gregg County, TX**
- All other Glenwood/Nicholson wells use `42-459-xxxxx` (Upshur County, TX)
- This may be correct if this particular well straddles county lines, but should be verified.

---

## 8. Operator Activity Data — Spot Check ✅ PASS

The 9 Upland Operating wells in the Activity tab were cross-checked:

| Well | API | Completion | TD | Water | Status |
|---|---|---|---|---|---|
| TROUBADOUR 32-5 2H | 35-129-24144 | Mar 2024 | 11,306' | 35.4M | ✅ |
| Telli Camdyn 30-19 1H | 35-129-24160 | Jan 2025 | 11,904' | 26.3M | ✅ |
| Blue Clear Sky 19-30-31 1H | 35-129-24182 | Jul 2025 | 11,250' | 27.0M | ✅ |
| Big John 10-3 2H | 35-129-00033 | Nov 2025 | 11,300' | — | ✅ |
| Telli Camdyn 30-19 2H | 35-129-00034 | Nov 2025 | 11,300' | — | ✅ |

**Note:** The Nov 2025 wells (Big John 10-3 2H, Telli Camdyn 30-19 2H) have API numbers `35-129-00033` and `35-129-00034` — these are unusually low sequence numbers. These may be placeholder/temporary APIs pending final assignment from OCC.

---

## 9. Contact Information ✅ PASS

| Field | VDR Value |
|---|---|
| Name | Michael J. Zak |
| Phone | 713.724.0984 |
| Email | MJZPetroleum@yahoo.com |

---

## 10. Map Coordinates — Spot Check ✅ PASS

| Feature | VDR Lat/Lng | Expected Area | Status |
|---|---|---|---|
| W. Cheyenne Field | 35.65, -99.75 | Roger Mills Co., OK | ✅ |
| Tuttle Field | 35.28, -97.82 | Grady Co., OK | ✅ |
| Glenwood Field | 32.65, -94.95 | Upshur Co., TX | ✅ |
| Deroan Field | 32.75, -101.95 | Dawson Co., TX | ✅ |
| Phelps Co. Minerals | 37.90, -91.80 | Phelps Co., MO | ✅ |
| Bailey Co. Minerals | 34.05, -102.85 | Bailey Co., TX | ✅ |

All coordinates are within the correct counties.

---

## 11. Security ✅ NOTE

The VDR access code is hardcoded in JavaScript: `const ACCESS_CODE='7714'`. This provides minimal security — anyone viewing page source can bypass it. The `sessionStorage` approach also means the gate can be bypassed by setting `sessionStorage.setItem('mjzp-auth','1')` in the console.

**Recommendation:** This is acceptable for a data room shared via controlled distribution, but should not be relied upon as actual security. Consider server-side authentication if wider distribution is needed.

---

## Summary of Issues

### Critical (1)
1. **Tracy Trust API number** (`42-459-30528`) is a Texas/Upshur County API assigned to an Oklahoma/Roger Mills County well — likely incorrect.

### Moderate (5)
2. **Well count discrepancy**: Banner says 76, array has 78, field table sums to 73.
3. **Glenwood field count**: Table says 30 wells, array has 34.
4. **W. Cheyenne field count**: Table says 20 wells, array has 21.
5. **Patricia field missing** from Economics rollup sub-table (included in Texas total but not broken out).
6. **NICHOLSON JW GU 3 API** uses Gregg County prefix — verify if correct.

### Minor/Cosmetic (8)
7. XLSX source has typo "Chreokee" (VDR correctly fixed to "Cherokee").
8. Well naming conventions differ between XLSX and VDR (NICHOLSON abbreviated, SOUTH BRAWLEY abbreviated, dash vs parentheses for MJZP).
9. Minor rounding differences in economics rollups (max $0.05K on PV10 total).
10. Mineral interests displayed in different order than DOCX Exhibit A.
11. Nov 2025 operator activity API numbers (00033, 00034) appear to be placeholders.
12. Banner oil reserves shows 5.93 Mbbl (from TRC table) vs 5.94 computed from well data.
13. Banner gas reserves shows 61.5 MMcf (rounded from 61.51 table value).
14. Access code `7714` is visible in page source (inherent to client-side implementation).

---

*Report generated February 13, 2026*
