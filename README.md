# Smart-Quality-System-for-Consumer-Electronics-Manufacturing-Case-iPhone-Camera-Module-Cork-

# Apple-Style Quality Control Tower (iPhone Camera Module, Cork)

End-to-end Quality Control Tower spanning **IQC → Reliability → OQA/FPY → Field → Predictive** for an iPhone camera module line in **Cork**. Built from **CSV datasets only** (implicit aggregations; no custom measures). Data is **synthetic but realistic** with controlled irregularities to mirror factory conditions.

---

## TL;DR

* **Scope:** Supplier quality, incoming inspection, reliability stress tests, line yield/OQA, field returns, predictive triage
* **Stack:** Power BI Desktop (dashboards), CSV datasets
* **Design:** Black & white, Apple-clean; volume-aware visuals; Top-N focus
* **Fit:** Demonstrates Apple Product Operations Quality Engineer skills (NPI, quality systems, supplier, yield, reliability, field, analytics, documentation)

---

## Repository Structure

```
.
├─ data/
│  └─ raw/
│     ├─ suppliers.csv
│     ├─ incoming_inspection.csv
│     ├─ reliability_tests.csv
│     ├─ oqa_yield.csv
│     ├─ field_quality.csv
│     └─ predictions.csv          # optional
├─ dashboards/
│  └─ Quality_Control_Tower.pbix  # optional – or build via steps below
├─ docs/
│  ├─ Report.pdf                  # executive-ready report
│  ├─ OnePager_OpsMemo.pdf        # concise ops memo
│  └─ Screenshots/                # exported page PNGs
└─ README.md
```

---

## Datasets (keys, row counts, purpose)

* **suppliers.csv** — master data (EU/APAC, audit\_score, aql\_level, ppap\_status, capability\_index\_est, preferred\_supplier)
  *Rows:* \~6 • **Key:** `supplier_id`
* **incoming\_inspection.csv** — IQC lots (qty\_received, samples\_inspected, defects\_found, critical/major/minor, accept\_reject, ppm\_incoming, CTQ stats)
  *Rows:* \~51,997 • **Key:** `lot_id` • **FK:** `supplier_id`
* **reliability\_tests.csv** — weekly stress (thermal, drop, vibration, humidity; units\_tested/failed, pass\_rate, fail\_mode)
  *Rows:* \~355 • **Key:** `build_id`
* **oqa\_yield.csv** — daily line/build (units\_built, first\_pass\_yield, rework\_units, scrap\_units, oqa\_defects, oqa\_pass\_rate, shift)
  *Rows:* \~4,472 • **FK:** `build_id`
* **field\_quality.csv** — monthly region (units\_sold, field\_returns, rma\_rate, top\_field\_issue)
  *Rows:* \~80
* **predictions.csv (optional)** — per date/build risk score for low FPY (`pred_prob`), used for triage tables

**Linkage:** `supplier_id` joins Suppliers→IQC; `build_id` joins Reliability↔OQA; time is day/week/month per file.

**Irregularities included intentionally:** small nulls, a few duplicate lots removed, outliers (PPM spikes, FPY dips), seasonal shifts, minor typos, brief AQL/PPAP drift, occasional CTQ scale inconsistency (pre-normalized).

---

## Getting Started (Power BI, no custom measures)

1. **Get Data → Text/CSV**: load all files in **data/raw**.
2. **Data types:**

   * Dates (Date), rates (Decimal), counts/qty (Whole Number), IDs (Text).
3. **Relationships (recommended):**

   * `suppliers[supplier_id]` → `incoming_inspection[supplier_id]` (1:\*).
   * `reliability_tests[build_id]` → `oqa_yield[build_id]` (1:\*).
   * Use each table’s native date for slicers on that page.
4. **Aggregations (strict):**

   * **Average** for rates/percentages: `first_pass_yield`, `oqa_pass_rate`, `pass_rate`, `ppm_incoming`, `rma_rate`.
   * **Sum** for volumes: `units_built`, `rework_units`, `scrap_units`, `oqa_defects`, `units_tested`, `units_failed`, `qty_received`, `field_returns`.

> This build uses **implicit aggregations only** (no DAX measures/tables). A minimal “pro upgrade” is listed below if you want weighted KPIs and SPC limits.

---

## Dashboards (pages)

### 1) Executive Overview

* **Signals:** Avg FPY (\~0.97) with seasonal dips; Avg Reliability Pass (\~0.98); Avg Incoming PPM (\~115); Avg RMA (\~1%).
* **Volumes:** \~10M units built; Rework \~208k; Scrap \~92.8k.
* **Visuals:** FPY vs Units (line+columns), Rework/Scrap by line, Supplier PPM Top-N, Reliability by test type, RMA by region.

### 2) Supplier Quality (IQC)

* **Insights:** Higher Avg PPM in S5002/S7142/S6577; EU suppliers lower; Accept dominates with major/minor defects most common.
* **Visuals:** Avg PPM by supplier, defect mix (critical/major/minor), Accept-vs-Reject by supplier, PPM by AQL, supplier table (PPM, qty, audits).

### 3) Reliability

* **Insights:** **Humidity** has the lowest pass rate; \~60k tested / \~1,082 failed; fail modes led by **sensor\_shift**, then **ois\_drift**.
* **Visuals:** Avg pass by test type, weekly trend, failure-mode Pareto, test summary table.

### 4) OQA & Yield

* **Insights:** FPY inversely correlates with OQA defects; rework/scrap concentrated on high-volume lines (L1/L2).
* **Visuals:** FPY vs Units by day, Rework/Scrap by line (small multiples by shift), scatter FPY vs OQA defects (size = units\_built), build table.

### 5) Field Quality

* **Insights:** \~1.0–1.35M sold; \~8.2k returns; RMA peaks late summer/autumn; issues: camera\_focus, app\_crash\_camera, ois\_noise, battery\_swelling.
* **Visuals:** Avg RMA by region, monthly RMA trend, returns by top\_field\_issue, region×month matrix (Avg RMA with color scale).

### 6) Predictive Alerts (optional)

* **Insights:** \~1,628 builds flagged; Avg predicted risk \~0.22; workload peaks Mar–Jun; both shifts exposed.
* **Visuals:** Top-risk builds table (sort by `pred_prob`), alerts per day (count by date), distribution of risk (bin/column), numeric slicer on `pred_prob`.

---

## Insights → Actions (2–4 week plan)

* **Tighten AQL** for S5002/S7142 (e.g., 1.5 → 1.0) temporarily; monitor Incoming PPM.
* **Add humidity screen** in monsoon windows; watch Reliability Pass and OQA defects.
* **Kaizen at Line/Shift** hotspots (e.g., L2-B): standard work, handling poka-yokes, gage checks.
* **Daily triage** by risk threshold (e.g., ≥0.55 if available); open CAPA on repeat offenders.

---

## Limitations & Upgrade Path

* **Current:** implicit averages (unweighted), no SPC lines, no COPQ costing.
* **Minimal Pro Upgrade (add later):**

  * Weighted FPY/PPM/RMA measures
  * FPY control limits (mean/std/UCL/LCL)
  * Composite risk & NPI readiness gates
  * Basic COPQ (€) with What-If costs

These eight measures make it factory-grade while keeping the model lean.

---

## License

Data is synthetic and safe to share. Choose a license that fits your use (MIT recommended).

## Author

Prepared by **\Krishnali Lahane** — Manufacturing Quality & Analytics (Product Operations / Quality Engineering focus).
