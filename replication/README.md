# QILP Replication and Extension

Replicates the Quarterly Industry Labor Productivity (QILP) dataset from Hobijn,
Mestieri, Werquin, and Zhang (Chicago Fed Economic Perspectives No. 1, 2025) and
extends the series through 2026Q1 using June 2026 BEA and post-benchmark CES data.

## What this produces

- `data/processed/qilp_alp_panel.csv` — industry-level ALP growth rates (81 industries, 2006Q2–2026Q1)
- `data/processed/qilp_aggregate_alp.csv` — Törnqvist-weighted aggregate ALP, hours, and output growth
- `data/processed/qilp_extended.xlsx` — published workbook levels through 2025Q1 grafted to our growth rates for 2025Q2–2026Q1
- `data/processed/qilp_baumol_denison.csv` — Baumol-Denison decomposition
- `data/processed/qilp_validation_*.csv` — comparison statistics against the published workbook

## Data requirements

The following files must be in place before running the pipeline.

**Downloaded automatically by the pipeline (Step 1–2):**
- Chicago Fed QILP workbook (qilp_latest.xlsx)
- Chicago Fed paper PDF
- BEA industry-NAICS concordance
- BEA GDP-by-industry data via API (requires `BEA_API_KEY` in `.Renviron`; see below)
- BLS CES payroll and hours data via BLS API (requires `BLS_API_KEY` in `.Renviron`; optional but recommended)

**Requires manual download before running:**

| File | Place at |
|------|----------|
| BEA current-dollar VA quarterly workbook | `data/raw/bea/manual/bea_value_added_quarterly_20260625.xlsx` |
| BEA real VA quarterly workbook | `data/raw/bea/manual/bea_real_value_added_quarterly_20260625.xlsx` |
| BEA VA price index quarterly workbook | `data/raw/bea/manual/bea_value_added_price_index_quarterly_20260625.xlsx` |
| CES published-series reference workbook | `data/raw/bls/ces/cesseriespub.xlsx` |
| IPUMS CPS microdata extract | `data/raw/cps_00003.csv.gz` |

The three BEA workbooks are manual exports from the BEA GDP-by-Industry interactive
tables (TVA105-Q, TVA106-Q, TVA104-Q). The CES workbook is the official BLS
cesseriespub file. The CPS extract requires an IPUMS account; the variables needed
are YEAR, MONTH, WTFINL, EMPSTAT, CLASSWKR, IND, UHRSWORKT, UHRSWORK1, UHRSWORK2.

**API keys** (optional but speeds up Steps 2 and 8):

```
BEA_API_KEY=your_key_here
BLS_API_KEY=your_key_here
```

Add these to `~/.Renviron`. Without `BEA_API_KEY` the pipeline falls back to the
manual BEA workbooks. Without `BLS_API_KEY` the BLS API still works but at a lower
rate limit.

## How to run

Open `code/qlip_replication.Rmd` and knit to HTML, or run chunk-by-chunk in
RStudio. Required R packages are listed in the setup chunk; the pipeline will stop
with an informative error if any are missing.

Runtime is approximately 15 minutes depending on API response times.

## Validation summary

Comparison against the published workbook (2006Q2–2025Q2, n = 74 quarters):

| Series | Correlation | MAE |
|--------|-------------|-----|
| Hours growth | 0.987 | 0.0025 |
| ALP growth | 0.872 | 0.0031 |

The residual gap is attributable to two unresolvable data vintage differences: the
BEA August 2025 comprehensive revision (Issue 1 in the pipeline documentation) and
the BLS February 2026 CES benchmark revision (Issue 2). Neither vintage was
available when the published workbook was built.

## Known open issues

**Issue E3: Housing payroll employment overcount.** Under Method A (zero hours for
housing), Housing is excluded from the aggregate, so this does not affect the
Törnqvist aggregate ALP. It does affect the Housing row in the industry-level
validation panel. BEA NIPA Table 6.5D was checked but only publishes "Real estate"
as a single row with no Housing/ORE split, so the required disaggregation is not
available from that source.

**Other transportation equipment employment level.** The residual CES construction
(transport total minus motor vehicles) gives 2.6x the published employment level.
Aerospace all-employees (`CEU3133640001`) does not exist in the published CES panel,
so the residual cannot be further refined. AWH uses the aerospace proxy
(`CEU3133640002`) and is unaffected.

## Diagnostic script

`code/test_sa_truncation.R` investigates whether the 2025Q2 hours divergence from
the published workbook is driven by seasonal adjustment methodology (truncating the
X-13 sample at June 2025 vs. running it through 2026Q1) or by the underlying raw
CES data. Finding: the gap originates in the raw data, not in SA. See Issue 2 in
the pipeline documentation for details.
