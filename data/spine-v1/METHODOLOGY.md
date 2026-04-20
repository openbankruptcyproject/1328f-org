# Methodology Note: §1328(f) Estimated Violations National Spine

**Artifact:** `district_violations_fy2008_2024.csv`
**Version:** v1
**Retrieved:** 2026-04-19
**Source:** FJC Integrated Database (C13 pending file), FY2008-2024, via local `fjc` MCP server

## 0. Purpose

This note is the citation anchor for every downstream vignette, news write-up, and academic analysis that draws on the spine. It documents what the numbers measure, what they assume, and how to reproduce them from primary sources.

## 1. The statute

11 U.S.C. § 1328(f) bars discharge in a Chapter 13 case where the debtor:

1. received a Chapter 7 discharge within four years before filing the Chapter 13, or
2. received a Chapter 13 discharge within two years before filing the Chapter 13.

The bar is not discretionary. It is not an affirmative defense a creditor must raise. If the statutory condition is met, the court is prohibited from entering a discharge.

## 2. Data source

The underlying data is the **Federal Judicial Center's Integrated Database (IDB), Bankruptcy -- Chapter 13 Pending File**, covering federal fiscal years 2008 through 2024.

- Local store: `pacer/data/fjc_national.db` (SQLite, ~37.9M rows, table `fjc_cases`)
- Query server: `docker/fjc-mcp/server.py` (Python FastMCP, stdio transport)
- Covers 92 reporting districts (territorial districts D.V.I. and D.N. Mar. I. have zero-row returns and are excluded from this spine)

The IDB `fjc_cases` table stores **one row per case per fiscal-year snapshot**. Each case therefore appears multiple times (typically 2-8) as it ages through pending-case reports. The MCP server deduplicates to the latest snapshot via:

```sql
WITH latest AS (
  SELECT casekey, MAX(id) AS max_id
  FROM fjc_cases
  WHERE crntchp = '13'
    AND CAST(filefy AS INTEGER) BETWEEN ? AND ?
  GROUP BY casekey
)
SELECT f.* FROM fjc_cases f JOIN latest l ON f.id = l.max_id
```

Naive aggregation without this deduplication overcounts by a factor of ~7-8. Any external reproduction must dedupe the same way.

## 3. Fields used

- `crntchp` -- current chapter. Filter: `= '13'`.
- `prfile` -- prior-filing indicator. Values `'Y'`, `'N'`, blank. `'Y'` means the debtor self-reported or the clerk recorded a prior bankruptcy filing of any chapter and any age.
- `d1fdsp` -- first-debtor disposition code (see taxonomy below).
- `d1fprse` -- first-debtor pro se flag.
- `filefy` -- filing fiscal year (Oct 1 through Sep 30, labeled by the ending calendar year).

**Important:** `dschrgd` is a dollar amount of debt discharged, not a binary flag. It is not used in this estimator.

## 4. Disposition taxonomy

From `server.py`:

- `DISCHARGED_CODES = ('A', 'B', '1')` -- case closed with a discharge order entered
- `DISMISSED_CODES = ('H', 'I', 'J', 'K', 'T', 'U', '5')` -- case closed without discharge

Cases with disposition codes outside these two sets are not counted as "closed" in the spine. The universe of closed cases is `discharged + dismissed`.

## 5. The estimator

The spine's `est_violations` column is produced by:

```
est_violations = round(prior_discharged * 0.43)
```

where `prior_discharged` is the deduplicated count of cases satisfying **all** of:

- `crntchp = '13'`
- `prfile = 'Y'`
- `d1fdsp` in `('A', 'B', '1')` (discharge entered)
- `filefy` in the requested fiscal-year range
- one row per `casekey` (latest snapshot)

`0.43` is a constant defined in `server.py` as `ESTIMATED_BAR_RATE`. It represents the assumed fraction of prior-filer-with-discharge cases where the prior filing fell inside the §1328(f) statutory windows.

## 6. What the numbers are, and are not

**MEASURED directly from the IDB (no coefficient):**
- National count of Ch. 13 cases filed FY2008-2024 with `prfile = 'Y'` and a discharge entered: **391,951**
- National Ch. 13 total filings in period: **4,895,163**
- National dismissal rate: **58.3%**
- National prior-filer rate: **33.2%**

**ESTIMATED via the 0.43 coefficient:**
- National §1328(f) violations FY2008-2024: **168,539** (= round(391,951 × 0.43))
- District-level violation counts in the CSV (same derivation, per-district)

**NOT established by this dataset:**
- Any specific case is an adjudicated §1328(f) violation. No court has made that finding for any row here.
- The 0.43 coefficient is an empirical measurement. It is an assumption pending validation.
- Individual debtor identity across cases. FJC has no national debtor key. Name-joining across chapters is out of scope for the spine.

## 7. Known limitations

1. **The 0.43 coefficient is the central assumption.** It attempts to estimate what fraction of prior filers fell inside the §1328(f) time windows (4 yr / 2 yr). Without the prior case's chapter and discharge date linked to the current case, the spine cannot measure this directly. Validation requires sampling ~500 prior-filer discharged cases from RECAP, reconstructing the prior case, and computing the empirical fraction. That work is Phase 3 of the spine. Until complete, treat the violation estimate as a plausibility range, not a point measurement.

2. **Debtor matching**: FJC carries no persistent debtor identifier across cases. Marriage, divorce, and data-entry variants will suppress matches. Direction of bias: undercounts prior filings.

3. **Prior-filing breadth**: `prfile = 'Y'` captures any prior bankruptcy, not just discharged ones. A debtor whose prior Ch. 13 dismissed without discharge still shows `prfile = 'Y'` but would not trigger §1328(f). This is part of why the coefficient is less than 1.

4. **No attorney attribution**: The IDB carries no attorney name or bar number. Attorney-level rollups require PACER/RECAP docket attachment (Phase 3 of the spine).

5. **Small-sample suppression**: Districts with fewer than 20 prior-closed cases have their rate fields suppressed. D. Guam (n=396 total) is the only district in the CSV where this matters materially; its `est_violations = 6` is retained, rates unsuppressed only because `prior_closed` exceeded the threshold.

6. **Fiscal year vs. calendar year**: `filefy` is the federal fiscal year (Oct-Sep), labeled by ending calendar year. FY2024 runs 10/1/2023 through 9/30/2024. Any cross-reference to calendar-year sources must adjust.

7. **Territorial coverage**: D.V.I. and D.N. Mar. I. exist in the district registry but return zero matching rows in this period. Puerto Rico (`04`) is included and substantial.

## 8. Reproducibility

Every value in the CSV is reproducible from one MCP call:

```
fjc_rank_districts(
  metric="est_violations",
  top_n=100,
  start_year=2008,
  end_year=2024,
  ascending=false
)
```

The call executes `rank_districts()` in `docker/fjc-mcp/server.py` (lines 251-308), which is a pure SQL + Python aggregation against `pacer/data/fjc_national.db`. No network calls. No proprietary logic. No cached intermediates.

For external reproduction without the MCP server, the equivalent SQL is in Section 2 of this note plus the code at server.py:251-308.

## 9. Citation format

Any downstream use of the spine should cite as:

> District-level §1328(f) violation estimates derived from the FJC Integrated Database (C13 pending file, FY2008-2024) via the 1328f.org research spine v1 (retrieved 2026-04-19). See methodology note at [URL] Section 5 for the 0.43 coefficient assumption and Section 7.1 for the validation path.

## 10. Revision history

- **v1** (2026-04-19): Initial district-level rollup. 92 districts, one fiscal-year period.
- **v2** (planned): Per-district year-by-year trend artifacts (`{district_code}_trend_fy08_24.csv`). Shows the rise/fall of violations over time.
- **v3** (planned): Empirical validation of the 0.43 coefficient via RECAP docket sample. Will replace the hard-coded constant with a measured estimate plus confidence interval.
- **v4** (planned): Attorney-level rollup for the top 10 districts by violation count. Requires PACER/RECAP docket pulls at scale.
- **v5** (planned): Individual-case vignette pipeline, each vignette pointing to the dataset row that surfaced it.
